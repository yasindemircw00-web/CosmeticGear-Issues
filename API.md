# Cosmetic Gear — Public API

**Package (Paket):** `com.cosmeticgear.api`

**Main class (Ana sınıf):** `CosmeticGearApi`

**Minecraft:** 1.20.1  
**Mod Loader (Mod yükleyici):** Forge

## 1. Overview (Genel Bakış)

`CosmeticGearApi` is a server-authoritative API for integrating other Minecraft mods with Cosmetic Gear.

Read methods accept any `Player`. Mutation methods require the target `ServerPlayer`.

Equipment methods accept either a zero-based slot index or an `EquipmentSlot`.

| Slot Index | Equipment Slot |
|---|---|
| `0` | HEAD |
| `1` | CHEST |
| `2` | LEGS |
| `3` | FEET |

## 2. Equipment Management (Ekipman Yönetimi)

### Available Methods (Kullanılabilir Metotlar)

- `getEquipment(player, slot)` — Returns a defensive copy of the stored cosmetic `ItemStack`.
- `setEquipment(serverPlayer, slot, stack)` — Validates equipment compatibility and writes only into an empty cosmetic slot. Returns whether data changed.
- `removeEquipment(serverPlayer, slot)` — Clears the selected cosmetic slot and returns a copy of the previous item.

**Important:** `setEquipment` never silently overwrites an occupied slot.

Pass `ItemStack.EMPTY` to clear a slot.

To replace an existing item, first remove the previous item and safely retain or return it before equipping the replacement.

## 3. Visibility Management (Görünürlük Yönetimi)

### Cosmetic Armor Visibility (Kozmetik Zırh Görünürlüğü)

- `isCosmeticVisible`
- `setCosmeticVisible`

These methods read or modify individual cosmetic slot visibility.

### Real Armor Visibility (Gerçek Zırh Görünürlüğü)

- `isRealArmorVisible`
- `setRealArmorVisible`
- `setAllRealArmorVisible`

These methods control real armor rendering independently of cosmetic armor.

### Global Cosmetic Visibility (Genel Kozmetik Görünürlük)

- `isCosmeticsHidden`
- `setCosmeticsHidden`

These methods read or change the global cosmetic visibility flag.

## 4. Server-Side Usage (Sunucu Tarafında Kullanım)

Mutation methods must be called on the server thread.

```java
// Read a defensive copy on either logical side.
ItemStack shown = CosmeticGearApi.getEquipment(
    player,
    EquipmentSlot.HEAD
);

boolean realHelmetShown =
    CosmeticGearApi.isRealArmorVisible(player, 0);

// Mutate only from server-side integration code.
if (serverPlayer.getServer().isSameThread()) {
    CosmeticGearApi.setEquipment(
        serverPlayer,
        EquipmentSlot.HEAD,
        new ItemStack(Items.GOLDEN_HELMET)
    );

    CosmeticGearApi.setRealArmorVisible(
        serverPlayer,
        0,
        false
    );
}
```

Each successful mutation marks the player's saved capability dirty.

The server broadcasts one state snapshot on the next player tick to the affected player and tracking clients.

Repeating the same value does not mark the state dirty or produce another event.

The API does not expose a mutation method targeting a client player.

## 5. Forge Events (Forge Olayları)

Successful mutations post public Forge events.

| Event | Description |
|---|---|
| `EquipmentChangedEvent` | Cosmetic equipment slot changed |
| `VisibilityChangedEvent` | Individual visibility value changed |
| `AllCosmeticsVisibilityChangedEvent` | Global cosmetic visibility changed |

### Equipment Change Event (Ekipman Değişikliği Olayı)

```java
@SubscribeEvent
public static void onCosmeticChanged(
    CosmeticGearApi.EquipmentChangedEvent event
) {
    ItemStack previous = event.oldStack();
    ItemStack current = event.newStack();
}
```

Both `oldStack()` and `newStack()` return copies.

### Other Event Data (Diğer Olay Verileri)

`VisibilityChangedEvent` provides the changed slot, `VisibilityKind`, previous value, and new value.

`AllCosmeticsVisibilityChangedEvent` provides the previous and new global cosmetic visibility values.

## 6. Client Rendering Integration (İstemci Görsel İşleme Entegrasyonu)

Client renderers running inside a player render layer can read the appearance selected by Cosmetic Gear through:

- `Player.getItemBySlot`
- `Player.getArmorSlots`

The same appearance view is available within Forge's `RenderHandEvent`.

These reads affect appearance for the current player only.

### Rendering Outside Callbacks (Olay Çağrıları Dışında Görsel İşleme)

Use:

```java
GearVisuals.equipment(player, slot);
```

This method returns the appearance selected by Cosmetic Gear.

Use:

```java
GearVisuals.cosmeticEquipment(player, slot);
```

This method allows the renderer to distinguish active cosmetic equipment from visible real armor.

**Class (Sınıf):**

`com.cosmeticgear.client.render.GearVisuals`

**Important:**

- Call these methods only from client render code.
- Treat the returned `ItemStack` as read-only.
- Draw the selected item only once.
- Use your own rendering model, textures, and animation logic.

## 7. Integration Safety (Entegrasyon Güvenliği)

Mutation operations require a `ServerPlayer` and are intended for trusted server-side mod integrations.

Integrating mods must enforce their own gameplay permissions before changing another player's cosmetic equipment or visibility settings.

Do not invoke mutation operations from client-side rendering code.

Do not silently discard items returned by `removeEquipment`.

---

## Support and Issue Reporting (Destek ve Sorun Bildirimi)

Report integration problems, unexpected behavior, or compatibility issues through the official issue tracker.

[Cosmetic Gear — GitHub Issues](https://github.com/yasindemircw00-web/CosmeticGear-Issues/issues/new/choose)
