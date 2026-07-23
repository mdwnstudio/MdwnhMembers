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
