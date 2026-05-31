# Sentinel-NPCs — Developer API

Packet-based NPC system for Spigot / Paper / Folia. **This page documents the developer API.**
For installation, commands, configuration and NPC types, see the **[Wiki](../../wiki)**.

## Add Sentinel-NPCs as a dependency

The plugin jar doubles as the API jar — the `api` package and the model types (`NPC`, the six NPC
classes, `NPCAction`, …) are kept **un-obfuscated** so you can compile against them. Add it as a
**compile-only** dependency and declare the plugin as a `softdepend` (or `depend`) so it loads first.

**plugin.yml**
```yaml
softdepend: [ Sentinel-NPCs ]   # use depend: if your plugin requires it
```

**Gradle**
```groovy
dependencies {
    compileOnly files("libs/Sentinel-NPCs.jar")
}
```

**Maven**
```xml
<dependency>
    <groupId>github.dimazbtw</groupId>
    <artifactId>sentinel-npcs</artifactId>
    <version>1.0.0</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/libs/Sentinel-NPCs.jar</systemPath>
</dependency>
```

## Access the API

```java
SentinelAPI api = SentinelNPCs.get().getAPI();
```

## Create NPCs

```java
// Player
PlayerNPC npc = api.createPlayer("greeter", location, "Steve");

// Other types
api.createEntity("guard", location, org.bukkit.entity.EntityType.IRON_GOLEM);
api.createArmorStand("statue", location);
api.createFloatingHead("head", location);
api.createItemDisplay("loot", location);
api.createModelEngine("dragon", location, "my_model");

// Generic
NPC any = api.create(NPCType.PLAYER, "id", location);
```

Each `create*` registers the NPC and returns it. After changing an NPC, call `api.save(npc)` to
persist it and refresh it for viewers.

## Player skins

```java
npc.setDisplayName("&aGuard");
npc.setSkinByName("Notch");            // async — skin applies a moment later
npc.setSkinByURL("https://textures.minecraft.net/texture/...");
npc.setSkinByUUID(UUID.fromString("..."));  // async
npc.setMirrorSkin(true);               // each viewer sees their own skin
api.save(npc);
```

## Manage NPCs

```java
NPC npc = api.get("greeter");          // by id (null if none)
Collection<NPC> all = api.all();
NPCRegistry registry = api.registry();

api.save(npc);                          // persist + reload for viewers
api.reload(npc);                        // re-spawn for viewers (no disk write)
NPC copy = api.copy(npc, "greeter2");   // deep copy under a new id
NPC vip  = api.rename(npc, "greeter_vip");
api.remove(npc);                        // unregister + delete
```

## Add actions

```java
import github.dimazbtw.npcs.npc.action.NPCAction;

npc.getRightClickActions().add(new NPCAction(NPCAction.Type.MESSAGE, "&aWelcome!"));
npc.getRightClickActions().add(new NPCAction(NPCAction.Type.SOUND, "ENTITY_VILLAGER_YES 1 1"));
npc.getLeftClickActions().add(new NPCAction(NPCAction.Type.COMMAND, "spawn"));
api.save(npc);
```

Action types: `MESSAGE`, `COMMAND`, `CONSOLE`, `SOUND`, `TITLE`, `ACTIONBAR`, `DELAY`, `GUI`,
`TRIGGER`, plus the gate types `PERMISSION`, `CHANCE`, `COOLDOWN`, `MONEY`, `PLACEHOLDER`
(see the wiki's **Actions** page).

## Events

All under `github.dimazbtw.npcs.api.events` — standard Bukkit events. Every one exposes
`getNPC()` and `getPlayer()`.

| Event | Cancellable | Fired when |
|---|---|---|
| `NPCSpawnEvent` | ✅ | an NPC becomes visible to a player |
| `NPCDespawnEvent` | — | an NPC stops being shown to a player |
| `NPCClickEvent` / `NPCLeftClickEvent` / `NPCRightClickEvent` | ✅ | a player clicks an NPC |
| `NPCActionExecuteEvent` | ✅ | just before an NPC action runs |

```java
@EventHandler
public void onRightClick(NPCRightClickEvent e) {
    if (e.getNPC().getId().equals("greeter")) e.getPlayer().sendMessage("Hi!");
}

@EventHandler
public void onAction(NPCActionExecuteEvent e) {
    // Cancel ONE action; the rest of the chain still runs.
    if (e.getAction().type == NPCAction.Type.CONSOLE) e.setCancelled(true);
}
```

## Custom GUIs & triggers

```java
// Opened by an action payload "GUI:shop"
plugin.getGuiManager().registerCustom("shop", player -> myShopApi.open(player));

// Fired by an action payload "TRIGGER:openShop <arg>"
plugin.getHookRegistry().registerTrigger("openShop", (player, arg) -> myShopApi.open(player, arg));
```

## Custom NPC types

```java
api.registerType("hologram", (id, loc) -> new MyCustomNPC(id, loc));
NPC custom = api.createCustom("hologram", "myid", location);
```

`MyCustomNPC` extends `github.dimazbtw.npcs.npc.NPC` (or one of the built-in types).

---

*Requires the Sentinel-NPCs plugin + PacketEvents on the server. Internal classes (renderers,
pathfinder, GUIs, packet plumbing) are obfuscated — build only against the `api` package and the
model types shown above.*
