# Real-Time Status Queries

> **Goal:** Live device health, comms status, fault/down detection
> **Interface:** Workbench `CTRL+L`

---

## 1. Device Health — All Protocols

### All Devices + Live Status
```
station:|slot:/Drivers|bql:select slotPath as 'Path', displayName as 'Device', status as 'Status' from driver:Device
```

### Devices Currently DOWN
```
station:|slot:/Drivers|bql:select slotPath, displayName, status from driver:Device where status.down = true
```

### Devices Currently FAULTED
```
station:|slot:/Drivers|bql:select slotPath, displayName, status from driver:Device where status.fault = true
```

### DOWN or FAULTED (either)
```
station:|slot:/Drivers|bql:select slotPath as 'Path', displayName as 'Device', status as 'Status' from driver:Device where status.down = true or status.fault = true
```

### Disabled Devices
```
station:|slot:/Drivers|bql:select slotPath, displayName, enabled, status from driver:Device where enabled = false
```

### Count: DOWN Devices
```
station:|slot:/Drivers|bql:select count(name) as 'Down' from driver:Device where status.down = true|cell:0,0
```

### Count: FAULTED Devices
```
station:|slot:/Drivers|bql:select count(name) as 'Faulted' from driver:Device where status.fault = true|cell:0,0
```

### Count: Healthy (no fault, no down)
```
station:|slot:/Drivers|bql:select count(name) as 'Healthy' from driver:Device where status.down = false and status.fault = false|cell:0,0
```

---

## 2. Point-Level Status

### All Points with ANY Status Flag Set
```
station:|slot:/|bql:select slotPath, displayName, out, status from control:ControlPoint where status != {ok}
```

### Stale Points (no update received)
```
station:|slot:/|bql:select slotPath, displayName, status from control:ControlPoint where status.stale = true
```

### Overridden Points
```
station:|slot:/|bql:select slotPath, displayName, out, status from control:ControlPoint where status.overridden = true
```

### Points in NULL State
```
station:|slot:/|bql:select slotPath, displayName, status from control:ControlPoint where status.null = true
```

### Points in Down State
```
station:|slot:/|bql:select slotPath, displayName, status from control:ControlPoint where status.down = true
```

### Points — All Status Flags at Once
```
station:|slot:/Drivers|bql:select slotPath, displayName, out, status from control:ControlPoint
```

---

## 3. Network / Driver Health

### All Networks + Status
```
station:|slot:/Drivers|bql:select name, displayName, status from driver:DeviceNetwork
```

### Faulted Networks
```
station:|slot:/Drivers|bql:select name, displayName, status from driver:DeviceNetwork where status.fault = true
```

---

## 4. Last Communication Time

### Devices with LastOkTime Extension
```
station:|slot:/Drivers|bql:select slotPath, displayName, lastOkTime, status from driver:Device
```

### Devices NOT heard from (lastOkTime old — adjust AbsTime to your threshold)
```
station:|slot:/Drivers|bql:select slotPath, displayName, lastOkTime from driver:Device where lastOkTime < AbsTime '2024-01-01T00:00:00.000-0'
```

---

## 5. JENEsys ProBuilder Specific

### All JENEsys Controller I/O Points + Status
```
station:|slot:/Drivers|bql:select slotPath, displayName, out, status from control:ControlPoint where status.down = true or status.fault = true or status.stale = true
```

### ProBuilder Network Status Summary
```
station:|slot:/Drivers|bql:select parent.displayName as 'Network', count(name) as 'Total', status as 'Net Status' from driver:Device
```
