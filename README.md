<div align="center">

# Sentinel-NPCs

**High-performance, fully packet-based NPC system for Spigot / Paper 1.20.1 – 1.21.x and Folia.**

6 NPC types · packet holograms · A\* roaming · motion capture · conditional actions · developer API

</div>

Sentinel-NPCs creates NPCs entirely out of packets — with one exception (ModelEngine), no real
Bukkit entities are ever spawned. Each NPC exists only as packets sent to players within range, so
it stays light even with hundreds of NPCs.

> **Folia:** the plugin is `folia-supported` and routes all scheduling through a region-aware
> facade. ModelEngine NPCs auto-disable on Folia (ModelEngine itself isn't Folia-compatible).

---

## Table of contents

- [Requirements](#requirements)
- [Installation](#installation)
- [License](#license)
- [Configuration](#configuration)
- [Commands](#commands)
- [NPC types](#npc-types)
- [Actions & triggers](#actions--triggers)
- [Holograms](#holograms)
- [Movement & paths](#movement--paths)
- [Particles](#particles)
- [Developer API](#developer-api)
- [FAQ & troubleshooting](#faq--troubleshooting)

---

## Requirements

| | |
|---|---|
| **Server** | Spigot, Paper, or Folia **1.20.1 – 1.21.x** |
| **Java** | 17+ |
| **Required dependency** | [PacketEvents](https://modrinth.com/plugin/packetevents) (free) |
| **Optional hooks** | PlaceholderAPI · Vault · DecentHolograms · ItemsAdder · ModelEngine (R4) |

Players need nothing — no client mods. Everything is server-side packets.

---

## Installation

**1. Install PacketEvents (required).** Sentinel-NPCs is built on PacketEvents and will not start
without it.

1. Download [PacketEvents](https://modrinth.com/plugin/packetevents) and drop it into `plugins/`.

> PacketEvents is licensed separately (GPL-3.0) and is installed as its own plugin, exactly like
> ProtocolLib. If it's missing, Sentinel-NPCs disables itself on startup with a clear message.

**2. Install Sentinel-NPCs.**

1. Drop `Sentinel-NPCs.jar` into `plugins/`.
2. Start the server once — it generates `plugins/Sentinel-NPCs/config.yml`.
3. **Paste your license key** into `config.yml` (see [License](#license)).
4. Restart, or `/npc reload`.

Run `/npc` in-game to open the editor, or `/npc create player greeter` to make your first NPC.

### Optional integrations (auto-detected)

| Plugin | Adds |
|---|---|
| **PlaceholderAPI** | `%placeholders%` in holograms and action text (per-viewer) |
| **Vault** | the `MONEY:` action gate (charge players) |
| **DecentHolograms** | alternative hologram renderer (`holograms.provider: decentholograms`) |
| **ItemsAdder** | `:fontimage:` support in holograms and action text |
| **ModelEngine (R4)** | the `MODEL_ENGINE` NPC type (custom 3D models) |

---

## License

Sentinel-NPCs uses a single license key. Paste the key you received into `config.yml`:

```yaml
license: 'your-key-here'
```

Restart (or `/npc reload`). On a valid key the console shows:

```
[Sentinel-NPCs] License key accepted — thank you for your purchase!
```

If the key is missing or wrong, the plugin disables itself (the server keeps running) and tells you
to set a valid key. The key is validated **locally** — no internet connection required. One key is
issued per purchase; keep it private, as leaked copies are traceable.

---

## Configuration

`plugins/Sentinel-NPCs/config.yml`:

```yaml
license: ''

proximity:
  # Maximum distance (blocks) at which an NPC will spawn for a player.
  range: 48.0
  # How often (ticks) we re-check proximity for every NPC. Applied at startup.
  checkIntervalTicks: 20

holograms:
  # packet          - built-in packet holograms (no plugin, per-viewer PlaceholderAPI, Folia-ready)
  # decentholograms - use the DecentHolograms plugin instead (must be installed)
  provider: packet

# Anonymous usage statistics via bStats. Aggregate-only, no personal data.
metrics:
  enabled: true
```

| Key | Default | Description |
|---|---|---|
| `license` | `''` | Your license key. |
| `proximity.range` | `48.0` | How close a player must be for an NPC to spawn for them. |
| `proximity.checkIntervalTicks` | `20` | Ticks between proximity re-checks (read at startup). |
| `holograms.provider` | `packet` | `packet` (built-in) or `decentholograms`. |
| `metrics.enabled` | `true` | Anonymous bStats statistics. |

NPCs are **not** stored in `config.yml` — each is its own file under
`plugins/Sentinel-NPCs/npcs/<id>.yml`, written automatically. `/npc reload` re-reads the config and
all NPCs (except `checkIntervalTicks`, which needs a restart).

---

## Commands

Base command: `/npc` (aliases `/snpc`, `/sentinelnpc`). Everything works from the **GUI** (`/npc`)
or **commands**. Commands act on your **selected** NPC, so you don't repeat the id.

**Select an NPC:** `/npc select <id>`, **shift-click** it (admins: shift-left selects, shift-right
opens the editor), or `/npc tool` for the selector stick.

| Command | Description |
|---|---|
| `/npc` | Open the main menu |
| `/npc help [page]` | Command overview |
| `/npc tool` | Get the selection stick |
| `/npc select <id>` · `deselect` | Set / clear your selection |
| `/npc create <type> <id>` | Create and select a new NPC |
| `/npc list [page]` · `near [radius]` | List all / nearby NPCs |
| `/npc edit` · `info` · `remove` | Edit GUI / print info / delete (selected) |
| `/npc clone <id>` · `rename <id>` | Duplicate / rename (selected) |
| `/npc move` · `tp` · `tpcoords <x y z [yaw pitch]>` | Move NPC to you / you to NPC / NPC to coords |
| `/npc set <prop> <value…>` | Common + type-specific properties (below) |
| `/npc action <left\|right\|approach\|leave> <add\|remove\|list\|clear\|up\|down>` | Click & proximity actions |
| `/npc particle <add\|remove\|list\|set>` | Particle effects |
| `/npc hologram <add\|set\|remove\|list\|clear\|offset\|spacing>` | Hologram lines |
| `/npc path <setup\|mode\|speed\|wait\|clear>` | Walking path |
| `/npc roam <setup\|mode\|speed\|trigger\|idle\|clear>` | Area AI roaming |
| `/npc record <start\|stop\|clear>` | Motion capture |
| `/npc pose <part> <x y z>` · `equip <slot> [hand\|clear]` | Armor Stand pose / equipment |
| `/npc reload` | Reload config + NPCs |

`<type>`: `player`, `entity`, `armor_stand`, `floating_head`, `item_display`, `model_engine`. Every
level has tab-completion.

### `/npc set` properties

`set` changes a property on your selected NPC. The common ones are `cooldown`, `lookat`,
`permission`, and `interactrange`; each type also has its own fields. **See
[NPC types](#npc-types) for every type's settings — their accepted values and what each one does.**

### Permissions

| Node | Grants |
|---|---|
| `sentinelnpcs.command.create` / `.remove` / `.edit` / `.list` / `.reload` | The matching command |
| `sentinelnpcs.*` | All of the above (default: `op`) |

All nodes default to `op`. The selection stick and every mutation command require `.edit`.

---

Create with `/npc create <type> <id>`, then edit in the GUI (`/npc edit`) or with `/npc set …`.
Each type has its **own settings** (below) on top of the shared ones. All values are set with
`/npc set <field> <value>` on your selected NPC and saved instantly.

All settings are applied with `/npc set <field> <value>` on your selected NPC. The blocks below
read like a config file — the field on the left, an example value, and `#` what it does.

### Settings every type has

```yaml
# Works on EVERY type   —   /npc set <field> <value>
cooldown: 500          # ms between clicks from the same player (0 = no cooldown)
lookat: on             # head tracks the nearest player (up and down too)
permission: rank.vip   # only players with this node can see/use the NPC ("clear" removes it)
interactrange: 5       # blocks at which approach/leave actions fire (0 = off)
```

Every type can also have **holograms**, **particles**, and **click/proximity actions**. Player,
Entity, and ModelEngine additionally support **movement** (paths / roaming / motion capture).

### Player — a real player NPC

```yaml
# Player   —   /npc set <field> <value>
name: "&aGuard"        # name shown above the NPC (supports & colors)
skin: Notch            # use this player's current skin
skinurl: https://...   #   ...or load a skin from a texture URL
skinuuid: <uuid>       #   ...or use a skin by account UUID
mirror: off            # on = every viewer sees their OWN skin (overrides skin above)
```

### Entity — any vanilla mob

```yaml
# Entity   —   /npc set <field> <value>
entitytype: zombie     # any vanilla mob: villager, warden, cow, ...
baby: off              # on = baby variant (for mobs that have one)
```

### Armor Stand — statues, poses, decorations

```yaml
# Armor Stand   —   /npc set <field> <value>
invisible: off         # on = hide the stand body (show only equipment/pose)
arms: on               # show arms (needed to hold items in the hands)
small: off             # small armor stand
baseplate: on          # show the base plate underneath
marker: off            # on = no hitbox + zero size (decoration only)
emote: none            # looping animation: wave | nod | shake | dance | bow | none
```

> **Emotes** are simple procedural loops applied on top of your static pose (so a posed statue can
> still wave). The arm emotes — `wave` and `dance` — only show with `arms: on`.

Two extra commands (not `set` fields):

```
/npc equip <slot> [hand|clear]   # slots: head chest legs feet mainhand offhand
                                 #   no argument = item in your hand,  clear = empty the slot
/npc pose <part> <x> <y> <z>     # in degrees; parts: head body leftarm rightarm leftleg rightleg
```

### Floating Head — a floating/spinning head

```yaml
# Floating Head   —   /npc set <field> <value>
texture: Notch         # head skin: a player name or a texture URL
rotate: on             # spin the head continuously
rotatespeed: 5         # spin speed in degrees per tick
float: on              # bob up and down
floatamplitude: 0.2    # how far it bobs (blocks)
floatperiod: 40        # length of one bob cycle (ticks; higher = slower)
small: off             # smaller head
```

### Item Display — any item as a prop

```yaml
# Item Display (1.19.4+)   —   /npc set <field> <value>
item:                  # hold an item, then run "/npc set item" to show it
scale: 1               # size: one number, or "x,y,z" (e.g. 1,2,1)
billboard: center      # facing: fixed | vertical | horizontal | center   (center = faces you)
transform: none        # pose: none | thirdperson_left | thirdperson_right | firstperson_left
                       #       | firstperson_right | head | gui | ground | fixed
glow: FF0000           # glowing outline as a hex color   ("clear" to remove)
brightness: 15         # forced light level 0-15 (15 = fullbright; "clear" = world light)
shadowradius: 0        # ground shadow size in blocks (0 = no shadow)
shadowstrength: 1      # how dark the shadow is
viewrange: 1.0         # render-distance multiplier
```

### ModelEngine — a custom 3D model

Requires **ModelEngine R4** (the only type that uses a real, invisible base entity).
**Auto-disabled on Folia.**

```yaml
# ModelEngine   —   /npc set <field> <value>
model: my_model        # the ModelEngine model id
idleanim: idle         # animation played while idle    ("clear" to remove)
interactanim: wave     # animation played when clicked   ("clear" to remove)
```

---

## Actions & triggers

Actions run on **click** (left/right) and on **approach/leave**. Effects do things; **gates**
conditionally stop the rest of the chain.

```
/npc action <left|right|approach|leave> <add <type> <payload…>|remove <i>|list|clear|up <i>|down <i>>
```

For `approach`/`leave`, set `/npc set interactrange <blocks>`.

**Effects:**

| Type | Payload | Does |
|---|---|---|
| `MESSAGE` | text | Chat message (`&` colors) |
| `COMMAND` | command | Runs as the **player** |
| `CONSOLE` | command | Runs from **console** (use `%player%`) |
| `SOUND` | `SOUND_NAME [volume] [pitch]` | Plays a sound |
| `TITLE` | `title\|subtitle` | Title/subtitle |
| `ACTIONBAR` | text | Action-bar message |
| `DELAY` | `<ticks>` | Waits before the next actions |
| `GUI` | `<key>` | Opens a custom GUI (API-registered) |
| `TRIGGER` | `<name> [arg]` | Fires a hook (API-registered) |

**Gates** (stop the chain if the condition fails):

| Gate | Payload | Continues if… |
|---|---|---|
| `PERMISSION` | `<node>` | the player has the node |
| `CHANCE` | `<percent>` | a random roll passes |
| `COOLDOWN` | `<seconds> [key]` | the per-player cooldown elapsed |
| `MONEY` | `<amount>` | the player can pay (Vault) |
| `PLACEHOLDER` | `<%papi% OP value>` | the comparison is true (`>= <= == != > <` / `contains`) |

**Example** — a shop button with cooldown & cost:

```
/npc action right add COOLDOWN 3
/npc action right add MONEY 100
/npc action right add MESSAGE &aPurchased! &7-100 coins
/npc action right add SOUND ENTITY_PLAYER_LEVELUP 1 1
```

All action text supports `&` colors, PlaceholderAPI, and ItemsAdder font images.

---

## Holograms

Every NPC can carry a multi-line floating hologram, rendered by the **built-in packet renderer** —
no hologram plugin required.

```
/npc hologram add <text>        # append a line
/npc hologram set <i> <text>    # edit line i
/npc hologram remove <i>        # delete line i
/npc hologram clear             # remove all
/npc hologram offset <y>        # vertical offset
/npc hologram spacing <v>       # gap between lines
```

The packet renderer (`holograms.provider: packet`, default) needs no plugin, resolves
**PlaceholderAPI per viewer**, **follows walking NPCs**, and **works on Folia**. Set
`provider: decentholograms` to use DecentHolograms instead.

---

## Movement & paths

NPCs move in three ways, with precedence **motion capture → roaming → waypoint path** (Player,
Entity, ModelEngine). Vertical movement is player-like: NPCs jump up steps, walk down slopes, and
fall off real ledges.

**Waypoint paths:**

```
/npc path setup        # click blocks to add waypoints
/npc path mode <loop|ping_pong>
/npc path speed <blocks-per-tick>
/npc path wait <ticks>
/npc path clear
```

**Area roaming (A\* AI):**

```
/npc roam setup        # click two corners to define a cuboid
/npc roam mode <wander|follow|flee>
/npc roam speed <blocks-per-tick>
/npc roam trigger <blocks>   # follow/flee + greeting range
/npc roam idle <on|off>
/npc roam clear
```

Real A\* bounded to your region — avoids non-solid blocks, steps up/down one block, and stops to
face players within interact range.

**Motion capture** — record your own walk and replay it:

```
/npc record start      # walk how you want the NPC to move
/npc record stop       # saves and starts replaying
/npc record clear
```

**Emotes** (Armor Stand): `/npc set emote <wave|nod|shake|dance|bow|none>`.

---

## Particles

Each NPC can emit particle effects. Animations are **moving trails** that loop around the NPC.

```
/npc particle add <type>
/npc particle list
/npc particle remove <i>
/npc particle set <i> <field> <value>     # radius, height, count, density, animspeed, trail, spread…
```

Animations: `CIRCLE`, `SPIRAL_UP`, `SPIRAL_DOWN`, `HELIX`, `TORNADO`, `HALO`, `ATOM`, `VORTEX`,
`WINGS`, and `PULSE` (random ambient scatter). The **trail** field controls how much of the path is
drawn at once. Any server particle can be used (the GUI has a searchable picker).

---

## Developer API

The `github.dimazbtw.npcs.api` package and the model types (`NPC`, the six NPC classes, actions,
etc.) are kept **un-obfuscated** so you can compile against them. Add the plugin jar as a
**compile-only** dependency and `softdepend: [ Sentinel-NPCs ]` in your `plugin.yml`.

```java
SentinelAPI api = SentinelNPCs.get().getAPI();

PlayerNPC npc = api.createPlayer("greeter", loc, "Steve");
npc.setSkinByName("Notch");
npc.getRightClickActions().add(new NPCAction(NPCAction.Type.MESSAGE, "&aWelcome!"));
api.save(npc);

// Bukkit-typed entity creation (no PacketEvents types needed)
api.createEntity("guard", loc, org.bukkit.entity.EntityType.IRON_GOLEM);

// Query / clone / rename / remove
NPC copy = api.copy(api.get("greeter"), "greeter2");
api.rename(copy, "greeter_vip");
```

**Events** (under `api.events`):

| Event | Cancellable | Fired when |
|---|---|---|
| `NPCSpawnEvent` | ✅ | an NPC becomes visible to a player |
| `NPCDespawnEvent` | — | an NPC stops being shown to a player |
| `NPCClickEvent` / `NPCLeftClickEvent` / `NPCRightClickEvent` | ✅ | a player clicks an NPC |
| `NPCActionExecuteEvent` | ✅ | just before an NPC action runs |

```java
@EventHandler
public void onAction(NPCActionExecuteEvent e) {
    if (e.getAction().type == NPCAction.Type.CONSOLE) e.setCancelled(true); // cancel one action
}
```

**Custom GUIs, triggers & types:**

```java
plugin.getGuiManager().registerCustom("shop", player -> myShopApi.open(player));
plugin.getHookRegistry().registerTrigger("openShop", (player, arg) -> myShopApi.open(player, arg));
api.registerType("hologram", (id, l) -> new MyCustomNPC(id, l));
```

---

## FAQ & troubleshooting

**Plugin disables itself on startup** — install [PacketEvents](https://modrinth.com/plugin/packetevents)
(required), and paste a valid key into `license:` (see [License](#license)).

**NPCs don't appear** — be within `proximity.range` (default 48), check you don't have a view
permission set that you lack, and confirm with `/npc list`.

**Does it work on Folia?** — Yes. Only the **ModelEngine** type auto-disables (ModelEngine isn't
Folia-compatible).

**Do players need anything?** — No. Everything is server-side packets; only the server needs PacketEvents.

**Placeholders not per-player in holograms?** — Use `holograms.provider: packet` and install
PlaceholderAPI. The packet renderer resolves placeholders per viewer; DecentHolograms does not.

**How many NPCs can I run?** — Hundreds. Cost scales with how many are **visible at once** (in range
of players), not how many exist — lower `proximity.range` for very dense areas.

**Where is NPC data?** — One YAML file per NPC under `plugins/Sentinel-NPCs/npcs/<id>.yml`, written
automatically. Use the GUI/commands rather than editing by hand.

---

<div align="center">

**Support:** *(your Discord / ticket link)* · Built on [PacketEvents](https://github.com/retrooper/packetevents)

</div>
