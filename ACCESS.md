# Access Control

Access is configured in `~/.claude/channels/max/access.json`.

Example:

```jsonc
{
  "dmPolicy": "pairing",       // "pairing" | "allowlist" | "disabled"
  "allowFrom": ["1234567890"], // dialog chat IDs allowed to reach Claude
  "groups": {
    "9876543210": {            // group chat ID
      "requireMention": true,  // only respond when @botname is mentioned
      "allowFrom": []          // empty = all group members allowed
    }
  },
  "mentionPatterns": ["^hey claude\\b"],  // extra regex triggers for groups
  "textChunkLimit": 4096,      // max chars per message (Max hard limit)
  "chunkMode": "newline"       // "length" | "newline" (split on paragraphs)
}
```

## How pairing works

1. Someone DMs your bot in Max
2. The bot replies with a 6-char code: `Pairing required — run in Claude Code: /max:access pair abc123`
3. You run `/max:access pair abc123` in your terminal
4. Their dialog chat ID is added to `allowFrom`
5. They receive "Paired! Say hi to Claude."

## DM policies

- **pairing** — unknown users get a code to approve (default, temporary)
- **allowlist** — only `allowFrom` IDs can reach Claude (recommended)
- **disabled** — DMs are completely ignored

## Groups

Groups are disabled by default. Enable with `/max:access group add <chatId>`.
The group chat ID can be found in the incoming `<channel>` tag when someone
messages the bot in a group.

## Managing access

```
/max:access                    — show current status
/max:access pair <code>        — approve a pairing request
/max:access deny <code>        — reject a pairing request
/max:access allow <chatId>     — add a chat ID directly
/max:access remove <chatId>    — revoke access
/max:access policy allowlist   — lock down to allowlist only
/max:access group add <chatId> — enable a group chat
/max:access group rm <chatId>  — disable a group chat
```
