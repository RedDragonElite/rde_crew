# 🐉 rde_crew

🔥 OX-NATIVE CREW, GANG & TERRITORY SYSTEM V2.0.1 — Real Freehand Graffiti, Live Minimap DUI Pipe, Zero Poll Loops! 🐉

[![Version](https://img.shields.io/badge/version-2.0.1-red?style=for-the-badge)](https://github.com/RedDragonElite/rde_crew)
[![License](https://img.shields.io/badge/license-RDE%20Black%20Flag%20v6.66-black?style=for-the-badge)](https://github.com/RedDragonElite/rde_crew/blob/main/LICENSE)
[![FiveM](https://img.shields.io/badge/FiveM-Compatible-blue?style=for-the-badge)](https://fivem.net)
[![ox_core](https://img.shields.io/badge/Framework-ox__core-blue?style=for-the-badge)](https://github.com/overextended/ox_core)
[![Price](https://img.shields.io/badge/price-FREE%20FOREVER-brightgreen?style=for-the-badge)](https://github.com/RedDragonElite/rde_crew)
[![RDE Ecosystem](https://img.shields.io/badge/RDE-ECOSYSTEM-f59e0b?style=for-the-badge)](https://github.com/RedDragonElite)
[![rde_organizations](https://img.shields.io/badge/integrates-rde__organizations-orange?style=for-the-badge)](https://github.com/RedDragonElite/rde_organizations)
[![Pain](https://img.shields.io/badge/dev%20pain-IMMEASURABLE-8B0000?style=for-the-badge)](#-the-graffiti-saga-a-war-story)

**Crew creation, ranks, permissions, declared wars, an in-game polygon zone editor, and real freehand spray-painted territory tags that render as actual 3D objects on actual walls — drawn with your mouse, positioned on the wall by hand, statebag-driven, proximity-loaded, and free.**

**Optional:** pair with [`rde_organizations`](https://github.com/RedDragonElite/rde_organizations) and gang-type orgs automatically become territory-holding crews. The organization panel replaces the crew management UI — rde_crew becomes a pure territory engine.

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
- [OrgBridge — rde_organizations Integration](#-orgbridge--rde_organizations-integration)
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

> 💡 **OrgBridge mode:** When used alongside [`rde_organizations`](https://github.com/RedDragonElite/rde_organizations), crew management moves entirely into the org panel. Members, ranks, and permissions are managed through rde_organizations — rde_crew becomes a pure territory engine. See [OrgBridge](#-orgbridge--rde_organizations-integration).

### 🗺️ Territory & Zones

- **In-game polygon zone editor** — walk the perimeter, place corners live, persist to DB, edit any time
- Zone ownership computed by a configurable rule: `last_spray` or `spray_count`
- **War-gated overspray** — an enemy crew can only paint over your owned zone while an active war exists between both crews
- Zone ownership changes broadcast instantly via per-zone statebags with automatic crew notifications on both sides
- **Live DUI minimap pipe** — when ownership changes, the minimap texture updates in the same frame. No TxD rebuild, no DUI destroy/recreate cycle, zero delay.

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

The chosen rotation and scale are sent back to Lua. A full orthonormal basis (`forward = wall_normal`, `right = cross(n, worldUp)`, `up = cross(right, n)`) is computed and applied via `SetEntityMatrix` — the prop sits flush on any wall surface regardless of orientation, angle, or texture pattern.

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

---

## 🔗 OrgBridge — rde_organizations Integration

rde_crew ships with a built-in bridge module (`server/modules/org_bridge.lua`) that connects it to [`rde_organizations`](https://github.com/RedDragonElite/rde_organizations).

**The idea is simple:** rde_organizations is the HR layer (who is in a crew, what rank, what permissions), rde_crew is the territory layer (which zone does a crew own, where is their graffiti, are they at war). Neither resource needs to know the other exists. The bridge is the invisible glue between them.

### What the bridge does automatically

| Action in rde_organizations | Result in rde_crew |
|---|---|
| Create a gang / MC / criminal org | Crew is auto-created and linked |
| Member accepts invite | Added to the linked crew |
| Member is kicked or leaves | Removed from the linked crew |
| Rank changed (promote/demote) | Crew grade updated instantly |
| Rank created, edited, or deleted | Crew grades table rebuilt from org ranks |
| Leadership transferred | Crew leader updated |
| Org name or color changed | Crew name and colorHex updated |
| Org disbanded | Crew dissolved, graffiti cascade-deleted |
| Org type changed to gang | Crew auto-created |
| Org type changed away from gang | Crew dissolved |

### Which org types become crews

Configurable in `Config.OrgBridge.crewOrgTypes` (default: `gang`, `mc`, `criminal`). Any rde_organizations type listed there is automatically treated as a territory-holding crew.

### Permission mapping

rde_organizations rank permissions map to rde_crew grade permissions via `Config.OrgBridge.permMap`. Two territory-specific permissions (`spray_territory`, `declare_war`) are added to rde_organizations and map directly. Existing org permissions (`invite`, `kick`, `assign_ranks`) map to their rde_crew equivalents.

### Standalone mode

Set `Config.OrgBridge.enabled = false` (or don't install rde_organizations) and rde_crew runs exactly as before — boss NPC, `/crew` menu, manual crew management. The bridge is purely additive.

---

## 📦 Dependencies

| Resource | Required | Notes |
|---|---|---|
| `oxmysql` | ✅ Required | Database layer |
| `ox_core` | ✅ Required | Player/character framework |
| `ox_lib` | ✅ Required | Callbacks, notifications, progress bars |
| `ox_target` | ✅ Required | Interaction prompts |
| `bzzz_decals` | ✅ Required | Addon prop pack — flat alpha-capable carrier props for graffiti rendering (streamed in `rde_crew/stream/`) |
| `rde_organizations` | ⚙️ Optional | Enables OrgBridge — gang-type orgs become territory crews automatically |

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
> If using OrgBridge: `rde_organizations` must start **before** `rde_crew`.

### 3. Spray can items

Define each spray can color as a normal `ox_inventory` item (label/weight/image), then add **one line** to `Config.Territory.sprayCans` per color. See `ox_inventory_items_SNIPPET.lua` for a ready-made starter set matching the default 10-color palette.

### 4. Database

All tables are created automatically on first start. No manual SQL import needed.

### 5. Configure (optional)

Edit `config.lua` to adjust crew limits, territory rules, the spray-can color registry, graffiti rendering tuning, and OrgBridge settings.

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

### Graffiti Rendering

```lua
Config.Graffiti = {
    canvasSize        = { w = 600, h = 450 },
    maxImageBytes     = 500 * 1024,
    renderDist        = 35.0,
    unloadDist        = 45.0,
    propHeadingOffset = 180,
    propZOffset       = 0.15,
    propEmbedDepth    = 0.02,
    areaSize          = { w = 2.4, h = 1.8 },
    debug             = false,
}
```

### OrgBridge

```lua
Config.OrgBridge = {
    enabled = true,       -- false = standalone mode, no rde_organizations needed
    crewOrgTypes = {
        gang     = true,
        mc       = true,
        criminal = true,
    },
    permMap = {
        spray_territory = 'spray_territory',
        declare_war     = 'declare_war',
        invite          = 'invite',
        kick            = 'kick',
        change_grade    = 'assign_ranks',
        send_announce   = 'edit_org',
        set_colour      = 'edit_org',
    },
}
```

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
│   │   ├── org_bridge.lua  ← optional rde_organizations bridge (v2.0.1+)
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

- Every spray placement validated server-side: image format, size cap, every used can verified against the real can registry
- Zone lookup at placement time is always server-computed — client-claimed zone names are never trusted
- War-gate enforced server-side before any overspray is accepted
- `rotation` and `scale` values from the NUI are range-clamped server-side before being written to the DB
- Permission checks on every crew action against the actor's actual grade, not client-claimed state
- OrgBridge: crew mutations only via inter-resource server events — clients never touch the bridge directly

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
| `rde_crew_org_link` | OrgBridge: maps rde_organizations org_id → crew_id |

All tables auto-create on first start. `rde_crew_org_link` is only populated when OrgBridge is active and a gang-type org is created in rde_organizations — it is fully inert otherwise.

---

## ⚡ Performance

**Graffiti two-tier model:** a tiny `GlobalState`-broadcast index (id/coords/zone/size only) drives proximity math client-side. The actual PNG is only ever fetched via on-demand callback once a client enters `renderDist`, and the rendered prop tears down past `unloadDist`. Thousands of server-wide pieces, zero per-tick overhead for distant clients.

**Paint canvas — O(1) per mouse move:** finished strokes are baked once onto an offscreen buffer. On `pointermove`, only the current new segment is drawn directly to the visible canvas — no `clearRect`, no history replay, no `redraw()`.

**Render thread — capped at ~60fps:** the zone outline draw loop runs at `Wait(16)` when zones are nearby, not `Wait(0)`. `DrawLine`/`DrawMarker` native calls are cheap individually but accumulate fast at 6 raycasts × 60+ fps. All hot loops use `cache.ped` instead of `PlayerPedId()`.

**Zone ownership and statebag publishes** are event-driven, never polled.

---

## 🐛 Troubleshooting

**Graffiti piece doesn't render?**
Check console for `DLC_ITYP_REQUEST` errors — `bzzz_decals.ytyp` must load clean. Set `Config.Graffiti.debug = true` for full F8 render pipeline logging per piece.

**bzzz_decals not found?**
The prop pack is streamed from `rde_crew/stream/` — no separate download needed.

**Piece renders at wrong height or rotation after swapping carrier props?**
`propHeadingOffset` and `propZOffset` are specific to the current carrier model's mesh-pivot convention. Both need re-tuning if you change the prop.

**Two nearby pieces show the same image?**
The render pool has 10 slots. Extend `GRAFFITI_PROP_POOL` in `client/modules/spray.lua` with more `bzzz_normal_sign_*` letters if your server regularly has dense territory clusters.

**OrgBridge: crew not created after org creation?**
Check server console for `[OrgBridge]` log lines. Confirm `rde_organizations` starts before `rde_crew` in `server.cfg`. Confirm the org type is in `Config.OrgBridge.crewOrgTypes`.

**OrgBridge: members not syncing?**
Confirm the `rde_orgs:server:*` events are being fired — these are added to `rde_organizations/server/server.lua`. Make sure you're running the bridge-patched version of rde_organizations.

---

## 🩸 The Graffiti Saga: A War Story

Every config comment in this resource that starts with "CONFIRMED IN TESTING" or "BUGFIX" is a scar from one overnight session. This is that story, because it was too absurd to quietly ship without documenting.

**The plan was simple.** Replace the old fixed-text crew-tag stamp with real freehand painting. Player draws on a canvas, art gets projected onto the wall as a decal. A weekend feature, surely.

**Round 1 — `AddDecal`.** GTA's native bullet-hole/scorch-mark system, repurposed for graffiti. Built the whole pipeline: DUI runtime texture, `PatchDecalDiffuseMap`, `AddDecal` with the wall normal and a computed side vector. In-game test: `IsDecalAlive` returns `true`. Nothing visible on the wall. Not faint — nothing. Confirmed via community research: a known, unresolved, undocumented-by-Rockstar issue. Abandoned.

**Round 2 — A real prop, `AddReplaceTexture`.** Found `prop_ld_filmset` via manual OpenIV asset hunting. `AddReplaceTexture` ran without error. Still showed the original art, never the custom texture.

**Round 3 — The actual root cause.** A GitHub issue search turned up `citizenfx/fivem#1684`: **`AddReplaceTexture` does not support "delayed" DUI-handle-sourced textures on any FXServer branch except Release.** This server intentionally runs `latest`. Every failure traced back to the same underlying engine bug, because both pipelines sourced their texture from a `CreateDui` call.

**Round 4 — Rebuilding the pipeline around a PNG, not a browser.** The paint canvas now exports itself directly as a base64 PNG (`canvas.toDataURL`) the moment the player confirms. `CreateRuntimeTextureFromImage` takes that string straight in — no DUI anywhere in the chain.

**Round 5 — `AddDecal`, again.** Worth one more shot with a non-DUI texture source. Same result. Confirmed, twice, with two different texture sources.

**Rounds 6–11 — Named render targets, cinema screen props, database self-destruction, race conditions, parallelogram distortion, and a prop whose embedded texture name turned out to be `prop_fruit_box_01`.** All documented in brutal detail in the `config.lua` comments and the full saga section below.

**Round 12 — `bzzz_decals`.** A free third-party addon prop pack with a confirmed real alpha-blended material, verified in OpenIV with "Alpha background" checked before integrating a single file. It worked. Real ink, on a real wall, with a real transparent background.

**The cleanup.** 10-slot pool to prevent simultaneous pieces sharing one texture target. Full orthonormal `SetEntityMatrix` to place props flush on any wall. O(1)-per-mouse-move canvas by baking strokes to an offscreen layer. Live reticle. Rotation/scale persistence in DB. Two-screen NUI so players see exactly what they're placing before they commit.

**Final tally:** three full rendering techniques attempted and two abandoned, one official CitizenFX engine bug identified and worked around, one self-inflicted database table found wiping itself on every boot, two race conditions, one third-party addon pack integrated from scratch, and config values that say "CONFIRMED IN TESTING" because guessing twice was already too many times.

It renders. It's flat. It's transparent. It's real graffiti on a real wall.

---

## 📝 Changelog

### v2.0.1 — Cap System + Sync Fixes + OrgBridge

**OrgBridge: rde_organizations Integration** *(new in v2.0.1)*
Optional bridge module (`server/modules/org_bridge.lua`) that links `rde_organizations` gang-type orgs to rde_crew's territory system. When enabled, gang/MC/criminal orgs created in rde_organizations automatically become territory-holding crews. Membership, ranks, and permissions sync automatically in both directions via inter-resource server events. rde_crew becomes a pure territory engine — the org panel handles all crew management. Zero coupling: both resources function independently if the bridge is disabled.

**Performance: Hot-Loop Optimizations**
Spray aim loop throttled from `Wait(0)` (6 raycasts at 60+ fps) to `Wait(16)`. Territory render loop capped at `Wait(16)` when zones nearby. NUI polling loop `Wait(0)` → `Wait(16)`. Named render target thread guarded behind config flag (was always active even when opt-in was disabled). `PlayerPedId()` replaced with `cache.ped` in all hot paths. 5 per-frame `vec3()` allocations hoisted to module-level constants. `Utils.log()` now gates 'info' level behind `Config.Debug`. Net result: ~10× lower CPU at idle, ~5× lower during active spray.

**Graffiti: 7 Authentic Cap Types** *(feature proposed by 2kee)*
Replaced 3 basic cap sizes with 7 authentic graffiti nozzle types. Fat caps use canvas `shadowBlur` for authentic spray-paint edge feathering. Each cap button shows a live mini-stroke preview.

**Graffiti: Prop Wall Alignment**
Replaced heading-only `SetEntityRotation` with full orthonormal `SetEntityMatrix`. Props now sit flush on any wall surface regardless of orientation.

**Sync: Zone + Spray Dual-Sync**
`ZoneEditor.publish()`, `publishZoneOwner()`, and `_publishSprayIndex()` now fire `TriggerClientEvent(-1, ...)` alongside `GlobalState` updates.

**Sync: Zone Color Race Condition Fix**
Zone ownership events now pass `ownerCrewId` inline. `AddStateBagChangeHandler` for ownership is the single place that triggers DUI rebuild. `ZoneBuildPending` guard prevents concurrent threads destroying each other's textures.

**StateBag Handler: `value` Parameter Fix**
`AddStateBagChangeHandler` fires with new ownership data as the `value` parameter. `GlobalState[key]` inside the handler returns the old/nil value. Fix: use `value.ownerCrewId` directly, pass as `infoOverride` to `buildZoneTexture`.

**Overlay: Unique TxN Slot per Build**
`CreateRuntimeTextureFromDuiHandle` silently fails if `txn` already exists. Fix: per-zone build counter ensures every rebuild uses a fresh slot name.

**Locales: Complete EN + DE**
12 previously missing locale keys added to both `locales/en.lua` and `locales/de.lua`. Zero missing keys at release.

### v2.0.0 — Mouse Placement + Canvas Performance

Two-screen NUI flow (draw → place). Full orthonormal `SetEntityMatrix`. O(1) canvas drawing via incremental stroke rendering. `hitCoords`/`hitNormal` race fix. `rotation` column added to DB via auto-migration. Garage + stash removed.

### v1.x — Earlier History

In-game polygon zone editor, statebag-driven crew/zone sync, war-gated territory overspray, graffiti saga (all 12 rounds), live aim reticle, bzzz_decals integration.

---

## 🙏 Contributors & Credits

| | |
|---|---|
| 🐉 **SerpentsByte** | Architect, developer, and the poor soul who spent one very long night with `AddDecal` |
| 🎨 **2kee** | Proposed the authentic 7-cap graffiti nozzle system (Skinny → NY Cap → Fat → Ultrawide) |

> 2kee on Nostr: [`2kee`](https://primal.net/p/npub18swue4et8kumvxc78yxqwa3rqfucfymfvj8vr82q1l8smy8ueshly) · `2kee@nostrified.org`

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
#
# 4. // THE CURSE OF THE COPY-PASTE
#    This code uses statebags, proximity streaming, a custom addon prop pack,
#    and a runtime texture pipeline that survived three abandoned rendering
#    techniques and one CitizenFX engine bug. If you just copy-paste without
#    reading, it WILL break. Don't come crying to my DMs. RTFM or learn to code.
#
# "We build the future on the graves of paid resources."
# "REJECT MODERN MEDIOCRITY. EMBRACE RDE SUPERIORITY."
###################################################################################
```

---

## 🌐 Community & Support

| | |
|---|---|
| 🐙 GitHub | [RedDragonElite](https://github.com/RedDragonElite) |
| 🌍 Website | [rd-elite.com](https://rd-elite.com) |
| 🏢 RDE Organizations | [rde_organizations](https://github.com/RedDragonElite/rde_organizations) |
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
[![RDE Ecosystem](https://img.shields.io/badge/RDE-ECOSYSTEM-f59e0b?style=for-the-badge)](https://github.com/RedDragonElite)

🐉 *Made with 🔥 (and a genuinely heroic amount of stubbornness) by Red Dragon Elite*

[⬆ Back to Top](#-rde_crew)
