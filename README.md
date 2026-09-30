# Email Spam Tester MCP server

Email Spam Tester is a free, independent email deliverability testing tool built by Serhii Tanichev. This repository is the client side of its MCP server: how to connect an agent to it, what the tools do, and how to read what they return. There is nothing to install on a server; the MCP endpoint is hosted with the tool.

- Site: https://email-spam-tester.com/
- Agent guide: https://email-spam-tester.com/skill/
- API and examples: https://github.com/serg-tanichev/email-spam-tester


The service exposes an MCP server at `https://email-spam-tester.com/mcp` over streamable HTTP. No key, no account. The one-hour test works in every client:

| Tool | What it does |
|---|---|
| `get_test_address(lang, include_placement=false)` | Reserves a disposable address and returns it with the slug and the report URL. `lang` decides the language of the fix plan a person sees on the report page. With `include_placement` the same send also shows which folder the message lands in at different mailbox providers: you get one list of recipients and a marker for the subject. Some of those inboxes are public test inboxes, so use it for mail that is not confidential. |
| `wait_for_report(slug, timeout_seconds=150)` | Blocks until the analysis is done and returns a summary: scores, the failing and warning checks with their evidence, the fix plan. Nothing to poll and no sleep loop to get wrong. |
| `get_report(slug)` | Reads an existing report without waiting. |
| `get_message_source(slug)` | The message as delivered: headers in wire order, both bodies, the raw source. |

`get_report` also returns provider placement so far, when the test asked for it.

In ChatGPT there is more, because ChatGPT tells the server which of its users is asking. Permanent mailboxes collect a series of letters (a welcome sequence, a newsletter) and check every one: `open_series_mailbox(domain)`, `list_series_mailboxes()`, `get_series_letters(mailbox_id)`, `write_fix_plan(slug)` and `close_series_mailbox(mailbox_id)`. Other clients get a short error from these and the rest works as before. Every tool also returns structured content for an MCP Apps card (`ui://widget/email-spam-tester-v3.html`): the address with copy buttons that turns into the report when the mail lands. Clients that draw no card read the same result as text.

The agent's job in between is to send a real message to the address, from the platform that will send the campaign. Half the checks read headers the sending platform adds, so a copy sent from a laptop says little about it.

## Claude Desktop

Add the server to `claude_desktop_config.json` (see [claude-desktop.json](claude-desktop.json) for the whole file):

```json
{
  "mcpServers": {
    "email-spam-tester": {
      "url": "https://email-spam-tester.com/mcp"
    }
  }
}
```

Clients that do not speak streamable HTTP directly can bridge it with `mcp-remote`:

```json
{
  "mcpServers": {
    "email-spam-tester": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://email-spam-tester.com/mcp"]
    }
  }
}
```

## ChatGPT

The ChatGPT plugin package (manifest, icons, review test cases) is in [serg-tanichev/email-spam-tester-chatgpt-plugin](https://github.com/serg-tanichev/email-spam-tester-chatgpt-plugin). Until it is in the plugin directory you can add the server yourself: Plugins → Add → Create MCP App, server URL `https://email-spam-tester.com/mcp`, no authentication.

## Claude Code

```bash
claude mcp add --transport http email-spam-tester https://email-spam-tester.com/mcp
```

## Reading what comes back

- The per-check lines the tools return are in English whatever `lang` was set to; `lang` decides what the person opening the report URL sees.
- A check status of `skip` is not a pass: the check did not apply. `error` means it could not run and is left out of the score.
- `complete: false` means something could not be checked. Treat the score as optimistic until it is true.
- The slug is the capability: whoever holds it can read the report, including the subject, the sender, the bounce address and the full source. Do not test somebody else's mail without asking.

The same instructions, written for an agent to read before it starts, are in [SKILL.md](https://email-spam-tester.com/skill/SKILL.md), and in the project repository [serg-tanichev/email-spam-tester](https://github.com/serg-tanichev/email-spam-tester).

## Where it is listed

- [MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=io.github.serg-tanichev/email-spam-tester), the official one, as `io.github.serg-tanichev/email-spam-tester`
- [Smithery](https://smithery.ai/servers/tanichev/email-spam-tester)
- [Glama](https://glama.ai/mcp/connectors/io.github.serg-tanichev/email-spam-tester)

## Files

- `claude-desktop.json`: a complete Claude Desktop configuration with the server added.
- `server.json`: the manifest in the MCP registry format, for registries and directories that read one.

## License

MIT, see [LICENSE](LICENSE). The license covers this repository, not the service.
