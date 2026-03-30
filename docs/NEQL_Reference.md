# NEQL — Full Reference

> Niagara Entity Query Language — tag and relation-based filtering
> Requires: Niagara 4.4+, tagDictionary module installed

---

## Syntax

```
station:|slot:[scope]|neql:[dictionary]:[tagName]|bql:[statement]
```

NEQL inserts between the scope ORD and the BQL clause. It filters entities by **tag or relation** before BQL processes them.

---

## How NEQL Works

1. Scope ORD selects a subtree of the station
2. NEQL filters that subtree to only entities carrying the specified tag
3. BQL then queries the filtered set

```
station:|slot:/Drivers         ← scope: all drivers subtree
  |neql:n:point                ← filter: only entities tagged n:point
  |bql:select displayName, ord ← query: return these fields
```

---

## Tag Dictionaries

### n: — Niagara Standard Dictionary

| Tag | Type | Meaning |
|-----|------|---------|
| `n:point` | Marker | This entity is a control point |
| `n:equip` | Marker | This entity is equipment |
| `n:history` | Marker | This entity has history |
| `n:vendor` | String | Vendor name |
| `n:ordInSession` | Ord | ORD within current session |
| `n:childDevice` | Relation | Child device relation |

### hs: — Project Haystack Dictionary

| Tag | Type | Meaning |
|-----|------|---------|
| `hs:site` | Marker | A physical site |
| `hs:equip` | Marker | Equipment |
| `hs:point` | Marker | Data point |
| `hs:chiller` | Marker | Chiller equipment |
| `hs:ahu` | Marker | Air handling unit |
| `hs:vav` | Marker | VAV box |
| `hs:boiler` | Marker | Boiler |
| `hs:fan` | Marker | Fan |
| `hs:sensor` | Marker | Sensor point |
| `hs:cmd` | Marker | Command/output point |
| `hs:sp` | Marker | Setpoint |
| `hs:temp` | Marker | Temperature point |
| `hs:elec` | Marker | Electrical point |
| `hs:chilledWaterPlantRef` | Relation | Links chiller → chilled water plant |
| `hs:ahuRef` | Relation | Links point → AHU |
| `hs:siteRef` | Relation | Links equip/point → site |
| `hs:equipRef` | Relation | Links point → equipment |

### b: — Custom Baja Tags (User-Defined)

User-defined tags created in TagDictionaryService:
```
b:floor         BInteger    Floor number
b:zone          BString     Zone name
b:buildingId    BString     Building identifier
b:assetId       BString     External asset ID
```

---

## NEQL Query Examples

### All Niagara-Tagged Points
```
station:|slot:|neql:n:point|bql:select displayName, slotPath, ord
```

### All Haystack Sites
```
station:|slot:|neql:hs:site|bql:select displayName, slotPath
```

### All Haystack Equipment
```
station:|slot:|neql:hs:equip|bql:select displayName, slotPath
```

### Equipment Scoped to a Subtree
```
station:|slot:/Drivers/BacnetNetwork|neql:hs:equip|bql:select displayName, slotPath
```

### Count Tagged Assets
```
station:|slot:|neql:hs:equip|bql:select count(out)|cell:0,0
```

### Get Tag Value (Vykon Pro required)
```
station:|slot:/Drivers|bql:select name, vykonPro:Lib.tagValue('n:vendor') as 'Vendor' from baja:Component where vykonPro:Lib.hasTag('n:vendor') = 'true'
```

### Check Tag Existence (Vykon Pro)
```
station:|slot:/Drivers/BacnetNetwork|bql:select * from baja:Component where vykonPro:Lib.hasTag('n:history') = 'true'
```

---

## Relation Queries

Relations are directional links between entities. They cannot be queried via CTRL+L BQL — they require the Java Entity API.

### Concepts
- **Outbound** relation: this entity points TO another
- **Inbound** relation: another entity points TO this one
- Endpoint is always an ORD to the related entity

### Java API — Add a Relation
```java
Relation r = entity.relations().add(
    Id.newId("hs:ahuRef"),
    targetEntity
);
```

### Java API — Traverse a Relation
```java
// Find the chilled water plant a chiller belongs to
Id plantRef = Id.newId("hs", "chilledWaterPlantRef");
chiller.relations().get(plantRef, Relations.OUT).ifPresent(relation -> {
    Entity plant = relation.getEndpoint();
    System.out.println(plant.getOrdToEntity());
});
```

### Java API — Get All Related Entities
```java
// All child devices of a BACnet network
bacnetNetwork.relations()
    .getAll(Id.newId("n:childDevice"))
    .stream()
    .map(Relation::getEndpoint)
    .forEach(entity -> System.out.println(entity.getOrdToEntity()));
```

### Java API — Filter by Tag While Traversing
```java
Id chillerId = Id.newId("hs:chiller");
bacnetNetwork.relations()
    .getAll(Id.newId("n:childDevice"))
    .stream()
    .map(Relation::getEndpoint)
    .filter(entity -> entity.tags().get(chillerId).isPresent())
    .forEach(entity -> System.out.println(entity.getOrdToEntity()));
```

---

## Entity API Reference

```java
// javax.baja.tag.Entity
entity.tags()                           // Tags collection
entity.relations()                      // Relations collection
entity.getOrdToEntity()                 // Optional<BOrd>

// Tags
entity.tags().get(Id.newId("n:point"))  // Optional<Tag>
entity.tags().set(tag)                  // Add/update tag
entity.tags().remove(id)                // Remove tag

// Tag constructor
Tag.newTag("b:floor", BInteger.make(1))
Tag.newTag("n:point")                   // marker (no value)

// Relations
entity.relations().add(id, endpoint)    // Add relation
entity.relations().get(id, Relations.OUT)  // Get outbound
entity.relations().getAll(id)           // Get all with this id
relation.getEndpoint()                  // Entity at other end
relation.getEndpointOrd()               // BOrd to other end
relation.isOutbound()
relation.isInbound()
```

---

## Setting Up NEQL Tagging

1. Install `tagDictionary` module on your station
2. Open **TagDictionaryService** in Workbench
3. Add dictionaries (`n:`, `hs:`, or custom)
4. Navigate to each component → right-click → **Edit Tags**
5. Add marker tags or string/value tags as needed
6. Run NEQL queries as above

For automated bulk tagging use the Smart Tag Dictionary or import via CSV.
