# AnchorFall AI ticket helper

This bot answers members inside standard private support-ticket threads, asks for missing details, and hands the ticket to staff once the issue is clear or the member asks for a human. It sends one staff-role ping per ticket, then stops replying after handoff.

## Before starting

- The bot's token is stored as `BOT_TOKEN` in Wispbyte.
- Add an OpenAI API key as `OPENAI_API_KEY` in Wispbyte. API usage is billed separately from ChatGPT.
- In the Discord Developer Portal, enable **Message Content Intent** for this bot.
- Invite the AI bot to the same Discord server with **View Channels**, **Read Message History**, **Send Messages**, and **Send Messages in Threads**.
- Ticket conversations are sent to the OpenAI API to generate the AI replies and staff summaries. Let ticket users know an AI service processes their messages.

## Wispbyte

Upload the repository files, then use **Startup** with:

- Startup command: `npm start`
- Required variables: `BOT_TOKEN`, `OPENAI_API_KEY`
- Optional variables: `OPENAI_MODEL` (defaults to `gpt-6-astra`), `TICKET_PARENT_CHANNEL_ID`, `STAFF_ROLE_ID`

The ticket channel and staff-role variables default to the IDs used by the existing AnchorFall bot.

## Connect it to the existing ticket bot

The existing bot creates **private threads**. It must add the AI bot to each new support ticket thread; the AI bot cannot read a private thread it has not been invited to. Configure `AI_TICKET_BOT_ID` on the existing bot host and use the companion integration in the `dsmp-pvp-mod-bot` repository. Its startup step will also invite the AI bot into existing active standard tickets.

Once the AI bot is invited to a thread, it can read and reply there. If staff were already pinged in an older ticket, the AI helper will not send a duplicate handoff ping.
