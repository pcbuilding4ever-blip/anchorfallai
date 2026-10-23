# AnchorFall AI ticket helper

This bot answers members inside standard private support-ticket threads, asks for missing details, and hands the ticket to staff once the issue is clear or the member asks for a human. It sends one staff-role ping per ticket, then stops replying after handoff.

## Before starting

- The bot's token is stored as `BOT_TOKEN` in Wispbyte.
- Add an OpenAI API key as `OPENAI_API_KEY` in Wispbyte. API usage is billed separately from ChatGPT.
- In the Discord Developer Portal, enable **Message Content Intent** for this bot.
- Invite the AI bot to the same Discord server with **View Channels**, **Read Message History**, **Send Messages**, and **Send Messages in Threads**. For message moderation, grant **Moderate Members** and place the bot's role above members it may timeout.
- Ticket conversations are sent to the OpenAI API to generate AI replies and staff summaries. Eligible non-ticket server messages are also sent to classify possible scams, real-money trading, gambling promotion, and self-promotion. Let members know an AI service processes these messages.

## Wispbyte

Upload the repository files, then use **Startup** with:

- Startup command: `npm start`
- Required variables: `BOT_TOKEN`, `OPENAI_API_KEY`
- Optional variables: `OPENAI_MODEL` (defaults to `gpt-6-astra`), `TICKET_PARENT_CHANNEL_ID`, `STAFF_ROLE_ID`, `MESSAGE_MODERATION_ENABLED`, `MODERATION_LOG_CHANNEL_ID`, `MODERATION_EXEMPT_CHANNEL_IDS`

The ticket channel and staff-role variables default to the IDs used by the existing AnchorFall bot.

## AI message moderation

When enabled, the AI bot checks new messages in server text channels using the OpenAI API. It skips bots, configured exempt channels, and standard private ticket threads. Staff messages are checked like other member messages.

- Clear messages receive no action.
- A clear violation receives a **3-hour timeout**.
- A message with a concrete but ambiguous signal receives a **1-hour timeout**.
- If OpenAI is unavailable or returns an invalid result, the bot applies no timeout.
- Every eligible message, including staff messages, is logged with its text, AI reason, and source link: green for safe, orange for a 1-hour timeout, and red for a 3-hour timeout. Bots, private ticket threads, configured exempt channels, and the log channel itself are skipped. Messages without readable text or failed AI checks are logged gray and receive no timeout.
- The bot sends the timed-out member a notice. Configure a private moderation log channel so staff can review the category, reason, and message link. Without that channel, actions are written only to the server process log.
- The bot needs **Moderate Members** permission, and its role must be above the members it may timeout.
- Eligible non-ticket messages are sent to the OpenAI API for classification. Tell server members that AI processes these messages.

Optional Wispbyte variables:

- `MESSAGE_MODERATION_ENABLED=false` disables message moderation; it is enabled by default.
- `MODERATION_LOG_CHANNEL_ID` sets the staff log channel; it defaults to `1563113650552569896`.
- `MODERATION_EXEMPT_CHANNEL_IDS` is a comma-separated list of channel, category, or forum IDs to skip.

## Connect it to the existing ticket bot

The existing bot creates **private threads**. It must add the AI bot to each new support ticket thread; the AI bot cannot read a private thread it has not been invited to. Configure `AI_TICKET_BOT_ID` on the existing bot host and use the companion integration in the `dsmp-pvp-mod-bot` repository. Its startup step will also invite the AI bot into existing active standard tickets.

Once the AI bot is invited to a thread, it can read and reply there. If staff were already pinged in an older ticket, the AI helper will not send a duplicate handoff ping.
