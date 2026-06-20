# 🐉 rde_crew

🔥 OX-NATIVE CREW, GANG & TERRITORY SYSTEM V1.12.0 — Real Freehand Graffiti, Zone Warfare, Zero Poll Loops! 🐉

[![Version](https://img.shields.io/badge/version-1.12.0-red?style=for-the-badge)](https://github.com/RedDragonElite/rde_crew)
[![License](https://img.shields.io/badge/license-RDE%20Black%20Flag%20v6.66-black?style=for-the-badge)](https://github.com/RedDragonElite/rde_crew/blob/main/LICENSE)
[![FiveM](https://img.shields.io/badge/FiveM-Compatible-blue?style=for-the-badge)](https://fivem.net)
[![ox_core](https://img.shields.io/badge/Framework-ox__core-blue?style=for-the-badge)](https://github.com/overextended/ox_core)
[![Price](https://img.shields.io/badge/price-FREE%20FOREVER-brightgreen?style=for-the-badge)](https://github.com/RedDragonElite/rde_crew)
[![Pain](https://img.shields.io/badge/dev%20pain-IMMEASURABLE-8B0000?style=for-the-badge)](#-the-graffiti-saga-a-war-story)

**Crew creation, ranks, permissions, garage, shared stash, declared wars, an in-game polygon zone editor, and real freehand spray-painted territory tags that render as actual 3D objects on actual walls — all statebag-driven, all proximity-loaded, all free.**

> Built on ox_core · ox_lib · ox_target · ox_inventory · oxmysql
>
> Built by Red Dragon Elite | SerpentsByte

---

## 📖 Table of Contents

- [Overview](#-overview)
- [Why RDE Crew?](#-why-rde-crew)
- [Features](#-features)
- [Dependencies](#-dependencies)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Usage & Controls](#-usage--controls)
- [Architecture](#-architecture)
- [Security](#-security)
- [Commands](#-commands)
- [Database](#-database)
- [Performance](#-performance)
- [Troubleshooting](#-troubleshooting)
- [🩸 The Graffiti Saga: A War Story](#-the-graffiti-saga-a-war-story)
- [Changelog](#-changelog)
- [License](#-license)

---

## 🎯 Overview

RDE Crew is a production-grade crew/gang and territory-control system for FiveM servers. Crews are created, ranked, and permissioned through ox_core characters — not a separate identity layer bolted on top. Territory is real, in-game-editable polygon zones, claimed not by an admin spreadsheet but by physically standing at a wall and **freehand-painting your tag with the mouse**, which then renders as an actual flat object stuck to that actual wall, visible to every nearby player, persisted in the database, and contested through declared wars between crews.

Everything syncs through prefixed statebags and a lightweight `GlobalState` index — no polling loops, no per-tick distance checks for things that haven't changed, no broadcasting full payloads to people who don't need them. Heavy data (the actual graffiti artwork) is only ever sent to a client once they're physically close enough to render it.

---

## 🔥 Why RDE Crew?

| Other Crew/Gang Scripts | ✅ RDE Crew |
|---|---|
| Territory tag = a fixed font string stamped on the wall | Real freehand painting — your mouse, your lines, your tag |
| Zones hardcoded as lat/long boxes in a config file | In-game polygon zone editor — walk the perimeter, save, done |
| Every player polls distance-to-everything every tick | Proximity index + statebag reactions — zero work when nothing's nearby |
| Permissions = "admin" or "not admin" | 11 granular permission flags per custom grade, up to 8 grades per crew |
| Territory ownership = whoever last touched a database row, manually | Configurable rule (`last_spray` / `spray_count`), war-gated overspray |
| ESX / QBCore bloat | ox_core only — the future, not the past |
| "Aim and pray" placement UX | Live aim reticle, same pattern as `rde_doors` — see exactly where it lands before you commit |
| Paid or locked down | 100% free forever — RDE Black Flag |

---

## ✨ Features

### 👑 Crew Management

- Full crew lifecycle: create (configurable fee, removed from player's money item), rename, disband
- **11 granular permission flags** per grade: invite, kick, change grade, send announcements, use the shared stash, use the garage, (re)place the stash spot, (re)place the garage spot, change crew colour, declare war, spray territory
- Up to **8 custom grades** beyond the built-in defaults, fully permission-configurable per grade
- Crew invites, member kicking (grade-gated — can't kick someone equal or above you), promote/demote
- Crew-wide announcements and a light internal chat log
- A boss NPC (configurable model/coords/scenario/blip) as a social/admin anchor point

### 🗺️ Territory & Zones

- **In-game polygon zone editor** — no hardcoded vec2 lists required, walk the perimeter and place corners live, persisted to the database, editable and re-editable any time
- Zone ownership computed by a configurable rule: `last_spray` (whoever tagged most recently owns it) or `spray_count` (whichever crew has the most pieces in the zone owns it)
- **War-gated overspray** — an enemy crew can only paint over another crew's owned zone while an active war exists between the two crews (`Config.Territory.requireWarToOverspray`)
- Zone ownership changes broadcast instantly via per-zone statebags (`rde_crew_zone_<name>`) with automatic crew notifications on both sides (taken-over / lost)

### 🎨 Graffiti — Real Freehand Painting

This is the part of the resource that took an entire overnight session to get right, and it shows:

- **Actual freehand drawing**, not a font stamp — an NUI canvas where the player draws with the mouse, picking from whichever spray cans they're actually carrying (server-confirmed) and a cap size (thin/medium/thick, no separate item needed)
- **Admin-extensible color registry** — every spray can color is one config line away (`Config.Territory.sprayCans`), the painting palette builds itself from whatever cans the player is holding, no code changes needed to add a new color
- The finished canvas is captured as a **real PNG with a genuine alpha-transparent background** and rendered as an actual flat 3D object physically stuck to the wall — not a decal hack, not a floating UI sprite, a real prop with real transparency, confirmed via direct file verification before it was ever shipped (see [the saga](#-the-graffiti-saga-a-war-story) for exactly how much that cost)
- **Live aim reticle** — a semi-transparent marker follows your raycast hit point in real time while aiming, showing exactly where the piece will land before you commit, instead of aiming blind
- **Proximity-loaded** — only a lightweight index (id/coords/zone/size, never the actual artwork) is ever broadcast via `GlobalState`. The real PNG is only fetched, once, the instant a client gets within render distance, and the prop despawns again once they leave — territory can hold thousands of pieces server-wide without every client carrying every piece in memory
- A short `progressCircle` mini-game (`WORLD_HUMAN_GRAFFITI` scenario) gates the actual placement — the "physical effort" beat, can't just teleport-paint
- Server-side validation on every submission: image format and size caps, every used can verified against the real can registry (never trusts a client-sent color), zone lookup is always server-computed (never trusts a client-sent zone name)

### ⚔️ Wars

- Crews with the `declare_war` permission can formally declare war on another crew
- Active wars are the sole gate on enemy overspray — no war, no painting over someone else's claimed zone
- War state syncs via prefixed statebags, same pattern as everything else

### 🚗 Garage & 📦 Stash

- Crew-owned vehicle garage with a (re)placeable pickup/dropoff spot
- Shared crew stash (inventory) with its own (re)placeable spot and dedicated permission gate

### 🛠️ Admin Tools

- In-game admin hub for crew management without touching the database by hand
- `/crew_reload` to force a full reload of crews/territory/wars from the database after manual edits

---

## 📦 Dependencies

| Resource | Required | Notes |
|---|---|---|
| `oxmysql` | ✅ Required | Database layer |
| `ox_core` | ✅ Required | Player/character framework |
| `ox_lib` | ✅ Required | Callbacks, notifications, progress bars, input dialogs |
| `ox_target` | ✅ Required | Interaction prompts |
| `ox_inventory` | ✅ Required | Spray cans, crew stash items |

---

## 🚀 Installation

### 1. Clone the repository

```bash
cd resources
git clone https://github.com/RedDragonElite/rde_crew.git
```

### 2. Add to server.cfg

```
ensure oxmysql
ensure ox_lib
ensure ox_core
ensure ox_target
ensure ox_inventory
ensure rde_crew
```

> ⚠️ Order matters. `rde_crew` must start **after** all its dependencies.

### 3. Spray can items

Define each spray can color as a normal `ox_inventory` item (label/weight/image — `rde_crew` doesn't care about any of that), then add **one line** to `Config.Territory.sprayCans` per color. See `ox_inventory_items_SNIPPET.lua` for a ready-made starter set matching the default 10-color registry.

### 4. Database

All tables (`rde_crews`, `rde_crew_members`, `rde_crew_invites`, `rde_crew_announcements`, `rde_crew_messages`, `rde_crew_garage`, `rde_crew_sprays`, `rde_crew_wars`, `rde_crew_territory_zones`) are created automatically on first start. No manual SQL import needed.

### 5. Configure (Optional)

Edit `config.lua` to adjust crew limits, territory rules, the spray-can color registry, and graffiti rendering tuning.

### 6. Restart

```
restart rde_crew
```

---

## ⚙️ Configuration

### Crew Defaults

```lua
Config.Crew = {
    tagMinLen  = 2,  tagMaxLen  = 4,
    nameMinLen = 3,  nameMaxLen = 32,
    maxMembers = 30,
    maxGrades  = 8,
}
Config.CreatePrice = 50000
```

### Territory & Spray Cans

```lua
Config.Territory = {
    sprayPersistDays       = 14,      -- old pieces auto-delete after this many days
    sprayMaxDistance       = 3.0,     -- max raycast distance to a wall
    sprayCans = {
        rde_spraycan_white  = '#FFFFFF',
        rde_spraycan_red    = '#E63946',
        -- ...add a color: one line here, one ox_inventory item, done
    },
    sprayConsume           = true,
    takeoverRule           = 'last_spray',   -- or 'spray_count'
    requireWarToOverspray  = true,
}
```

### Graffiti Rendering

```lua
Config.Graffiti = {
    canvasSize     = { w = 600, h = 450 },   -- NUI drawing resolution
    maxImageBytes  = 500 * 1024,             -- server-side upload size cap
    renderDist     = 35.0,                   -- proximity load distance
    unloadDist     = 45.0,                   -- proximity unload distance (hysteresis)
    propHeadingOffset = 180,                 -- carrier prop's mesh-axis correction
    propZOffset       = 0.15,                -- vertical pivot correction
    propEmbedDepth    = 0.02,                -- anti z-fighting offset into the wall
    debug             = false,               -- verbose F8 logging
}
```

> See `config.lua` for the full, heavily-commented set — every tuning value that exists has a comment explaining exactly why it has the value it does, including several that were earned the hard way (see [the saga](#-the-graffiti-saga-a-war-story)).

---

## 🎮 Usage & Controls

| Action | Trigger |
|---|---|
| Open crew menu | `/crew` |
| Place a graffiti piece | Use a spray can item from inventory (or `/crew_spray`) |
| Aim & confirm placement | Aim at a wall — live reticle shows the landing spot — **[E]** to confirm |
| Cancel placement | **[G]** |
| Open/edit a territory zone | `/crew_zone_editor`, `/crew_zone_new` *(admin)* |
| Cancel a stuck zone edit | `/crew_zone_cancel` |
| Crew admin hub | `/crew_admin` *(admin)* |
| Reload crews/territory/wars from DB | `/crew_reload` *(admin)* |
| Unstuck a hung spray session | `/crew_unstuck` |

---

## 🏗️ Architecture

```
rde_crew/
├── fxmanifest.lua
├── config.lua                    ← all tuning, including graffiti rendering
├── shared/
│   ├── enums.lua                 ← permissions, statebag key prefixes
│   ├── utils.lua
│   └── locale.lua
├── server/
│   ├── database.lua               ← schema + migrations
│   ├── classes/crew.lua
│   ├── modules/
│   │   ├── crews.lua / members.lua / invites.lua / permissions.lua
│   │   ├── territory.lua          ← zones, ownership, graffiti placement
│   │   ├── zone_editor.lua
│   │   ├── wars.lua
│   │   ├── garage.lua / inventory.lua / money.lua / announcements.lua
│   │   └── admin.lua
│   ├── callbacks.lua
│   └── main.lua
├── client/
│   ├── modules/
│   │   ├── spray.lua               ← the graffiti placement + rendering saga lives here
│   │   ├── territory.lua / overlay.lua / zone_editor.lua
│   │   ├── blips.lua / boss_npc.lua / garage.lua / inventory.lua
│   ├── ui.lua / admin.lua / main.lua
├── html/                           ← zone editor + graffiti paint canvas NUI
└── stream/                         ← bzzz_decals addon prop pack (see saga)
```

### Statebags (`rde_crew_` prefix)

| Key | Meaning |
|---|---|
| `rde_crew_<id>` | Crew snapshot |
| `rde_crew_zone_<name>` | `{ ownerCrewId, since }` |
| `rde_crew_index` | Array of all known crew IDs |
| `rde_crew_spray_index` | Lightweight array of graffiti piece metadata (never the artwork) |

---

## 🛡️ Security

- Every spray placement is validated server-side: image format (`data:image/png;base64,...`), size cap, every used can checked against the real can registry — a crafted payload cannot spawn a color that wasn't actually consumed
- Zone lookup at placement time is **always server-computed** from world coordinates — the client's claimed zone name is never trusted
- War-gate enforced server-side before any overspray is accepted
- Permission checks on every crew action against the actor's actual grade, not client-claimed state
- `created_at` is set explicitly via `NOW()` on every spray insert rather than relying on a column default — see the saga for exactly why that mattered

---

## 📋 Commands

| Command | Restricted | Description |
|---|---|---|
| `/crew` | No | Open the crew menu |
| `/crew_spray` | No | Start graffiti placement (normally triggered by using a can item) |
| `/crew_unstuck` | No | Recover from a hung spray session |
| `/crew_zone_editor` | Admin | Open the zone manager |
| `/crew_zone_new` | Admin | Start a new zone polygon |
| `/crew_zone_cancel` | No | Cancel an in-progress zone edit |
| `/crew_admin` | Admin | Open the admin hub |
| `/crew_reload` | Admin | Reload crews/territory/wars from the database |

---

## 🗄️ Database

| Table | Purpose |
|---|---|
| `rde_crews` | Core crew row — name, tag, leader, treasury, grades |
| `rde_crew_members` | Crew membership + grade |
| `rde_crew_invites` | Pending invitations |
| `rde_crew_announcements` | Crew-wide messages |
| `rde_crew_messages` | Light internal chat history |
| `rde_crew_garage` | Stored crew vehicles |
| `rde_crew_sprays` | Graffiti pieces — coords, wall normal, painted-area size, base64 PNG |
| `rde_crew_wars` | Active and historical wars |
| `rde_crew_territory_zones` | In-game-editable zone polygons |

All tables auto-create on first start. `rde_crew_sprays` carries a `created_at` migration that repairs the column default *and* backfills any corrupted timestamps — necessary because the table predates this rewrite (see the saga).

---

## ⚡ Performance

- Graffiti uses a two-tier model: a tiny `GlobalState`-broadcast index (id/coords/zone/size only) drives proximity math client-side; the actual PNG is only ever fetched via on-demand callback once a client is within `Config.Graffiti.renderDist`, and the rendered prop is torn down again past `unloadDist`
- The paint canvas itself bakes finished strokes onto an offscreen buffer once, rather than replaying full stroke history on every mouse movement — cost stays flat regardless of how detailed a piece gets
- Zone ownership recomputation and statebag publishes are event-driven, never polled

---

## 🐛 Troubleshooting

**Graffiti piece doesn't render?**
Confirm `bzzz_decals.ytyp` loaded without a `DLC_ITYP_REQUEST` error in console. Check `Config.Graffiti.debug = true` temporarily for verbose F8 logging of the render pipeline.

**Piece renders at the wrong height/rotation after changing the carrier prop?**
`propHeadingOffset` and `propZOffset` are specific to the *current* carrier model's own mesh-pivot convention — both need re-tuning if you ever swap which prop is used. See the saga for why.

**Two nearby pieces show the same image?**
The render pool (`GRAFFITI_PROP_POOL` in `client/modules/spray.lua`) only has 10 slots — more than 10 simultaneous pieces in render range at once can collide. Extend the pool with more `bzzz_normal_sign_*` letters if your server regularly has dense territory.

**"No permission" on admin commands?**
Verify the actor's ox_core group, not `Config.Admin` alone — confirm your permissions setup matches `Config.Admin.aceGroup`.

---

## 🩸 The Graffiti Saga: A War Story

Every config comment in this resource that starts with "CONFIRMED IN TESTING" or "BUGFIX" is a scar from one overnight session. This is that story, because it was too absurd to just quietly ship and never mention.

**The plan was simple.** Replace the old fixed-text crew-tag stamp with real freehand painting. Player draws on a canvas, art gets projected onto the wall as a decal. A weekend feature, surely.

**Round 1 — `AddDecal`.** GTA's native bullet-hole/scorch-mark system, repurposed for graffiti — exactly how most commercial FiveM graffiti scripts do it. Built the whole pipeline: DUI runtime texture, `PatchDecalDiffuseMap`, `AddDecal` with the wall normal and a computed side vector. In-game test: `IsDecalAlive` returns `true`. A genuinely live decal handle. Nothing visible on the wall. Not faint — nothing. Tried real built-in decal types, tried skipping the custom texture entirely to isolate the variable. Still nothing. Confirmed via community research: a known, unresolved, undocumented-by-Rockstar issue. `AddDecal` was abandoned, never to return (it got one more chance later, just to be thorough — see Round 5).

**Round 2 — A real prop, `AddReplaceTexture`.** Found `prop_ld_filmset` (a flat film-set billboard) via manual OpenIV asset hunting — including one genuinely funny detour where the prop's actual embedded texture name turned out to be `prop_fruit_box_01`, a leftover from Rockstar reusing an old texture slot. `AddReplaceTexture` ran without error. Prop spawned in the right place. Still showed the original art, never the custom texture.

**Round 3 — The actual root cause.** A GitHub issue search turned up `citizenfx/fivem#1684`: **`AddReplaceTexture` does not support "delayed" DUI-handle-sourced textures on any FXServer branch except Release.** This server intentionally runs `latest`. Every single failure up to this point — both the silent `AddDecal` and the silent `AddReplaceTexture` — traced back to the *same* underlying engine bug, because both pipelines sourced their runtime texture from a `CreateDui` call. The fix, straight from the FXServer dev himself: don't source the texture from a DUI at all.

**Round 4 — Rebuilding the pipeline around a PNG, not a browser.** The paint canvas now exports itself directly as a base64 PNG (`canvas.toDataURL`) the moment the player confirms. `CreateRuntimeTextureFromImage` takes that string straight in — confirmed by FiveM's own docs to accept a base64 data URL — no DUI anywhere in the chain. Simpler architecture *and* it sidesteps the bug entirely.

**Round 5 — `AddDecal`, again, now that the DUI excuse was gone.** Worth one more shot with a non-DUI texture source. Same result: `IsDecalAlive == true`, nothing visible. Confirmed, twice now, with two different texture sources: `AddDecal` is just not reliable on this engine/branch, full stop.

**Round 6 — Finding a prop that actually supports live replacement.** Not every screen-type prop does — confirmed via a community forum post describing the *exact* same symptom on a *different* prop (`prop_big_cin_screen` silently failed; `v_ilev_cin_screen`, using the same texture name, worked fine right next to it). Found `v_ilev_cin_screen` / `script_rt_cinscreen` this way — the `script_rt_` prefix itself turned out to be the real signal: Rockstar's own naming convention for script-driven runtime-replaceable texture slots. It worked. It also looked like a cinema screen, because it was one — curved, oversized, comically wrong for a wall tag.

**Round 7 — The database ate its own children.** Mid-investigation into prop sizing, a much bigger problem surfaced: every fresh server boot logged **zero** graffiti pieces loaded — even moments after successfully creating one in the same session. Root cause: `rde_crew_sprays` predates this rewrite, and its `created_at DEFAULT CURRENT_TIMESTAMP` had apparently never been reliably applied on the pre-existing table. An invalid timestamp made the routine 14-day housekeeping cleanup query nuke the *entire table*, every single restart, immediately. Three-layer fix: explicit `NOW()` on every insert, a migration repairing the column default, and a sanity-bounded cleanup query that can never again mass-delete on a bad timestamp.

**Round 8 — A race condition masquerading as a sync bug.** New pieces appeared to require a full resource restart to become visible to other players. Actual cause: the in-flight guard protecting the fetch-and-render sequence was released *before* rendering had actually finished, not after — letting the next proximity tick start a second full render cycle on top of an unfinished one, repeatedly, stacking duplicate objects. Fixed by holding the guard for the full operation, not just the network round-trip.

**Round 9 — Found `prop_tv_flat_01_screen`,** the actual screen-only sub-component of a flat-screen TV model (GTA splits some props into multiple drawables) — confirmed via a real community mod ("Watch Any TV Anywhere") *and* this server's own `rde_oxmedia` resource, which plays live video on the exact same prop. Genuinely flat, no bezel, no depth. Scaling it down via the unofficial `SetEntityMatrix` trick introduced visible parallelogram distortion — confirmed, then abandoned in favor of just not scaling at all.

**Round 10 — Named Render Targets.** Reading `rde_oxmedia`'s actual source revealed it doesn't use `AddReplaceTexture` for its TV at all — it uses `RegisterNamedRendertarget` + `LinkNamedRendertarget` + `DrawSprite`, redrawn every frame, the same official mechanism Rockstar's own in-game screens use. Promising — until testing confirmed the render target is shared **per model hash**, not per entity instance: every simultaneously-loaded piece showed whichever was drawn most recently, globally, every frame. Worse for multi-piece support than `AddReplaceTexture`, and it *still* didn't solve background transparency (a render target is itself an opaque buffer, full stop). Abandoned.

**Round 11 — The actual goal, restated.** The whole point was never "a working prop." It was a player saying, repeatedly and with increasing intensity, *"I just want the ink visible, nothing else."* Every vanilla screen material tested — cinema, TV — is deliberately opaque, because real screens always show *something*. That's not a bug to route around; it's how those materials were built.

**Round 12 — `bzzz_decals`.** A free, third-party addon prop pack, built specifically for retexturing, found via a forum thread describing flat decal props where "on one side there is texture, on the other side you can't see anything." Verified directly in OpenIV with "Alpha background" checked before integrating a single file: a genuine checkerboard transparency pattern around the logo content. The first prop in this entire saga where the material was confirmed alpha-capable *before* testing, instead of discovered opaque after the fact. Integrated as a streamed addon (`DLC_ITYP_REQUEST`), origTxd/origTxn reasoned out from file-size correlation since the loose third-party model didn't auto-resolve in OpenIV's viewer the way vanilla files do. It worked. Real ink, on a real wall, with a real transparent background. No box. No bezel. No DUI. No decal.

**The cleanup.** `AddReplaceTexture`'s override turned out to be global per `(origTxd, origTxn)` — confirmed when re-entering render range started showing whichever *other* piece had patched the shared target most recently. Solved with a 10-slot pool, deterministically assigned by piece ID. The new prop's mesh needed its own rotation correction (90°, then corrected to the actually-right 180° once tested) and its own vertical pivot correction, independent of every value already tuned for the previous prop. The paint canvas itself turned out to have an `O(n)`-per-mouse-move redraw bug, fixed by baking finished strokes onto an offscreen layer once instead of replaying full history on every frame. And finally, the aiming flow itself got upgraded from "aim blind, press E, hope" to a live reticle — the same tennis-ball entity-selection pattern already proven in `rde_doors`.

**Final tally:** three full rendering techniques attempted and two abandoned, one official CitizenFX engine bug identified and worked around, one self-inflicted database table found wiping itself on every boot, two race conditions, one third-party addon pack integrated from scratch, and a great many config values that say "CONFIRMED IN TESTING" because guessing twice was already too many times.

It renders. It's flat. It's transparent. It's real graffiti on a real wall.

---

## 📝 Changelog

### v1.12.0 — Live Aim Reticle
Real-time visual placement feedback, same pattern as `rde_doors`'s entity-selection loop.

### v1.11.x — Tuning + Paint Canvas Performance
Confirmed rotation/position values for the final carrier prop; fixed an `O(n)`-per-stroke canvas redraw bug; 10-slot texture pool to stop simultaneous pieces from sharing one `AddReplaceTexture` target; verbose diagnostic logging gated behind `Config.Graffiti.debug`.

### v1.10.x — bzzz_decals Integration
Pivoted to a free third-party addon prop pack with a confirmed real alpha-blended material after every vanilla screen material tested rendered fully opaque.

### v1.9.0 — Named Render Target Experiment
Tested the mechanism `rde_oxmedia` uses for live video; found it shared per model hash, not per instance — abandoned for graffiti's multi-piece use case.

### v1.7.x–v1.8.x — Root Cause & Recovery
Identified `citizenfx/fivem#1684` (`AddReplaceTexture` + delayed DUI textures) as the actual root cause behind both the `AddDecal` and `AddReplaceTexture` failures; rebuilt the texture pipeline around a direct base64 PNG export with no DUI involved; found and fixed a `created_at` migration bug that was silently wiping the entire `rde_crew_sprays` table on every server restart; found and fixed a render race condition causing duplicate spawned objects.

### v1.5.0–v1.6.x — Freehand Graffiti Rewrite
Replaced the fixed crew-tag text stamp with real freehand NUI painting; multiple rendering technique attempts (`AddDecal`, vanilla `AddReplaceTexture` props).

### Earlier
In-game polygon zone editor, statebag-driven crew/zone sync, war-gated territory overspray, full RDE OX Standards v2 pass.

---

## 📜 License

```
###################################################################################
# .:: RED DRAGON ELITE (RDE) - BLACK FLAG SOURCE LICENSE v6.66 ::.
# PROJECT: RDE_CREW v1.12.0 (OX-NATIVE CREW, GANG & TERRITORY SYSTEM)
# ARCHITECT: .:: RDE ⧌ Shin [△ ᛋᛅᚱᛒᛅᚾᛏᛋ ᛒᛁᛏᛅ ▽] ::. | https://rd-elite.com
# ORIGIN: https://github.com/RedDragonElite
# WARNING: THIS CODE IS PROTECTED BY DIGITAL VOODOO, PURE HATRED FOR LEAKERS,
#          AND THE COLLECTIVE TRAUMA OF ONE VERY LONG NIGHT WITH ADDDECAL.
#
# [ THE RULES OF THE GAME ]
#
# 1. // THE "FUCK GREED" PROTOCOL (FREE USE)
#    You are free to use, edit, and abuse this code on your server.
#    Learn from it. Break it. Fix it. That is the hacker way.
#    Cost: 0.00€. If you paid for this, you got scammed by a rat.
#
# 2. // THE TEBEX KILL SWITCH (COMMERCIAL SUICIDE)
#    Listen closely, you parasites:
#    If I find this script on Tebex, Patreon, or in a paid "Premium Pack":
#    > I will DMCA your store into oblivion.
#    > I will publicly shame your community.
#    > I hope every wall you ever try to tag rejects your AddReplaceTexture call.
#    SELLING FREE WORK IS THEFT. AND I AM THE JUDGE.
#
# 3. // THE CREDIT OATH
#    Keep this header. If you remove my name, you admit you have no skill.
#    You can add "Edited by [YourName]", but never erase the original creator.
#    Don't be a skid. Respect the architecture. Respect the pain it took.
#
# 4. // THE CURSE OF THE COPY-PASTE
#    This code uses statebags, proximity streaming, a custom addon prop pack,
#    and a runtime texture pipeline that survived three abandoned rendering
#    techniques and one CitizenFX engine bug. If you just copy-paste without
#    reading, it WILL break. Don't come crying to my DMs. RTFM or learn to code.
#
# --------------------------------------------------------------------------
# "We build the future on the graves of paid resources — and on the graves
#  of every rendering technique that didn't make it."
# "REJECT MODERN MEDIOCRITY. EMBRACE RDE SUPERIORITY."
# --------------------------------------------------------------------------
###################################################################################
```

**TL;DR:**

- ✅ Free forever — use it, edit it, learn from it
- ✅ Keep the header — credit where it's due
- ❌ Don't sell it — commercial use = instant DMCA
- ❌ Don't be a skid — copy-paste without reading won't work anyway

---

## 🌐 Community & Support

| | |
|---|---|
| 🐙 GitHub | [RedDragonElite](https://github.com/RedDragonElite) |
| 🌍 Website | [rd-elite.com](https://rd-elite.com) |
| 🎮 RDE Props | [rde_props](https://github.com/RedDragonElite/rde_props) |
| 🚪 RDE Doors | [rde_doors](https://github.com/RedDragonElite/rde_doors) |
| 🅿️ RDE Parking | [rde_parking](https://github.com/RedDragonElite/rde_parking) |
| 📺 RDE OxMedia | [rde_oxmedia](https://github.com/RedDragonElite/rde_oxmedia) |
| 🚨 RDE AIPD | [rde_aipd](https://github.com/RedDragonElite/rde_aipd) |

When asking for help, always include:
- Full error from server console or txAdmin
- Your `server.cfg` resource start order
- ox_core / ox_lib versions in use

---

> *"It renders. It's flat. It's transparent. It's real graffiti on a real wall.
> Ask us what that cost."*
>
> **REJECT MODERN MEDIOCRITY. EMBRACE RDE SUPERIORITY.**

[![Website](https://img.shields.io/badge/Website-Visit-red?style=for-the-badge&logo=google-chrome)](https://rd-elite.com)
[![Nostr](https://img.shields.io/badge/Nostr-Follow-purple?style=for-the-badge&logo=rss)](https://primal.net/p/npub1wr4e24zn6zzjqx8kvnelfvktf0pu6l2gx4gvw06zead2eqyn23sq9tsd94)

🐉 *Made with 🔥 (and a genuinely heroic amount of stubbornness) by Red Dragon Elite*

[⬆ Back to Top](#-rde_crew)
