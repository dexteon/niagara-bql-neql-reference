# ORD Encoding Rules

> Special characters in slot names and ORD strings must be encoded.

---

## Slot Name Encoding

| Character | Encoded Form | Notes |
|-----------|-------------|-------|
| `:` | `$3a` | Common in tag slot names like `n:history` |
| ` ` (space) | `$20` | |
| `-` | `$2d` | Common in device names |
| `#` | `$23` | |
| `$` | `$24` | Escape the escape char itself |
| `/` | `$2f` | Forward slash |
| `@` | `$40` | |

## BQL SlotExists — Double Encoding

When using `slotExists()` in a BQL WHERE clause, the `$` in the encoding itself must be re-escaped:

```
Slot name:  n:history
In slot:    n$3ahistory      (: → $3a)
In BQL:     n$243ahistory    ($ → $24, so $3a → $243a)
```

### Example
```
-- Find points that have a slot named n:history
station:|slot:/Drivers|bql:select * from control:NumericPoint where slotExists('n$243ahistory')

-- Find points with a slot named n:point
station:|slot:/Drivers|bql:select * from baja:Component where slotExists('n$243apoint')
```

---

## URL Encoding (HTTP ORD Queries)

For HTTP `/ord?` queries, spaces and some characters must be URL-encoded:

| Character | URL Encoded |
|-----------|------------|
| space | `%20` |
| `'` | `%27` |
| `*` | `%2A` |

### Example
```
-- Workbench query
station:|slot:/Drivers|bql:select slotPath, displayName, status from driver:Device where status.fault = true

-- HTTP version
http://192.168.1.100/ord?station:|slot:/Drivers|bql:select%20slotPath,displayName,status%20from%20driver:Device%20where%20status.fault%20=%20true
```

---

## Slot Path Special Characters

Device and folder names with special characters in the slot path:

```
-- Dash in name: IO-R-34 → IO$2dR$2d34
station:|slot:/Drivers/NrioNetwork/IO$2dR$2d34/points|bql:select *

-- Space in name: My Device → My$20Device
station:|slot:/Drivers/My$20Device|bql:select *
```
