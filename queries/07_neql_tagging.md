# NEQL — Tag & Relation Queries

> **Goal:** Query assets using Niagara's tag/relation model (N4.4+)
> **Interface:** Workbench `CTRL+L`
> **Requires:** `tagDictionary` module + N4.4 minimum

---

## 1. Basic NEQL Syntax

```
station:|slot:[scope]|neql:[dictionary]:[tagName]|bql:select [columns]
```

NEQL acts as a **pre-filter** — it selects entities by tag before BQL runs.

---

## 2. Niagara Standard Tags (n: dictionary)

### All Entities Tagged as Points
```
station:|slot:|neql:n:point|bql:select displayName, slotPath, ord
```

### All Entities Tagged as Equipment
```
station:|slot:|neql:n:equip|bql:select displayName, slotPath
```

### All Entities with History Tag
```
station:|slot:|neql:n:history|bql:select displayName, slotPath
```

### Points with ordInSession Tag
```
station:|slot:|neql:n:ordInSession|bql:select displayName, ordInSession, ord
```

### Entities with Vendor Tag — Get Tag Value (Vykon Pro)
```
station:|slot:/Drivers|bql:select name, vykonPro:Lib.tagValue('n:vendor') as 'Vendor' from baja:Component where vykonPro:Lib.hasTag('n:vendor') = 'true'
```

### Scoped to a Network
```
station:|slot:/Drivers/BacnetNetwork|neql:n:point|bql:select displayName, slotPath, ord
```

---

## 3. Project Haystack Tags (hs: dictionary)

### All Sites
```
station:|slot:|neql:hs:site|bql:select displayName, slotPath
```

### Count of All Sites
```
station:|slot:|neql:hs:site|bql:select count(out)|cell:0,0
```

### All Equipment
```
station:|slot:|neql:hs:equip|bql:select displayName, slotPath
```

### All Haystack Points
```
station:|slot:|neql:hs:point|bql:select displayName, slotPath
```

### Chiller Equipment
```
station:|slot:|neql:hs:chiller|bql:select displayName, slotPath
```

### AHU Equipment
```
station:|slot:|neql:hs:ahu|bql:select displayName, slotPath
```

### VAV Equipment
```
station:|slot:|neql:hs:vav|bql:select displayName, slotPath
```

### All Haystack Equipment + Point Count
```
station:|slot:|neql:hs:equip|bql:select displayName, slotPath, count(name) as 'Points'
```

---

## 4. Custom / User-Defined Tags

### Tag All Target Devices with Custom Marker

Use the Tags editor in Workbench to add tag `my:deviceForQuery` (marker) to each device you want to cross-query, then:

```
station:|slot:|neql:my:deviceForQuery|bql:select displayName, slotPath, status
```

### Query by Custom String Tag Value
```
station:|slot:|neql:my:building|bql:select displayName, slotPath
```

---

## 5. NEQL + BQL Combined Pipelines

### Tagged Points — Show Real-Time Value
```
station:|slot:/Drivers|neql:n:point|bql:select displayName, out, status
```

### Tagged Sites — Count Points Per Site
```
station:|slot:|neql:hs:site|bql:select displayName, count(name) as 'Points'
```

### NEQL Filter → BQL Count → Cell for PX Graphics
```
station:|slot:|neql:hs:equip|bql:select count(out)|cell:0,0
```

---

## 6. Relation Traversal (Java API — not CTRL+L)

Relations must be traversed in Java code, not via BQL/NEQL in Workbench.

```java
// Find the chilled water plant related to a chiller
Id plantRef = Id.newId("hs", "chilledWaterPlantRef");
chiller.relations().get(plantRef, Relations.OUT).ifPresent(relation -> {
    Entity plant = relation.getEndpoint();
    System.out.println(plant.getOrdToEntity());
});

// Get all BACnet devices tagged hs:chiller
Id chillerId = Id.newId("hs:chiller");
bacnetNetwork.relations().getAll(Id.newId("n:childDevice"))
    .stream()
    .map(Relation::getEndpoint)
    .filter(entity -> entity.tags().get(chillerId).isPresent())
    .forEach(entity -> System.out.println(entity.getOrdToEntity()));
```

---

## 7. Tag Dictionary Reference

| Prefix | Dictionary | Common Tags |
|--------|-----------|-------------|
| `n:` | Niagara built-in | `point`, `equip`, `history`, `vendor`, `ordInSession`, `childDevice` |
| `hs:` | Project Haystack | `site`, `equip`, `point`, `chiller`, `ahu`, `vav`, `boiler`, `fan`, `sensor`, `cmd`, `sp` |
| `b:` | Custom Baja | User-defined (e.g., `b:floor`, `b:zone`, `b:buildingId`) |
| `my:` | Site-specific | Anything your team defines in TagDictionaryService |

---

## 8. Adding Tags Programmatically (Java)

```java
// Add a direct tag
Tag floorTag = Tag.newTag("b:floor", BInteger.make(1));
entity.tags().set(floorTag);

// Add via Component model
myComponent.add(SlotPath.escape("b:floor"), BInteger.make(1), Flags.METADATA);

// Check if tag exists
boolean hasTag = entity.tags().get(Id.newId("n:point")).isPresent();

// Add a relation
Relation r = entity.relations().add(Id.newId("hs:ahuRef"), otherEntity);
```
