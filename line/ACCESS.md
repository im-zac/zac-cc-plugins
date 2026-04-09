# LINE — Access & Delivery

A LINE bot is addressable by anyone who adds it as a friend. Without a gate, those messages would flow straight into your assistant session. The access model described here decides who gets through.

By default, a DM from an unknown sender triggers **pairing**: the bot replies with a 6-character code and drops the message. You run `/line:access pair <code>` from your assistant session to approve them. Once approved, their messages pass through.

All state lives in `~/.claude/channels/line/access.json`. The `/line:access` skill commands edit this file; the server re-reads it on every inbound message, so changes take effect without a restart. Set `LINE_ACCESS_MODE=static` to pin config to what was on disk at boot (pairing is unavailable in static mode since it requires runtime writes).

## At a glance

| | |
| --- | --- |
| Default policy | `pairing` |
| Sender ID | LINE user ID (e.g. `U1234567890abcdef0123456789abcdef`) |
| Group key | Group ID (`C...`) or Room ID (`R...`) |
| Config file | `~/.claude/channels/line/access.json` |

## DM policies

`dmPolicy` controls how DMs from senders not on the allowlist are handled.

| Policy | Behavior |
| --- | --- |
| `pairing` (default) | Reply with a pairing code, drop the message. Approve with `/line:access pair <code>`. |
| `allowlist` | Drop silently. No reply. |
| `disabled` | Drop everything, including allowlisted users and groups. |

```
/line:access policy allowlist
```

## User IDs

LINE identifies users by **opaque string IDs** like `U1234567890abcdef0123456789abcdef`. These are stable per-bot (a user has a different ID per LINE Official Account). The allowlist stores these IDs.

Pairing captures the ID automatically. There's no public way for users to look up their own LINE user ID — pairing is the recommended flow.

```
/line:access allow U1234567890abcdef0123456789abcdef
/line:access remove U1234567890abcdef0123456789abcdef
```

## Groups

Groups are off by default. Opt each one in individually.

```
/line:access group add C1234567890abcdef0123456789abcdef
/line:access group rm C1234567890abcdef0123456789abcdef
```

## Reply tokens vs push messages

LINE has two message-sending mechanisms:

- **Reply** (free, unlimited): Uses a reply token from the webhook event. Token expires ~1 minute after the event. The plugin caches these automatically.
- **Push** (monthly quota): Sends to any user at any time. Free tier: 200/month. The plugin falls back to push when the reply token expires.

The reply tool tries the cached reply token first. If expired, it uses push. The push tool always uses push.

## Webhook setup

LINE requires HTTPS for webhooks. The server listens on `LINE_WEBHOOK_PORT` (default 3100).

For local development:
```bash
ngrok http 3100
```

Then set the webhook URL in LINE Developers Console:
```
https://<your-ngrok-domain>/webhook
```

## Environment variables

| Variable | Required | Description |
| --- | --- | --- |
| `LINE_CHANNEL_SECRET` | Yes | Channel secret from LINE Developers Console |
| `LINE_CHANNEL_ACCESS_TOKEN` | Yes | Long-lived channel access token |
| `LINE_WEBHOOK_PORT` | No | HTTP port for webhook (default: 3100) |
| `LINE_STATE_DIR` | No | Override state directory |
| `LINE_ACCESS_MODE` | No | Set to `static` to freeze config at boot |

Store credentials in `~/.claude/channels/line/.env`:
```
LINE_CHANNEL_SECRET=your_channel_secret
LINE_CHANNEL_ACCESS_TOKEN=your_channel_access_token
```
