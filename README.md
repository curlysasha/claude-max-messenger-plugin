# Max Messenger Channel for Claude Code

MCP server that connects Claude Code to [Max messenger](https://max.ru) via the Bot API.

## Features

- **reply** tool — send messages with optional file attachments (max 50MB each)
- **edit_message** tool — update previously sent bot messages (e.g. progress → result)
- Access control with pairing codes, allowlists, and group support
- Photo download and forwarding to Claude

## Limitations

- No message history or search (Max Bot API doesn't support it)
- The bot only sees messages as they arrive

## Setup

### 1. Get a bot token

Open Max messenger → search for **@MasterBot** → create a new bot → copy the token.

### 2. Save the token

```
/max:configure <your-token>
```

### 3. Launch with the channel flag

The channel won't connect without this flag. Exit your session and start a new one.

Since channels are in **research preview** and only official Anthropic plugins are on the approved allowlist, use the development flag for local plugins:

```sh
claude --dangerously-load-development-channels plugin:max@local-plugins
```

### 4. Pair yourself

DM your bot in Max. It will reply with a pairing code. Approve it:

```
/max:access pair <code>
```

### 5. Lock down access (recommended)

Once everyone is paired, switch to allowlist mode:

```
/max:access policy allowlist
```

## Access management

See [ACCESS.md](./ACCESS.md) for full documentation on access control.

## Environment variables

| Variable | Description |
|---|---|
| `MAX_BOT_TOKEN` | Bot token from @MasterBot (set via `/max:configure`) |
| `MAX_ACCESS_MODE` | Set to `static` to snapshot access at boot (no pairing) |

## State directory

All state is stored in `~/.claude/channels/max/`:

```
~/.claude/channels/max/
├── .env          ← bot token
├── access.json   ← access control config
├── inbox/        ← downloaded photos (temporary)
└── approved/     ← pairing confirmations (consumed by server)
```
