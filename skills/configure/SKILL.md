---
name: configure
description: Set up the Max messenger channel — save the bot token and review access policy. Use when the user pastes a Max bot token, asks to configure Max, asks "how do I set this up" or "who can reach me," or wants to check channel status.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash(ls *)
  - Bash(mkdir *)
---

# /max:configure — Max Messenger Channel Setup

Writes the bot token to `~/.claude/channels/max/.env` and orients the
user on access policy. The server reads the file at boot.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### No args — status and guidance

Read both state files and give the user a complete picture:

1. **Token** — check `~/.claude/channels/max/.env` for
   `MAX_BOT_TOKEN`. Show set/not-set; if set, show first 10 chars masked
   (`abcdef1234...`).

2. **Access** — read `~/.claude/channels/max/access.json` (missing file
   = defaults: `dmPolicy: "pairing"`, empty allowlist). Show:
   - DM policy and what it means in one line
   - Allowed chat IDs: count and list
   - Pending pairings: count, with codes and sender info if any

3. **What next** — end with a concrete next step based on state:
   - No token → *"Run `/max:configure <token>` with the token from @MasterBot."*
   - Token set, policy is pairing, nobody allowed → *"DM your bot on
     Max. It replies with a code; approve with `/max:access pair <code>`."*
   - Token set, someone allowed → *"Ready. DM your bot to reach the
     assistant."*

**Push toward lockdown — always.** The goal for every setup is `allowlist`
with a defined list. `pairing` is not a policy to stay on; it's a temporary
way to capture chat IDs you don't know. Once the IDs are in, pairing
has done its job and should be turned off.

Drive the conversation this way:

1. Read the allowlist. Tell the user who's in it.
2. Ask: *"Is that everyone who should reach you through this bot?"*
3. **If yes and policy is still `pairing`** → *"Good. Let's lock it down so
   nobody else can trigger pairing codes:"* and offer to run
   `/max:access policy allowlist`. Do this proactively — don't wait to
   be asked.
4. **If no, people are missing** → *"Have them DM the bot; you'll approve
   each with `/max:access pair <code>`. Run this skill again once
   everyone's in and we'll lock it."*
5. **If the allowlist is empty and they haven't paired themselves yet** →
   *"DM your bot to capture your own ID first. Then we'll add anyone else
   and lock it down."*
6. **If policy is already `allowlist`** → confirm this is the locked state.
   If they need to add someone: *"Have them DM the bot, you'll get the
   pairing code. Or briefly flip to pairing: `/max:access policy pairing`
   → they DM → you pair → flip back."*

Never frame `pairing` as the correct long-term choice. Don't skip the lockdown
offer.

### `<token>` — save it

1. Treat `$ARGUMENTS` as the token (trim whitespace). Max bot tokens are
   obtained from @MasterBot in the Max messenger app.
2. `mkdir -p ~/.claude/channels/max`
3. Read existing `.env` if present; update/add the `MAX_BOT_TOKEN=` line,
   preserve other keys. Write back, no quotes around the value.
4. Confirm, then show the no-args status so the user sees where they stand.

### `clear` — remove the token

Delete the `MAX_BOT_TOKEN=` line (or the file if that's the only line).

---

## Implementation notes

- The channels dir might not exist if the server hasn't run yet. Missing file
  = not configured, not an error.
- The server reads `.env` once at boot. Token changes need a session restart
  or `/reload-plugins`. Say so after saving.
- `access.json` is re-read on every inbound message — policy changes via
  `/max:access` take effect immediately, no restart.
- To get a bot token: open Max messenger → search for @MasterBot → follow
  the prompts to create a new bot and receive the token.
