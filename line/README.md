# line — LINE channel for Claude Code

MCP plugin that bridges a LINE Messaging API channel into a Claude Code
session. Sibling of `discord-interactive` in the `zac-cc-plugins`
marketplace.

## What it does

- Runs an HTTP webhook server (default port 3100) that receives events
  from LINE Platform
- Verifies HMAC-SHA256 signatures
- Delivers allowed messages to the Claude Code session as MCP
  `notifications/claude/channel` events
- Exposes `reply` and `push` MCP tools for outbound messages (reply
  token first, push fallback)
- Handles pairing, allowlists, and per-group policies via a local
  `access.json` state file

## Requirements

- [Bun](https://bun.sh) in `PATH`
- A publicly reachable HTTPS URL proxying to port 3100 (or your
  configured `LINE_WEBHOOK_PORT`)
- A LINE Messaging API channel with its **Channel secret** and
  **Channel access token**

## Installation

```bash
# From any Claude Code project
/plugin marketplace add <path-or-url-to>/zac-cc-plugins
/plugin install line@zac-cc-plugins
```

## Configuration

Create `~/.claude/channels/line/.env` with:

```
LINE_CHANNEL_SECRET=<32-char hex>
LINE_CHANNEL_ACCESS_TOKEN=<long token>
LINE_WEBHOOK_PORT=3100
```

Mode 600. See `ACCESS.md` for the full access control model.

## Webhook URL

LINE Platform must be able to POST to `https://<your-public-host>/webhook`.
Set this URL in **LINE Developers Console → your channel → Messaging API
→ Webhook URL**, then click **Verify**.

## Slash commands (provided by this plugin)

- `/line:access pair <code>` — approve a pending DM pairing
- `/line:access allow <userId>` — explicitly allow a user
- `/line:access group <groupId>` — allow a group
- `/line:configure` — configuration helper

(See `skills/access/` and `skills/configure/` for the full skill
definitions.)

## MCP tools

- `reply(user_id, text)` — try reply token (free), fall back to push
- `push(user_id, text)` — direct push (counts against monthly quota)

Both auto-chunk text to the 5000-char LINE limit.

## Known limitations

- `access.json` group policy `requireMention` field is read but NOT
  enforced in `gate()` — all group messages in an allowed group
  are delivered. Tracked as a follow-up; see elf spec §6 for
  rationale (all-messages-into-session is the current design).
- Push-to-group after reply token expiry: pushing to the sender's
  userId lands in DM, not the group. Use push with `to=<groupId>` for
  reliable group posting after 1 minute.
- No media upload; only receives images into `inbox/`.

## State

All state lives in `~/.claude/channels/line/`:
- `.env` — credentials
- `access.json` — access control (allowlists, pending pairings, group policies)
- `approved/` — approval signal files (polled by server every 5s)
- `inbox/` — downloaded images from inbound `image` messages

## License

Same as parent marketplace.
