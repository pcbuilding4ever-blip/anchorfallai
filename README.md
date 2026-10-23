# AnchorFall AI ticket helper

This bot answers members inside standard private support-ticket threads, asks for missing details, and hands the ticket to staff once the issue is clear or the member asks for a human. It sends one staff-role ping per ticket, then stops replying after handoff.

## Before starting

- The bot's token is stored as `BOT_TOKEN` in Wispbyte.
- Add a Groq API key as `GROQ_API_KEY` in Wispbyte. The free plan has per-model rate and daily limits; see [Groq’s current limits](https://console.groq.com/docs/rate-limits).
- In the Discord Developer Portal, enable **Message Content Intent** and **Server Members Intent** for this bot.
- Invite the AI bot to the same Discord server with **View Channels**, **Read Message History**, **Send Messages**, and **Send Messages in Threads**. For message moderation, grant **Moderate Members** and place the bot's role above members it may timeout. For captcha verification, also grant **Manage Roles**, **Manage Messages**, **Create Private Threads**, **Manage Threads**, and **Attach Files**; place the bot's role above the verified role.
- Ticket conversations are sent to Groq to generate AI replies and staff summaries. Eligible non-ticket server messages are also sent to classify possible scams, real-money trading, gambling promotion, and self-promotion. Let members know an AI service processes these messages.

## Wispbyte

Upload the repository files, then use **Startup** with:

- Startup command: `npm start`
- Required variables: `BOT_TOKEN`, `GROQ_API_KEY`
- Optional variables: `GROQ_MODEL` (defaults to `openai/gpt-oss-20b`), `TICKET_PARENT_CHANNEL_ID`, `STAFF_ROLE_ID`, `MESSAGE_MODERATION_ENABLED`, `MODERATION_LOG_CHANNEL_ID`, `MODERATION_EXEMPT_CHANNEL_IDS`, `VERIFICATION_CHANNEL_ID`, `VERIFIED_ROLE_ID`, `JOIN_ROLE_ID`

The ticket channel and staff-role variables default to the IDs used by the existing AnchorFall bot.

## Image captcha verification

The verification channel and role default to `1563125387238838422` and `1562149545604026468`. You can override them with `VERIFICATION_CHANNEL_ID` and `VERIFIED_ROLE_ID` in Wispbyte. In Discord's Developer Portal, enable **Server Members Intent**.

On joining, non-bot members are also given the join role `1550427313949122630`. Override it with `JOIN_ROLE_ID` if needed. This join role is separate from the verified role; make sure its channel permissions do not bypass any access restrictions you want the captcha to enforce.

When someone joins, the bot creates a private thread in the verification channel, invites them, and sends a randomly generated captcha image. They type the six digits in that thread; the bot deletes their answer and grants the verified role only when it matches. Captchas expire after 15 minutes, and five incorrect answers create a fresh captcha. Existing unverified members can type `!verify` in the verification channel. If the bot restarts during a pending captcha, it sends a fresh image to the existing thread.

Give the bot **Manage Roles** (with its role above the verified role), **Manage Messages**, **Create Private Threads**, **Manage Threads**, **Send Messages in Threads**, and **Attach Files** in the verification channel. Members must be able to view the verification channel and send `!verify`; private threads let each person see only their own captcha. Set the server's channel permissions so unverified members can access only the verification area until they receive the verified role. Verification messages and captcha answers are excluded from scam moderation.

## AI message moderation

When enabled, a local keyword filter first checks new messages in server text channels. Messages without matching terms are logged as passing the filter and are not sent to Groq; likely matches are sent to Groq so it can judge context. The filter includes terms such as Nitro, giveaways, bio, DM, scam/phishing, prizes, rewards, account verification, passwords and codes, gift cards, game currency, payment and crypto terms, gambling, trading, and common self-promotion phrases. It skips bots, configured exempt channels, and standard private ticket threads. Staff messages are checked like other member messages.

- Clear messages receive no action.
- A clear violation receives a **3-hour timeout**.
- A message with a concrete but ambiguous signal receives a **1-hour timeout**.
- If Groq is unavailable or returns an invalid result, the bot applies no timeout.
- Every eligible message, including staff messages, is logged with its text and source link. Messages that pass the local keyword filter are logged green as not sent to AI; AI-reviewed safe messages are green, uncertain matches are orange for a 1-hour timeout, and clear violations are red for a 3-hour timeout. Bots, private ticket threads, configured exempt channels, and the log channel itself are skipped. Messages without readable text or failed AI checks are logged gray and receive no timeout.
- The bot sends the timed-out member a notice. Configure a private moderation log channel so staff can review the category, reason, and message link. Without that channel, actions are written only to the server process log.
- The bot needs **Moderate Members** permission, and its role must be above the members it may timeout.
- Eligible non-ticket messages are sent to Groq for classification. Tell server members that AI processes these messages.

Optional Wispbyte variables:

- `MESSAGE_MODERATION_ENABLED=false` disables message moderation; it is enabled by default.
- `MODERATION_LOG_CHANNEL_ID` sets the staff log channel; it defaults to `1563113650552569896`.
- `MODERATION_EXEMPT_CHANNEL_IDS` is a comma-separated list of channel, category, or forum IDs to skip.

## Connect it to the existing ticket bot

The existing bot creates **private threads**. It must add the AI bot to each new support ticket thread; the AI bot cannot read a private thread it has not been invited to. Configure `AI_TICKET_BOT_ID` on the existing bot host and use the companion integration in the `dsmp-pvp-mod-bot` repository. Its startup step will also invite the AI bot into existing active standard tickets.

Once the AI bot is invited to a thread, it can read and reply there. If staff were already pinged in an older ticket, the AI helper will not send a duplicate handoff ping.

