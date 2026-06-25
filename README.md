# 🐉 rde_crew

🔥 OX-NATIVE CREW, GANG & TERRITORY SYSTEM V2.0.1 — Real Freehand Graffiti, Live Minimap DUI Pipe, Zero Poll Loops! 🐉

[![Version](https://img.shields.io/badge/version-2.0.1-red?style=for-the-badge)](https://github.com/RedDragonElite/rde_crew)
[![License](https://img.shields.io/badge/license-RDE%20Black%20Flag%20v6.66-black?style=for-the-badge)](https://github.com/RedDragonElite/rde_crew/blob/main/LICENSE)
[![FiveM](https://img.shields.io/badge/FiveM-Compatible-blue?style=for-the-badge)](https://fivem.net)
[![ox_core](https://img.shields.io/badge/Framework-ox__core-blue?style=for-the-badge)](https://github.com/overextended/ox_core)
[![Price](https://img.shields.io/badge/price-FREE%20FOREVER-brightgreen?style=for-the-badge)](https://github.com/RedDragonElite/rde_crew)
[![Pain](https://img.shields.io/badge/dev%20pain-IMMEASURABLE-8B0000?style=for-the-badge)](#-the-graffiti-saga-a-war-story)

**Crew creation, ranks, permissions, declared wars, an in-game polygon zone editor, and real freehand spray-painted territory tags that render as actual 3D objects on actual walls — drawn with your mouse, positioned on the wall by hand, statebag-driven, proximity-loaded, and free.**

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b50c0d5c-166c-43fe-adec-f2c849f06a3b" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1752cc2e-5a97-49ef-8e67-7a6754b2808f" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d78be03d-4a9e-4c94-a7a4-be0823d98cf4" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6fa9ac5f-e5c3-4e2b-9183-da42e7e564b3" />

> Built on ox_core · ox_lib · ox_target · oxmysql
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

RDE Crew is a production-grade crew/gang and territory-control system for FiveM servers. Crews are created, ranked, and permissioned through ox_core characters — not a separate identity layer bolted on top. Territory is real, in-game-editable polygon zones, claimed not by an admin spreadsheet but by physically standing at a wall, **freehand-painting your tag with the mouse**, then **rotating and scaling it directly on the wall surface** before confirming placement. The result renders as an actual flat object stuck to that actual wall — visible to every nearby player, persisted in the database, contested through declared wars.

Everything syncs through prefixed statebags and a lightweight `GlobalState` index. No polling loops. No per-tick distance checks for things that haven't changed. No broadcasting full payloads to people who don't need them. Heavy data (the actual graffiti artwork) is only ever sent to a client once they're physically close enough to render it.

---

## 🔥 Why RDE Crew?

| Other Crew/Gang Scripts | ✅ RDE Crew |
|---|---|
| Territory tag = a fixed font string stamped on the wall | Real freehand painting — your mouse, your lines, your tag |
| Placement = aim blind and press E, hope for the best | **Two-screen NUI flow**: draw first, then position the finished piece on the wall via mouse before confirming |
| Zones hardcoded as lat/long boxes in a config file | In-game polygon zone editor — walk the perimeter, save, done |
| Every player polls distance-to-everything every tick | Proximity index + statebag reactions — zero work when nothing changes |
| Permissions = "admin" or "not admin" | 9 granular permission flags per custom grade, up to 8 grades per crew |
| Minimap zone colors update on reconnect or restart | **Live DUI pipe** — zone turns crew color the instant the spray is confirmed, for every player simultaneously |
| Prop wall placement = random rotation, hope it looks ok | Full orthonormal `SetEntityMatrix` via `cross(n, worldUp)` — flush on ANY wall, any angle, automatic |
| Garage/stash bolted on for feature count | Vehicle storage → `rde_parking`. Prop interaction → `rde_props_interact`. One job done well. |
| ESX / QBCore bloat | ox_core only — the future, not the past |
| Paid or locked down | 100% free forever — RDE Black Flag |

---

## ✨ Features

### 👑 Crew Management

- Full crew lifecycle: create (configurable fee), rename, disband
- **9 granular permission flags** per grade: invite, kick, change grade, send announcements, (re)place boss NPC, change crew colour, declare war, spray territory, access admin tools
- Up to **8 custom grades** fully permission-configurable
- Crew invites, member kicking (grade-gated — can't kick someone equal or above you), promote/demote
- Crew-wide announcements and a light internal chat log
- A boss NPC (configurable model/coords/scenario/blip) as a social/admin anchor point

### 🗺️ Territory & Zones

- **In-game polygon zone editor** — walk the perimeter, place corners live, persist to DB, edit any time
- Zone ownership computed by a configurable rule: `last_spray` or `spray_count`
- **War-gated overspray** — an enemy crew can only paint over your owned zone while an active war exists between both crews
- Zone ownership changes broadcast instantly via per-zone statebags with automatic crew notifications on both sides
- **Live DUI minimap pipe** — when ownership changes, `SendDuiMessage` updates the existing zone DUI canvas with the new crew RGB. `CreateRuntimeTextureFromDuiHandle` is a live link, not a snapshot, so the minimap texture updates in the same frame. No TxD rebuild, no DUI destroy/recreate cycle, zero delay.

### 🎨 Graffiti — Real Freehand Painting with Mouse Placement

This is the part of the resource that took an entire overnight session to get right. Here is what that session produced:

**Screen 1 — Draw.**
An NUI canvas where the player draws freehand with the mouse, picking from whichever spray cans they're actually carrying (server-confirmed at open time) and a **cap type** from a palette of 7 authentic graffiti caps. Every finished stroke is baked once onto an offscreen buffer — no full-history replay on every mouse move, cost stays flat no matter how detailed the piece gets. Press **Z** to undo the last stroke. When satisfied: **Platzieren →**.

**Cap System — 7 authentic nozzle types:**

| Cap | Width | Edge |
|---|---|---|
| Skinny | 2px | Crisp — hairline precision |
| Soft Cap | 5px | Feathered edges |
| NY Cap | 8px | Crisp — the classic outline cap |
| Standard | 12px | Slightly soft — universal |
| Fat Cap | 20px | Feathered — fills |
| Ultra Fat | 32px | Heavy feathering |
| Ultrawide | 50px | Maximum spread — banana cap |

Fat caps use canvas `shadowBlur` matching the stroke color for authentic spray-paint feathering, not a glow effect. Skinny, NY Cap, and Standard remain hard-edged. Each cap button shows a live mini-stroke preview with real blur so the player knows exactly what they're picking before touching the canvas.

**Screen 2 — Place.**
The finished piece is shown on a simulated wall surface. The player rotates and scales it before confirming:

| Input | Action |
|---|---|
| LMB drag | Rotate |
| RMB drag | Scale |
| Scroll wheel | Rotate |
| Arrow keys ← → | Rotate |
| Arrow keys ↑ ↓ | Scale |
| **Sprühen ✓** | Confirm placement |
| **← Zurück** | Back to drawing |

The chosen rotation and scale are sent back to Lua. A full orthonormal basis (`forward = wall_normal`, `right = cross(n, worldUp)`, `up = cross(right, n)`) is computed and applied via `SetEntityMatrix` — the prop sits flush on any wall surface regardless of orientation, angle, or texture pattern. No heading-only approximation, no manual offset guessing.

**Rendering.**
The finished canvas exports as a real PNG with genuine alpha transparency. `CreateRuntimeTextureFromImage` applies it directly to the `bzzz_decals` addon carrier prop — no DUI, no decal hack, no floating UI sprite, a real flat 3D object with real transparency physically stuck to the wall, visible to every player in range.

**Proximity streaming.**
Only a lightweight index (id/coords/zone/size) is ever broadcast via `GlobalState`. The real PNG is only fetched the instant a client enters `Config.Graffiti.renderDist`. The prop despawns again past `unloadDist`. Thousands of server-wide pieces, zero per-tick overhead for distant clients.

**Mini-game gate.**
A short `progressCircle` (`WORLD_HUMAN_GRAFFITI` scenario) runs after the placement screen — the "physical effort" beat — before the server accepts the submission.

### ⚔️ Wars

- Crews with the `declare_war` permission can formally declare war on another crew
- Active wars are the sole gate on enemy overspray
- War state syncs via prefixed statebags

### 🛠️ Admin Tools

- In-game admin hub for crew management without touching the database
- `/crew_reload` to force a full DB reload after manual edits

> **Vehicle storage** is handled by `rde_parking`. **Prop interaction / shared stash** by `rde_props_interact`. Both do their one job correctly. RDE Crew does its one job — crews, territory, graffiti.

---

## 📦 Dependencies

| Resource | Required | Notes |
|---|---|---|
| `oxmysql` | ✅ Required | Database layer |
| `ox_core` | ✅ Required | Player/character framework |
| `ox_lib` | ✅ Required | Callbacks, notifications, progress bars |
| `ox_target` | ✅ Required | Interaction prompts |
| `bzzz_decals` | ✅ Required | Addon prop pack — flat alpha-capable carrier props for graffiti rendering (streamed in `rde_crew/stream/`) |

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
ensure rde_crew
```

> ⚠️ Order matters. `rde_crew` must start **after** all its dependencies.

### 3. Spray can items

Define each spray can color as a normal `ox_inventory` item (label/weight/image), then add **one line** to `Config.Territory.sprayCans` per color. See `ox_inventory_items_SNIPPET.lua` for a ready-made starter set matching the default 10-color palette.

### 4. Database

All tables are created automatically on first start. No manual SQL import needed. The `rde_crew_sprays` table gains a `rotation FLOAT DEFAULT 0.0` column automatically via `ADD COLUMN IF NOT EXISTS` — safe to run against any existing installation.

### 5. Configure (optional)

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
    sprayPersistDays       = 14,
    sprayMaxDistance       = 3.0,
    sprayCans = {
        rde_spraycan_white  = '#FFFFFF',
        rde_spraycan_red    = '#E63946',
        -- one line here + one ox_inventory item = new color, no code changes
    },
    sprayConsume           = true,
    takeoverRule           = 'last_spray',   -- or 'spray_count'
    requireWarToOverspray  = true,
}
```

> ⚠️ `sprayPersistDays` feeds a cleanup query that runs on every boot. A value of `0` will delete everything. Keep it at a sensible number.

### Graffiti Rendering

```lua
Config.Graffiti = {
    canvasSize        = { w = 600, h = 450 },
    maxImageBytes     = 500 * 1024,
    renderDist        = 35.0,
    unloadDist        = 45.0,
    propHeadingOffset = 180,   -- mesh-axis correction for bzzz_decals carrier
    propZOffset       = 0.15,
    propEmbedDepth    = 0.02,  -- anti z-fighting offset
    areaSize          = { w = 2.4, h = 1.8 },   -- default prop world size
    debug             = false,
}
```

> See `config.lua` for the full heavily-commented set — every value that exists has a comment explaining why it is what it is.

---

## 🎮 Usage & Controls

| Action | Trigger |
|---|---|
| Open crew menu | `/crew` |
| Start painting | Use a spray can item from inventory (or `/crew_spray`) |
| Aim at wall | Live reticle shows the landing spot |
| Open paint canvas | **[E]** while aimed at a valid wall |
| Cancel aiming | **[G]** |
| Undo last stroke (draw screen) | **Z** |
| Rotate tag (placement screen) | LMB drag / Scroll / ← → |
| Scale tag (placement screen) | RMB drag / ↑ ↓ |
| Confirm placement | **Sprühen ✓** button |
| Back to drawing | **← Zurück** button |
| Open zone editor | `/crew_zone_editor` *(admin)* |
| New zone polygon | `/crew_zone_new` *(admin)* |
| Cancel zone edit | `/crew_zone_cancel` |
| Admin hub | `/crew_admin` *(admin)* |
| Reload DB | `/crew_reload` *(admin)* |
| Unstuck hung spray session | `/crew_unstuck` |

---

## 🏗️ Architecture

```
rde_crew/
├── fxmanifest.lua
├── config.lua
├── shared/
│   ├── enums.lua          ← permissions, statebag key prefixes
│   ├── utils.lua
│   └── locale.lua
├── server/
│   ├── database.lua        ← schema + auto-migrations
│   ├── classes/crew.lua
│   ├── modules/
│   │   ├── crews.lua / members.lua / invites.lua / permissions.lua
│   │   ├── territory.lua   ← zones, ownership, graffiti placement
│   │   ├── zone_editor.lua
│   │   ├── wars.lua
│   │   ├── money.lua / announcements.lua / admin.lua
│   ├── callbacks.lua
│   └── main.lua
├── client/
│   ├── modules/
│   │   ├── spray.lua       ← placement flow, reticle, rendering
│   │   ├── territory.lua / overlay.lua / zone_editor.lua
│   │   ├── blips.lua / boss_npc.lua
│   ├── ui.lua / admin.lua / main.lua
├── html/
│   ├── graffiti_paint.html  ← two-screen NUI (draw + place)
│   ├── graffiti_paint.js
│   └── graffiti_paint.css
└── stream/                  ← bzzz_decals addon prop pack
```

### Statebags (`rde_crew_` prefix)

| Key | Meaning |
|---|---|
| `rde_crew_<id>` | Crew snapshot |
| `rde_crew_zone_<name>` | `{ ownerCrewId, since }` |
| `rde_crew_index` | Array of all known crew IDs |
| `rde_crew_spray_index` | Lightweight graffiti metadata (never the artwork) |

---

## 🛡️ Security

- Every spray placement validated server-side: image format, size cap, every used can verified against the real can registry — crafted payloads cannot inject invalid colors
- Zone lookup at placement time is always server-computed — client-claimed zone names are never trusted
- War-gate enforced server-side before any overspray is accepted
- `rotation` and `scale` values from the NUI are range-clamped server-side before being written to the DB
- Permission checks on every crew action against the actor's actual grade, not client-claimed state
- `created_at` set explicitly via `NOW()` on every spray insert rather than relying on a column default (see saga for why this mattered)

---

## 📋 Commands

| Command | Restricted | Description |
|---|---|---|
| `/crew` | No | Open the crew menu |
| `/crew_spray` | No | Start graffiti placement |
| `/crew_unstuck` | No | Recover from a hung spray session |
| `/crew_zone_editor` | Admin | Open zone manager |
| `/crew_zone_new` | Admin | Start a new zone polygon |
| `/crew_zone_cancel` | No | Cancel in-progress zone edit |
| `/crew_admin` | Admin | Open admin hub |
| `/crew_reload` | Admin | Reload from database |

---

## 🗄️ Database

| Table | Purpose |
|---|---|
| `rde_crews` | Core crew row — name, tag, leader, treasury, grades, base coords |
| `rde_crew_members` | Membership + grade |
| `rde_crew_invites` | Pending invitations |
| `rde_crew_announcements` | Crew-wide messages |
| `rde_crew_messages` | Internal chat history |
| `rde_crew_sprays` | Graffiti pieces — coords, wall normal, base64 PNG, rotation, scale |
| `rde_crew_wars` | Active and historical wars |
| `rde_crew_territory_zones` | In-game-editable zone polygons |

All tables auto-create on first start. `rde_crew_sprays` carries a `created_at` migration that repairs the column default and backfills corrupted timestamps, plus a `rotation` column added via `ADD COLUMN IF NOT EXISTS` — safe against any existing installation.

---

## ⚡ Performance

**Graffiti two-tier model:** a tiny `GlobalState`-broadcast index (id/coords/zone/size only) drives proximity math client-side. The actual PNG is only ever fetched via on-demand callback once a client enters `renderDist`, and the rendered prop tears down past `unloadDist`. Thousands of server-wide pieces, zero per-tick overhead for distant clients.

**Paint canvas — O(1) per mouse move:** finished strokes are baked once onto an offscreen buffer. On `pointermove`, only the current new segment is drawn directly to the visible canvas — no `clearRect`, no history replay, no `redraw()`. A piece with 500 completed strokes costs the same to draw the 501st as the 1st. On a long session (80+ pieces tested) the canvas stays fully responsive regardless of total stroke count.

**Zone ownership and statebag publishes** are event-driven, never polled.

---

## 🐛 Troubleshooting

**Graffiti piece doesn't render?**
Check console for `DLC_ITYP_REQUEST` errors — `bzzz_decals.ytyp` must load clean. Set `Config.Graffiti.debug = true` for full F8 render pipeline logging per piece.

**bzzz_decals not found?**
The prop pack is streamed from `rde_crew/stream/` — no separate download needed. If the ytyp error persists, confirm `rde_crew` is started fully before any script that might reference the model hashes.

**Piece renders at wrong height or rotation after swapping carrier props?**
`propHeadingOffset` and `propZOffset` are specific to the current carrier model's mesh-pivot convention. Both need re-tuning if you change which prop is used. Don't change the prop unless you know exactly which values to adjust — see the saga for why.

**Two nearby pieces show the same image?**
The render pool has 10 slots. More than 10 simultaneous pieces in render range can collide. Extend `GRAFFITI_PROP_POOL` in `client/modules/spray.lua` with more `bzzz_normal_sign_*` letters if your server regularly has dense territory clusters.

**Placement screen rotation/scale not sticking?**
Confirm the NUI callback for `graffiti:confirm` is firing — check F8 for the server-side `graffiti: stored Sprays[n]` log line. If missing, the `rotation`/`scale` fields weren't received. Ensure you're on v2.0.0+ of both `spray.lua` and `graffiti_paint.js`.

**"No permission" on admin commands?**
Verify the actor's ox_core ace group matches `Config.Admin.aceGroup` — not just the in-game grade.

---

## 🩸 The Graffiti Saga: A War Story

Every config comment in this resource that starts with "CONFIRMED IN TESTING" or "BUGFIX" is a scar from one overnight session. This is that story, because it was too absurd to quietly ship without documenting.

**The plan was simple.** Replace the old fixed-text crew-tag stamp with real freehand painting. Player draws on a canvas, art gets projected onto the wall as a decal. A weekend feature, surely.

**Round 1 — `AddDecal`.** GTA's native bullet-hole/scorch-mark system, repurposed for graffiti — exactly how most commercial FiveM graffiti scripts do it. Built the whole pipeline: DUI runtime texture, `PatchDecalDiffuseMap`, `AddDecal` with the wall normal and a computed side vector. In-game test: `IsDecalAlive` returns `true`. A genuinely live decal handle. Nothing visible on the wall. Not faint — nothing. Tried real built-in decal types, tried skipping the custom texture entirely to isolate the variable. Still nothing. Confirmed via community research: a known, unresolved, undocumented-by-Rockstar issue. `AddDecal` was abandoned, never to return (it got one more chance later, just to be thorough — see Round 5).

**Round 2 — A real prop, `AddReplaceTexture`.** Found `prop_ld_filmset` (a flat film-set billboard) via manual OpenIV asset hunting — including one genuinely funny detour where the prop's actual embedded texture name turned out to be `prop_fruit_box_01`, a leftover from Rockstar reusing an old texture slot. `AddReplaceTexture` ran without error. Prop spawned in the right place. Still showed the original art, never the custom texture.

**Round 3 — The actual root cause.** A GitHub issue search turned up `citizenfx/fivem#1684`: **`AddReplaceTexture` does not support "delayed" DUI-handle-sourced textures on any FXServer branch except Release.** This server intentionally runs `latest`. Every single failure up to this point — both the silent `AddDecal` and the silent `AddReplaceTexture` — traced back to the same underlying engine bug, because both pipelines sourced their runtime texture from a `CreateDui` call. The fix, straight from the FXServer dev himself: don't source the texture from a DUI at all.

**Round 4 — Rebuilding the pipeline around a PNG, not a browser.** The paint canvas now exports itself directly as a base64 PNG (`canvas.toDataURL`) the moment the player confirms. `CreateRuntimeTextureFromImage` takes that string straight in — confirmed by FiveM's own docs to accept a base64 data URL — no DUI anywhere in the chain. Simpler architecture and it sidesteps the bug entirely.

**Round 5 — `AddDecal`, again, now that the DUI excuse was gone.** Worth one more shot with a non-DUI texture source. Same result: `IsDecalAlive == true`, nothing visible. Confirmed, twice now, with two different texture sources: `AddDecal` is just not reliable on this engine/branch, full stop.

**Round 6 — Finding a prop that actually supports live replacement.** Not every screen-type prop does — confirmed via a community forum post describing the exact same symptom on a different prop (`prop_big_cin_screen` silently failed; `v_ilev_cin_screen`, using the same texture name, worked fine right next to it). Found `v_ilev_cin_screen` / `script_rt_cinscreen` — the `script_rt_` prefix turned out to be the real signal: Rockstar's own naming convention for script-driven runtime-replaceable texture slots. It worked. It also looked like a cinema screen, because it was one — curved, oversized, comically wrong for a wall tag.

**Round 7 — The database ate its own children.** Mid-investigation into prop sizing, a much bigger problem surfaced: every fresh server boot logged zero graffiti pieces loaded, even moments after successfully creating one in the same session. Root cause: `rde_crew_sprays` predates this rewrite, and its `created_at DEFAULT CURRENT_TIMESTAMP` had apparently never been reliably applied on the pre-existing table. An invalid timestamp made the routine 14-day housekeeping cleanup query nuke the entire table, every single restart, immediately. Three-layer fix: explicit `NOW()` on every insert, a migration repairing the column default, and a sanity-bounded cleanup query that can never again mass-delete on a bad timestamp.

**Round 8 — A race condition masquerading as a sync bug.** New pieces appeared to require a full resource restart to become visible to other players. Actual cause: the in-flight guard protecting the fetch-and-render sequence was released before rendering had actually finished — letting the next proximity tick start a second full render cycle on top of an unfinished one, repeatedly, stacking duplicate objects. Fixed by holding the guard for the full operation, not just the network round-trip.

**Round 9 — Found `prop_tv_flat_01_screen`,** the actual screen-only sub-component of a flat-screen TV model. Genuinely flat, no bezel. Scaling it down via `SetEntityMatrix` introduced visible parallelogram distortion. Abandoned in favor of not scaling at all — player-chosen scale is now applied within confirmed safe bounds.

**Round 10 — Named Render Targets.** Reading `rde_oxmedia`'s actual source revealed it doesn't use `AddReplaceTexture` for its TV — it uses `RegisterNamedRendertarget` + `LinkNamedRendertarget` + `DrawSprite`, redrawn every frame. Promising — until testing confirmed the render target is shared per model hash, not per entity instance: every simultaneously-loaded piece showed whichever was drawn most recently, globally, every frame. Worse for multi-piece support than `AddReplaceTexture`. Abandoned.

**Round 11 — The actual goal, restated.** The whole point was never "a working prop." It was a player saying, repeatedly and with increasing intensity: *"I just want the ink visible, nothing else."* Every vanilla screen material tested is deliberately opaque, because real screens always show something. That's not a bug to route around; it's how those materials were built.

**Round 12 — `bzzz_decals`.** A free third-party addon prop pack, built specifically for retexturing, found via a forum thread describing flat decal props where "on one side there is texture, on the other side you can't see anything." Verified directly in OpenIV with "Alpha background" checked before integrating a single file: a genuine checkerboard transparency pattern around the logo content. The first prop in this entire saga where the material was confirmed alpha-capable before testing, instead of discovered opaque after the fact. Integrated as a streamed addon, origTxd/origTxn reasoned out from file-size correlation. It worked. Real ink, on a real wall, with a real transparent background. No box. No bezel. No DUI. No decal.

**The cleanup.** `AddReplaceTexture`'s override turned out to be global per `(origTxd, origTxn)` — confirmed when re-entering render range started showing whichever other piece had patched the shared target most recently. Solved with a 10-slot pool, deterministically assigned by piece ID. The carrier prop needed its own rotation correction (90°, then corrected to 180° once tested in-game) and its own vertical pivot correction. The paint canvas had an O(n)-per-mouse-move redraw bug, fixed by baking finished strokes onto an offscreen layer once instead of replaying full history on every frame. The aiming flow was upgraded from "aim blind, press E, hope" to a live reticle — same tennis-ball entity-selection pattern already proven in `rde_doors`. And finally, v2.0.0 adds a full second NUI screen so the player can rotate and scale the piece directly on the wall surface before committing — because "somewhere in the right general area" was never good enough.

**Final tally:** three full rendering techniques attempted and two abandoned, one official CitizenFX engine bug identified and worked around, one self-inflicted database table found wiping itself on every boot, two race conditions, one third-party addon pack integrated from scratch, and a great many config values that say "CONFIRMED IN TESTING" because guessing twice was already too many times.

It renders. It's flat. It's transparent. It's real graffiti on a real wall.

---

## 📝 Changelog

### v2.0.1 — Cap System + Sync Fixes

**Graffiti: 7 Authentic Cap Types** *(feature proposed by 2kee)*
Replaced the 3 basic cap sizes (thin/medium/thick) with 7 authentic graffiti nozzle types: Skinny (2px crisp), Soft Cap (5px feathered), NY Cap (8px crisp), Standard (12px), Fat Cap (20px feathered), Ultra Fat (32px), Ultrawide (50px). Fat caps use canvas `shadowBlur` for authentic spray-paint edge feathering. Each cap button shows a live mini-stroke preview with real line width and blur.

**Graffiti: Prop Wall Alignment**
Replaced the heading-only `SetEntityRotation` approach with a full orthonormal `SetEntityMatrix` using `cross(n, worldUp)` for the right vector and `cross(right, n)` for the wall-up vector. Props now sit flush on any wall surface regardless of orientation. `isNormalSame` threshold raised from 0.01 to 0.05 to accept slightly uneven surfaces (ridged concrete, brick patterns).

**Sync: Zone + Spray Dual-Sync**
`ZoneEditor.publish()`, `publishZoneOwner()`, and `_publishSprayIndex()` now fire `TriggerClientEvent(-1, ...)` alongside `GlobalState` updates. Statebag propagation alone is not reliable on resource restart or in network edge cases. Both channels together cover all scenarios: net event for connected players, statebag for new joiners.

**Sync: Zone Color Race Condition Fix**
Zone ownership events now pass `ownerCrewId` inline so the client handler builds crew color from the event payload, never from `GlobalState` (which arrives ~50ms later and would still be stale). The `AddStateBagChangeHandler` for ownership is the single place that triggers the DUI rebuild — no concurrent threads destroying each other's textures.

**StateBag Handler: `value` Parameter Fix** *(the actual final fix)*
`AddStateBagChangeHandler(nil, 'global', handler)` fires with the new ownership data as the `value` parameter. However, reading `GlobalState[key]` inside the handler returns the OLD/nil value — the client-side state update is asynchronous from the handler invocation. This was confirmed via debug relay: `BUILD: zone txn=t2 claimed=false hex=nil rgb=255,255,255` even though the server had just set the ownership to crew #1.

Fix: use `value.ownerCrewId` directly from the handler parameter to build `newInfo`, then pass it as `infoOverride` to `renderZone` so `buildZoneTexture` uses the correct crew color without reading GlobalState for ownership. The crew snapshot (`GlobalState['rde_crew_<id>']`) is safe to read — it was set at startup, long before any spray event. Result: zone turns crew color on the minimap in real time, for every player, the moment the spray is confirmed.

**Overlay: Unique TxN Slot per Build**
`CreateRuntimeTextureFromDuiHandle(rtxd, txn, handle)` silently fails if `txn` already exists in that rtxd — no error, no update, old texture stays. Fix: increment a per-zone build counter so every rebuild uses a fresh slot name (`t1`, `t2`, ...) within a stable TxD. The scaleform reads the new slot and displays the correct crew color. `ZoneBuildPending` guard prevents concurrent builds from racing each other.

**zone.js: `JSON.parse` fix for `SendDuiMessage`**
`SendNUIMessage` delivers `e.data` pre-parsed as an object. `SendDuiMessage` delivers `e.data` as a **raw JSON string** — undocumented FiveM difference. Without `JSON.parse`, `d.action` is always `undefined` and the message is silently ignored.

**Overlay: Stale Lua rtxd Cache Fix**
`clearZoneTexture` calls `Overlay.clearCachedTxd(entry.txd)` to evict the Lua-level rtxd reference before destroying the DUI. Required for the fallback `renderZone()` path on first-time zone render.

**UI: Complete ox_lib Menu Overhaul**
Full rewrite of `client/ui.lua` — consistent `ICN.xxx` icons with `iconColor` on every item, contextual descriptions everywhere, `isLeader()` helper for permission gating. Settings menu now exposes Rename Crew, Set Color (hex), and Dissolve Crew for leaders; non-leaders see an informative disabled state. Member list shows online/offline status and leader badge. Member actions for leaders include Transfer Leadership, Change Grade, and Kick. Three new server callbacks added: `rde_crew:crews:rename`, `rde_crew:crews:setColor`, `rde_crew:members:transferLeadership` — all leader-only with server-side `isLeader()` validation.

**Config.Icons: Cleanup**
Removed `garage` and `inventory` (no longer used). Added `members`, `list`, `transfer`, `rename`, `flag`, `announce` for consistent icon coverage across all menus.

**Spray Can Prop + Animation + PTFX**
`prop_paint_spray01a` (cap on) attaches to the right hand during the aiming phase via bone `IK_R_Hand` (28422). During the mini-game, the prop is cleanly detached and a champagne-spray animation (`anim@mp_player_intupperspray_champagne/idle_a`) extends the arm toward the wall while a directional PTFX cloud (`core/ent_amb_rapid_dir_spray`) plays in front of the hand. Prop position tunable live via `Config.Graffiti.canProp.pos/rot` — change in `config.lua` and `restart rde_crew` without rebuilding.

**Spray Territory fix: `client/modules/territory.lua`**
Fixed syntax error at line 240 — two `Utils.log` debug calls had their format string literals accidentally stripped, leaving dangling `:format(args)` calls that crashed the Lua parser on resource start.

**Removed: Crew Stash & Garage**
`InventoryModule` and `GarageModule` removed from bootstrap and callbacks. Storage → `rde_parking`. Prop interaction → `rde_props_interact`. Dead code fully purged: `server/modules/inventory.lua`, `server/modules/garage.lua`, `client/modules/inventory.lua`, `client/modules/garage.lua` deleted. Stream assets `bzzz_normal_sign_*2.ydr`, `*3.ydr` (20 files, never spawned) and `bzzz_normal_sign_texture2.ytd`, `texture3.ytd` deleted. `server/database.lua` schema cleaned: `rde_crew_garage` table and four dead columns (`garage_coords`, `inventory_coords`, `garage_password`, `inventory_password`) removed from `rde_crews`. `server/classes/crew.lua` cleaned: dead `@field` annotations, dead `fromRow()` reads, and `setGarageCoords()`/`setInventoryCoords()` methods removed.

> **Honest debug trail — zone color, all iterations:**
> 1. Dual-sync: add `TriggerClientEvent` alongside GlobalState → HUD instant, minimap stale
> 2. Race condition: two concurrent `renderZone` threads destroying each other's DUIs → single-thread guard (`ZoneBuildPending`)
> 3. Lua rtxd cache: `state.runtimeTxd[txd]` never cleared → add `clearCachedTxd`
> 4. Engine TxD name cache: `CreateRuntimeTxd(same_name)` returns old TxD → unique TxD names
> 5. `CreateRuntimeTextureFromDuiHandle` on existing txn slot = silent no-op → unique TxN per build
> 6. `JSON.parse`: `SendDuiMessage` delivers raw string, not object → add parse
> 7. **StateBag handler reads stale GlobalState**: `ownerInfo()` inside handler sees nil → use `value` param directly ✅ **FIXED**
>
> Layer 7 confirmed via debug relay: `BUILD: zone claimed=false hex=nil rgb=255,255,255`. Each layer was correct in isolation.

---

### v2.0.0 — Mouse Placement + Canvas Performance

**NUI: Two-Screen Flow**
After drawing, a second placement screen shows the finished piece on a simulated wall. The player rotates and scales it before confirming. Rotation and scale are sent back to Lua. A full orthonormal basis (`forward = wall_normal`, `right = cross(n, worldUp)`, `up = cross(right, n)`) is applied via `SetEntityMatrix` — the prop sits flush on any wall surface regardless of orientation. *(Wall alignment further refined in v2.0.1.)*

**NUI: O(1) Canvas Drawing**
Previous implementation called `redraw()` (clearRect + full stroke history replay) on every `pointermove` event — O(n) per mouse move where n = total points in the current stroke. Replaced with direct incremental drawing: only the new segment since the last move event is drawn, the baked offscreen layer is not touched during an active stroke. Long sessions (80+ pieces, hundreds of strokes per piece) now stay fully responsive.

**Lua: Reticle / Placement Fix**
`hitCoords` and `hitNormal` are now captured into local upvalues the moment `[E]` is pressed, before `openPaintCanvas()` blocks for an arbitrary amount of time. The NUI and mini-game can no longer race with or overwrite the captured wall position.

**Lua: Rotation stored in DB**
`rde_crew_sprays` gains a `rotation FLOAT DEFAULT 0.0` column (auto-migrated via `ADD COLUMN IF NOT EXISTS`). Stored rotation is applied on prop spawn, so pieces persist their player-chosen angle across server restarts.

**Scope: Garage + Stash removed**
Vehicle storage is `rde_parking`'s job. Shared prop interaction is `rde_props_interact`'s job. Both do it better. The `USE_GARAGE` and `USE_STASH` permission keys, the four associated module files, and the `ox_inventory` dependency are removed.

---

### v1.12.0 — Live Aim Reticle
Real-time visual placement feedback, same pattern as `rde_doors`'s entity-selection loop.

### v1.11.x — Tuning + Canvas Offscreen Bake (partial)
Confirmed rotation/position values for the final carrier prop. 10-slot texture pool to prevent simultaneous pieces sharing one `AddReplaceTexture` target. Verbose diagnostic logging behind `Config.Graffiti.debug`.

### v1.10.x — bzzz_decals Integration
Pivoted to a free third-party addon prop pack with a confirmed real alpha-blended material after every vanilla screen material tested rendered fully opaque.

### v1.9.0 — Named Render Target Experiment
Tested the mechanism `rde_oxmedia` uses for live video; shared per model hash, not per instance — abandoned.

### v1.7.x–v1.8.x — Root Cause & Recovery
Identified `citizenfx/fivem#1684` as the actual root cause behind both the `AddDecal` and `AddReplaceTexture` failures. Rebuilt texture pipeline around direct base64 PNG with no DUI. Fixed `created_at` migration wiping the entire sprays table on every restart. Fixed render race condition causing duplicate spawned objects.

### v1.5.0–v1.6.x — Freehand Graffiti Rewrite
Replaced fixed crew-tag text stamp with real freehand NUI painting.

### Earlier
In-game polygon zone editor, statebag-driven crew/zone sync, war-gated territory overspray, full RDE OX Standards v2 pass.

---

## 🙏 Contributors & Credits

| | |
|---|---|
| 🐉 **SerpentsByte** | Architect, developer, and the poor soul who spent one very long night with `AddDecal` |
| 🎨 **2kee** | Proposed the authentic 7-cap graffiti nozzle system (Skinny → NY Cap → Fat → Ultrawide) with real blur feathering for fat caps — because "skinny, softcap, ny cap, standard, fat, ultrafat, ultrawide" is an accurate cap list and he knows it |

> 2kee on Nostr: [`Jkce`](https://primal.net/p/npub18swue4et8kumvxc78yxqwa3rqfucfymfvj8vr82q1l8smy8ueshly) · `2kee@nostrified.org`

---

## 📜 License

```
###################################################################################
# .:: RED DRAGON ELITE (RDE) - BLACK FLAG SOURCE LICENSE v6.66 ::.
# PROJECT: RDE_CREW v2.0.1 (OX-NATIVE CREW, GANG & TERRITORY SYSTEM)
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
| 🚗 RDE Parking | [rde_parking](https://github.com/RedDragonElite/rde_parking) |
| 🎯 RDE Props Interact | [rde_props_interact](https://github.com/RedDragonElite/rde_props_interact) |
| 🚪 RDE Doors | [rde_doors](https://github.com/RedDragonElite/rde_doors) |
| 📺 RDE OxMedia | [rde_oxmedia](https://github.com/RedDragonElite/rde_oxmedia) |
| 🚨 RDE AIPD | [rde_aipd](https://github.com/RedDragonElite/rde_aipd) |

When asking for help, always include:
- Full error from server console or txAdmin
- Your `server.cfg` resource start order
- ox_core / ox_lib versions in use

---

> *"It renders. It's flat. It's transparent. It's real graffiti on a real wall.*
> *Ask us what that cost."*
>
> **REJECT MODERN MEDIOCRITY. EMBRACE RDE SUPERIORITY.**

[![Website](https://img.shields.io/badge/Website-Visit-red?style=for-the-badge&logo=google-chrome)](https://rd-elite.com)
[![Nostr](https://img.shields.io/badge/Nostr-Follow-purple?style=for-the-badge&logo=rss)](https://primal.net/p/npub1wr4e24zn6zzjqx8kvnelfvktf0pu6l2gx4gvw06zead2eqyn23sq9tsd94)

🐉 *Made with 🔥 (and a genuinely heroic amount of stubbornness) by Red Dragon Elite*

[⬆ Back to Top](#-rde_crew)
