---
name: configure
description: Set up the LINE channel — save credentials and review access policy. Use when the user pastes LINE credentials, asks to configure LINE, asks "how do I set this up," or wants to check channel status.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash(ls *)
  - Bash(mkdir *)
---

# /line:configure — LINE Channel Setup

Writes the channel credentials to `~/.claude/channels/line/.env` and orients
the user on access policy. The server reads both files at boot.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### No args — status and guidance

Read both state files and give the user a complete picture:

1. **Credentials** — check `~/.claude/channels/line/.env` for
   `LINE_CHANNEL_SECRET` and `LINE_CHANNEL_ACCESS_TOKEN`. Show set/not-set;
   if set, show first 10 chars masked (`abc123...`).

2. **Access** — read `~/.claude/channels/line/access.json` (missing file =
   defaults: `dmPolicy: "pairing"`, empty allowlist). Show:
   - DM policy and what it means in one line
   - Allowed senders: count, and list IDs
   - Pending pairings: count, with codes if any

3. **Webhook** — remind: LINE webhook URL should point to
   `https://<your-domain>:<port>/webhook` (default port 3100). They'll need
   HTTPS (use ngrok or a reverse proxy for local dev).

4. **What next** — end with a concrete next step based on state:
   - No credentials → *"Get Channel Secret and Channel Access Token from
     LINE Developers Console, then run `/line:configure secret <secret>`
     and `/line:configure token <token>`."*
   - Credentials set, nobody allowed → *"DM your bot on LINE. It replies
     with a code; approve with `/line:access pair <code>`."*
   - Credentials set, someone allowed → *"Ready. DM your bot to reach the
     assistant."*

### `secret <value>` — save channel secret

1. Treat the argument as the secret (trim whitespace).
2. `mkdir -p ~/.claude/channels/line`
3. Read existing `.env` if present; update/add the `LINE_CHANNEL_SECRET=`
   line, preserve other keys. Write back.
4. `chmod 600 ~/.claude/channels/line/.env`
5. Confirm.

### `token <value>` — save channel access token

1. Treat the argument as the token (trim whitespace).
2. `mkdir -p ~/.claude/channels/line`
3. Read existing `.env` if present; update/add the
   `LINE_CHANNEL_ACCESS_TOKEN=` line, preserve other keys. Write back.
4. `chmod 600 ~/.claude/channels/line/.env`
5. Confirm, then show the no-args status.

### `port <number>` — set webhook port

1. Update `LINE_WEBHOOK_PORT=` in `.env`. Default is 3100.

### `clear` — remove all credentials

Delete the credential lines (or the file if those are the only lines).

---

## Implementation notes

- The channels dir might not exist if the server hasn't run yet. Missing file
  = not configured, not an error.
- The server reads `.env` once at boot. Credential changes need a session
  restart or `/reload-plugins`. Say so after saving.
- `access.json` is re-read on every inbound message — policy changes via
  `/line:access` take effect immediately, no restart.
- LINE requires HTTPS for webhooks. For local development, suggest ngrok:
  `ngrok http 3100` then set the forwarding URL in LINE Developers Console.
