# MdwnhMembers

Single source of truth for the **مدونة ستوديو** team roster — names, avatars, Discord UIDs, emails, gender, roles.

Consumed **live at runtime** by:

| Site | Repo | Uses |
|------|------|------|
| MdwnhLibrary | `YoussefDOT/MdwnhLibrary` | roster + avatars |
| MdwnhPoints | `YoussefDOT/MdwnhPoints` | avatars + gender (صحبة الفجر) |
| MdwnhYearPrepare | `iioiiioii99909-commits/MdwnhYearPrepare` | roster + avatars + gender |
| MdwnhCafe | `YoussefDOT/MaqrMdwnh` | Discord-UID → member lookup |

## Consuming it

```js
const ROSTER_BASE = 'https://raw.githubusercontent.com/mdwnstudio/MdwnhMembers/main';
const roster = await fetch(ROSTER_BASE + '/members.json').then(r => r.json());
// avatar url for a member:  ROSTER_BASE + '/' + roster.avatarBase + '/' + member.slug + '.png'
```

Repo is **public** so the static sites can fetch it directly (raw.githubusercontent sends `access-control-allow-origin: *`). No tokens in client code.

## Editing the roster

Everything lives in [`members.json`](members.json). To…

- **Add a member:** add an object to `members[]` + drop their picture at `avatars/<slug>.png`.
- **Change details:** edit their object (name, email, gender, discordId…).
- **Remove:** delete their object (optionally keep the avatar).
- **Deactivate (keep data, hide from sites):** set `"active": false`.

Then:

```bash
git add -A && git commit -m "roster: update" && git push
```

Sites pick up the change within minutes (raw CDN cache ~5 min). **No site redeploy needed.**

### Field reference

See `_fields` inside `members.json`. Key gotchas:

- `dbKey` is **byte-matched** against the Points Firebase database. Never "correct" its spelling.
- `slug` must match the avatar filename under `avatars/`.
- `admin: true` → the leader (نواف), special access.
- `dummy: true` → test profile (سراج), not a real member.
- `discordId` is a **string** to preserve integer precision.
- `telegramHandle` → `@username` (no `@`), for `t.me/<handle>` links. `null` if the member has no Telegram username set.
- `telegramName` → fallback Telegram **display name** to search by when `telegramHandle` is `null`. `null` if not applicable (e.g. the dummy profile).

## Grabbing this into a site (agent prompt)

To wire a new (or existing) site up to read this roster live, paste this to an agent working in that site's repo:

```
This site needs to consume the team roster from the MdwnhMembers repo instead of hardcoding member data.

Source of truth: https://raw.githubusercontent.com/mdwnstudio/MdwnhMembers/main/members.json
Avatars: https://raw.githubusercontent.com/mdwnstudio/MdwnhMembers/main/avatars/<slug>.png

Fetch members.json at runtime (it's public, CORS-enabled via raw.githubusercontent.com — no auth/token needed).
Read the top-level "_fields" object in the JSON for the meaning of every member field (name, dbKey, discordId,
email, gender, admin, active, dummy, telegramHandle, telegramName, etc.) before mapping it into this site's
data model — don't guess field semantics.

Filter out members with "dummy": true (test profile) from any user-facing roster.
Respect "active": false — hide inactive members from live rosters unless the feature explicitly needs
historical/inactive members too.

For each member's Telegram handle, use "telegramHandle" for a direct t.me/<handle> link/mention when present;
when it's null, fall back to showing "telegramName" (a display name to search by) instead — do not fabricate
a handle.

Do not hardcode or duplicate member data in this repo going forward — always read it from members.json at
runtime so edits to the roster propagate without a redeploy.
```
