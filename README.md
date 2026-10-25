# WhatsApp MCP Server for the official WhatsApp Business API

[![npm](https://img.shields.io/npm/v/mcp-wassenger.svg)](https://www.npmjs.com/package/mcp-wassenger)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

A hosted [Model Context Protocol](https://modelcontextprotocol.io) server that lets
Claude, Cursor, VS Code, the OpenAI API and any other MCP client work with WhatsApp
through the **official WhatsApp Business API**: send and read messages, reply with
approved templates, run campaigns, manage contacts and run a shared team inbox.

It is run by [Wassenger](https://wassenger.com). There is nothing to install:
point your client at `https://api.wassenger.com/mcp` and send your Wassenger API key
in the `Authorization` header. Claude and ChatGPT on the web have no field for a header,
so there the key goes in the address instead.

```text
Remote URL:  https://api.wassenger.com/mcp
Transport:   Streamable HTTP
Auth:        Authorization: Bearer <WASSENGER_API_KEY>
             or https://api.wassenger.com/mcp?key=<WASSENGER_API_KEY> for clients without headers
```

## Business API or personal account?

There are two kinds of WhatsApp MCP servers, and they solve different problems.

| | This server (Wassenger) | Personal-account servers such as [lharries/whatsapp-mcp](https://github.com/lharries/whatsapp-mcp) |
|---|---|---|
| How it connects | Official WhatsApp Business API (Meta Cloud API) | Links your personal account as a WhatsApp Web device |
| Setup | Hosted. Add a URL and an API key | Run a Go bridge and a Python server locally, then scan a QR code |
| Messages outside the 24-hour window | Yes, with approved templates | No templates |
| Several people on one number | Shared inbox, chat assignment, departments, labels | One user |
| Campaigns and bulk sends | Yes, paced by a delivery queue | No |
| Where messages are stored | Your Wassenger account | A SQLite database on your computer |
| WhatsApp's terms | Uses the API Meta provides for businesses | WhatsApp's [Terms of Service](https://www.whatsapp.com/legal/terms-of-service) don't allow unofficial clients, so the number can be restricted |
| Cost | Wassenger subscription; Meta bills template messages directly | Free and open source |

If you only want Claude to read and search your own personal chats, a personal-account
server is the simpler choice. If WhatsApp is how your business talks to customers, use
the Business API.

## Quick start

1. Create a [Wassenger](https://wassenger.com) account and connect your WhatsApp
   Business number.
2. Copy your API key from [app.wassenger.com/apikeys](https://app.wassenger.com/apikeys).
3. Add the server to your client with one of the configurations below.
4. Ask: *"Which of my WhatsApp numbers are connected?"* to check it works.

## Connect your client

### Claude (web and desktop app)

Open **Customize → Connectors**, click **+**, choose **Add custom connector**, give it a
name and paste the address with your key:

```text
https://api.wassenger.com/mcp?key=YOUR_WASSENGER_API_KEY
```

Click **Add**. In a chat, open the **+** menu, choose **Connectors** and switch it on. On
Team and Enterprise plans only an owner can add a custom connector, and everyone who uses
it shares that key.

### ChatGPT (web)

ChatGPT reaches a custom MCP server from the web app, on the plans where OpenAI enables
developer mode, and a workspace admin has to turn it on. Go to **Apps → Create**, paste the
same address, choose **No authentication**, click **Scan tools** and then **Create**.
Choosing OAuth fails, because this server publishes no OAuth metadata.

### Claude Code

```bash
claude mcp add --transport http wassenger https://api.wassenger.com/mcp \
  --header "Authorization: Bearer YOUR_WASSENGER_API_KEY"
```

### Clients that only run local servers

Some clients only start local processes, like Claude Desktop configured through
`claude_desktop_config.json`. Use the npm bridge from this repository
(`mcp-wassenger`): it connects to the hosted server and sends your key in the
`Authorization` header.

```json
{
  "mcpServers": {
    "wassenger": {
      "command": "npx",
      "args": ["-y", "mcp-wassenger", "YOUR_WASSENGER_API_KEY"]
    }
  }
}
```

### Cursor

`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "wassenger": {
      "url": "https://api.wassenger.com/mcp",
      "headers": { "Authorization": "Bearer YOUR_WASSENGER_API_KEY" }
    }
  }
}
```

### VS Code

`.vscode/mcp.json`. VS Code asks for the key the first time and stores it securely:

```json
{
  "inputs": [
    { "type": "promptString", "id": "wassenger-key", "description": "Wassenger API key", "password": true }
  ],
  "servers": {
    "wassenger": {
      "type": "http",
      "url": "https://api.wassenger.com/mcp",
      "headers": { "Authorization": "Bearer ${input:wassenger-key}" }
    }
  }
}
```

### OpenAI API (Responses API and Agents SDK)

```javascript
import OpenAI from 'openai'

const openai = new OpenAI()

const response = await openai.responses.create({
  model: 'gpt-6.1-sol',
  input: 'Summarize the unread WhatsApp chats from today',
  tools: [{
    type: 'mcp',
    server_label: 'wassenger',
    server_url: 'https://api.wassenger.com/mcp',
    headers: { Authorization: `Bearer ${process.env.WASSENGER_API_KEY}` },
    require_approval: 'always'
  }]
})
```

### Any other client

Use the remote URL with Streamable HTTP and the `Authorization` header. If your client has
no field for headers, use the `?key=` address; if it only runs local servers, use the npm
bridge: `npx -y mcp-wassenger YOUR_WASSENGER_API_KEY`.

## What you can ask

- *"Show me the unread chats from today and draft a reply to each one."*
- *"Assign every chat that mentions 'invoice' to the billing department."*
- *"Send the `order_shipped` template to +1 555 555 0100 with tracking number 1Z999."*
- *"Which customers asked about pricing this week and got no answer?"*
- *"Create a campaign with the `spring_offer` template for contacts labelled 'VIP'."*
- *"Move the deal for Acme to 'Proposal sent' and add a note with today's call summary."*
- *"Check whether these 20 numbers have WhatsApp before I import them."*
- *"How many chats did we open, resolve and leave pending this week?"*

## Tools

More than 50 tools, grouped by what they work on:

| Area | What the tools do |
|---|---|
| Messages | Send text, media, location, contacts and scheduled messages; reply, react and forward |
| Templates | List, create and delete templates; send approved templates outside the 24-hour window |
| Chats | Search and filter chats by status, label, agent, date or unread; read message history; export chats; chat statistics |
| Contacts and labels | Create, search, update and import contacts; create and apply labels |
| Team inbox | Manage agents and departments; hand a chat to a teammate or department, resolve or reopen it, leave internal notes, mark it read or unread |
| Campaigns | Create, start and stop campaigns and manage their recipients; monitor the delivery queue |
| Sales pipeline | Create pipelines and stages, track deals, add notes and tasks to deals |
| WhatsApp Flows | List flows, send a flow and read its responses |
| Files | Upload files from a URL, search sent and received files, download media |
| Numbers and devices | List connected numbers and their status; check whether a phone number is on WhatsApp |

Groups, Channels and Status updates only work on numbers connected by QR code, because
the official Business API doesn't offer them.

## Security

- Use the `Authorization` header whenever your client has a field for it. A key in the
  address (`?key=`) ends up in access logs, proxy logs, error reports and browser
  history, so treat it as exposed: for Claude and ChatGPT on the web, create a key used
  only there and revoke it if the address leaks.
- An agent with this server can read your customers' messages and send messages in your
  name. Treat untrusted message content as untrusted input: keep approval on for tools
  that send or delete, as in the OpenAI example above.
- Revoke a key at any time from [app.wassenger.com/apikeys](https://app.wassenger.com/apikeys).

## FAQ

**Is there a WhatsApp MCP server I can use with Claude?**
Yes. This one works in Claude on the web and in the desktop app as a custom connector,
in Claude Code and in any MCP client, on top of the official WhatsApp Business API. Setup
takes a URL and an API key.

**How do I connect WhatsApp to Claude or ChatGPT as an AI agent?**
Connect your number to Wassenger, then add this server: as a custom connector in Claude,
as an app in ChatGPT's developer mode, or as an MCP tool in an agent built on the OpenAI
API, with the configurations above. The agent can then read chats, reply, send templates
and hand conversations to people on your team.

**Do I need to keep a phone or computer switched on?**
No. Numbers on the Business API run in Meta's cloud and the MCP server is hosted.

**Can it message customers who haven't written in the last 24 hours?**
Yes, with a template approved by Meta. Free-form messages are only allowed within 24 hours
of the customer's last message; that is a WhatsApp rule for every provider.

**Is it free?**
The hosted server comes with your Wassenger account, and the npm bridge in this
repository is open source under the MIT license. You need a Wassenger account with a
connected number: see [pricing](https://wassenger.com/pricing). Meta bills template
messages directly.

## Links

- [Wassenger](https://wassenger.com) · [Pricing](https://wassenger.com/pricing) · [API documentation](https://app.wassenger.com/docs)
- [Connect WhatsApp to Claude: step-by-step guide](https://wassenger.com/blog/connect-whatsapp-to-claude-ai-mcp)
- [Model Context Protocol](https://modelcontextprotocol.io)

## License

MIT
