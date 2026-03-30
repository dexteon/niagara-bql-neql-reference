# BQL — Full Syntax Reference

> Baja Query Language (BQL) is the SQL-like query language built into Tridium Niagara 4.
> All queries run in Workbench via `CTRL+L`.

---

## Query Structure

```
<scope-ord> | [neql:<tag>] | bql:<statement> | [output]
```

### Full BQL Statement
```sql
bql:select <projection> [from <type>] [where <predicate>] [order by <col> [asc|desc]]
```

---

## SELECT Projection

| Expression | Returns |
|-----------|---------|
| `*` | All slots |
| `name` | Slot name |
| `displayName` | Human-readable name |
| `slotPath` | Full component path |
| `ord` | Full ORD string |
| `out` | Current output (with status) |
| `out.value` | Numeric value only |
| `status` | Full status flags |
| `status.alarm` | Alarm flag |
| `status.fault` | Fault flag |
| `status.down` | Down/offline flag |
| `status.stale` | Stale data flag |
| `status.overridden` | Override active flag |
| `status.null` | Null value flag |
| `status.unackedAlarm` | Unacked alarm flag |
| `enabled` | Component enabled state |
| `facets` | Units, precision, min/max |
| `parent.name` | Parent slot name |
| `parent.displayName` | Parent display name |
| `parent.slotPath` | Parent path |
| `parent.out` | Parent output value |
| `parent.parent.displayName` | Grandparent display name |
| `col as 'Label'` | Rename column |

---

## Aggregate Functions

```sql
count(out)          -- count of rows matching
count(name)         -- same, use name for non-nullable count
avg(out)            -- average value
min(out)            -- minimum value
max(out)            -- maximum value
sum                 -- sum (history rollup context)
```

---

## FROM Clause — Type Filtering

Only components of the specified type (or subtype) are returned.

```sql
from baja:Component          -- any component (broadest)
from baja:BObject            -- any object
from driver:Device           -- all devices, all protocols
from driver:DeviceNetwork    -- network containers
from control:ControlPoint    -- all control points
from control:NumericPoint    -- numeric data points
from control:BooleanPoint    -- boolean data points
from control:StringPoint     -- string points
from control:EnumPoint       -- enum points
from alarm:AlarmSourceExt    -- alarm extensions
from alarm:AlarmClass        -- alarm class objects
from history:HistoryExt      -- history extensions
from user:User               -- user accounts
from user:UserRole           -- user roles
```

Protocol-specific types — see `queries/06_protocol_specific.md`.

---

## WHERE Predicates

### String Matching
```sql
where name = 'ExactName'
where displayName like '*partial*'
where displayName like 'Prefix*'
where displayName like '*Suffix'
where alarmClass like '*Critical*'
where slotPath like '/Drivers/Bacnet*'
where name != 'ExcludeThis'
```

### Boolean / Status Flags
```sql
where status.fault = true
where status.down = true
where status.stale = true
where status.overridden = true
where status.null = true
where status.alarm = true
where enabled = true
where enabled = false
```

### Numeric Comparison
```sql
where out.value > 72
where out.value < 0
where out.value >= 100
where out.value <= 50
```

### Compound Conditions
```sql
where status.down = true or status.fault = true
where displayName like '*ZN_T*' and status.fault = false
where name like '*Fan*' or name like '*Pump*'
```

### Parent / Ancestor Traversal
```sql
where parent.displayName = 'System A1'
where parent.parent.displayName like '*Chiller*'
```

### Slot Existence
```sql
where slotExists('emergencyActive') = true
where slotExists('n$243ahistory')          -- n:history (colon-encoded)
```

### Time Ranges (History/Audit context)
```sql
where timestamp in bqltime.today
where timestamp in bqltime.yesterday
where timestamp in bqltime.last7days
where timestamp in bqltime.lastweek
where timestamp in bqltime.lastmonth
where timestamp in bqltime.yeartodate
where timestamp >= AbsTime '2024-01-01T00:00:00.000-0'
where timestamp <= AbsTime '2024-12-31T23:59:59.000-0'
where timestamp.year = 2024
where timestamp.month.ordinal = '3'
```

---

## ORDER BY

```sql
order by timestamp desc
order by timestamp asc
order by displayName asc
order by out.value desc
order by alarmClass, timestamp asc
```

---

## bqltime Constants

| Constant | Returns |
|----------|---------|
| `bqltime.now` | Current AbsTime |
| `bqltime.today` | Today AbsTimeRange |
| `bqltime.yesterday` | Yesterday |
| `bqltime.last24hours` | Last 24h |
| `bqltime.lastweek` | Last week |
| `bqltime.last7days` | Last 7 days |
| `bqltime.lastmonth` | Last month |
| `bqltime.lastyear` | Last year |
| `bqltime.weektodate` | Week to date |
| `bqltime.monthtodate` | Month to date |
| `bqltime.yeartodate` | Year to date |
| `bqltime.startofday` | Start of today |
| `bqltime.endofday` | End of today |
| `bqltime.startofweek` | Start of this week |
| `bqltime.startofmonth` | Start of this month |
| `bqltime.startofyear` | Start of this year |
| `bqltime.weekday` | Current weekday |
| `bqltime.month` | Current month |

---

## Output Modifiers

### Cell Selector (extract single value for PX binding)
```
|cell:0,0        -- row 0, column 0 (first result, first column)
|cell:0,1        -- row 0, column 1
```

### Collection Table View (Workbench display)
```
|view:workbench:CollectionTable
```

### Size (count of results)
```
|bql:size
```

---

## Chaining Multiple BQL Stages

```
station:|slot:/Drivers|bql:select * from driver:Device|bql:select name, status
```

---

## Scope ORD Prefixes

| Prefix | Scope |
|--------|-------|
| `station:\|slot:/` | Root of station |
| `station:\|slot:/Drivers` | All drivers |
| `station:\|slot:/Services` | All services |
| `station:\|slot:/Config` | Station config |
| `history:/[stationName]` | History database |
| `alarm:` | Alarm database |
| `local:\|fox:\|station:\|slot:/` | Remote station (Fox) |

---

## Special Encoding

| Character | In slot names |
|-----------|--------------|
| `:` | `$3a` → in BQL re-escape `$` as `$24` → use `$243a` |
| space | `$20` |
| `-` | `$2d` |
| `#` | `$23` |

Example — slot `n:history`:
```
where slotExists('n$243ahistory')
```
