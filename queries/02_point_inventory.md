# Point Inventory Queries

> **Goal:** Enumerate all control points, I/O points, virtual points across the station
> **Interface:** Workbench `CTRL+L`

---

## 1. All Points — Station Wide

### Every Control Point
```
station:|slot:/|bql:select slotPath, displayName, out, status from control:ControlPoint
```

### All Numeric Points
```
station:|slot:/|bql:select slotPath, displayName, out.value as 'Value', status from control:NumericPoint
```

### All Boolean Points
```
station:|slot:/|bql:select slotPath, displayName, out as 'State', status from control:BooleanPoint
```

### All String Points
```
station:|slot:/|bql:select slotPath, displayName, out as 'Value', status from control:StringPoint
```

### All Enum Points
```
station:|slot:/|bql:select slotPath, displayName, out as 'Value', status from control:EnumPoint
```

---

## 2. Scoped Point Queries

### Points Under a Specific Driver
```
station:|slot:/Drivers/BacnetNetwork|bql:select slotPath, displayName, out.value, status from control:NumericPoint
```

### Points Under a Specific Device
```
station:|slot:/Drivers/BacnetNetwork/MyDevice|bql:select slotPath, displayName, out, status from control:ControlPoint
```

### Points Under a Folder
```
station:|slot:/VAV_Building_1|bql:select slotPath as 'Path', displayName as 'Name', out.value as 'Value' from control:NumericPoint
```

---

## 3. Point Filtering

### Points with Overrides Active
```
station:|slot:/|bql:select slotPath, displayName, out, status from control:ControlPoint where status.overridden = true
```

### Points in NULL/No Value State
```
station:|slot:/|bql:select slotPath, displayName, status from control:ControlPoint where status.null = true
```

### Points with Unacknowledged Conditions
```
station:|slot:/|bql:select slotPath, displayName, out, status from control:ControlPoint where status.unackedAlarm = true
```

### Points in a Specific Value Range
```
station:|slot:/|bql:select slotPath, displayName, out.value from control:NumericPoint where out.value > 100
```
```
station:|slot:/|bql:select slotPath, displayName, out.value from control:NumericPoint where out.value < 0
```

---

## 4. Point Metadata / Configuration

### Points with History Extensions (historized points)
```
station:|slot:/|bql:select parent.slotPath as 'Point Path', parent.displayName as 'Name' from history:HistoryExt
```

### Points with Their Facets (units, precision, min/max)
```
station:|slot:/Drivers|bql:select slotPath, displayName, out.value, facets from control:NumericPoint
```

### Points with a Specific Slot Existing
```
station:|slot:/Drivers|bql:select slotPath, displayName from control:ControlPoint where slotExists('emergencyActive') = true
```

### Points with Colon-Named Slot (e.g., n:history)
```
station:|slot:/Drivers|bql:select * from control:NumericPoint where slotExists('n$243ahistory')
```

---

## 5. Point Counts

### Total Point Count
```
station:|slot:/|bql:select count(name) as 'Total Points' from control:ControlPoint|cell:0,0
```

### Count by Point Type
```
station:|slot:/|bql:select count(name) as 'Numeric Points' from control:NumericPoint|cell:0,0
```
```
station:|slot:/|bql:select count(name) as 'Boolean Points' from control:BooleanPoint|cell:0,0
```

### Points Per Device (parent grouping)
```
station:|slot:/Drivers/BacnetNetwork|bql:select parent.displayName as 'Device', count(name) as 'Points' from control:ControlPoint
```

---

## 6. Tag-Augmented Point Queries (NEQL Required)

### All Points Tagged n:point
```
station:|slot:|neql:n:point|bql:select displayName, slotPath, ord
```

### Points Tagged with Haystack equip
```
station:|slot:|neql:hs:equip|bql:select displayName, slotPath
```

### Points with Specific Tag — Get Tag Value
```
station:|slot:/Drivers|bql:select name, vykonPro:Lib.tagValue('n:vendor') as 'Vendor' from baja:Component where vykonPro:Lib.hasTag('n:vendor') = 'true'
```
> Requires Vykon Pro module

### All Points with n:ordInSession Tag
```
station:|slot:/Drivers|neql:n:ordInSession|bql:select displayName, ordInSession, ord
```
