# Device Inventory Queries

> **Goal:** Enumerate every device on the station — all protocols, all networks
> **Interface:** Workbench `CTRL+L`

---

## 1. Universal Device Discovery

### All Devices — Every Protocol
```
station:|slot:/Drivers|bql:select * from driver:Device
```

### All Devices with Key Fields
```
station:|slot:/Drivers|bql:select slotPath as 'Path', displayName as 'Name', name as 'Slot Name', status as 'Status' from driver:Device
```

### All Devices Including Nested (Full Tree)
```
station:|slot:/|bql:select slotPath, displayName, name, status from driver:Device
```

### Count Devices Per Network Driver
```
station:|slot:/Drivers|bql:select parent.displayName as 'Network', count(name) as 'Device Count' from driver:Device
```

### Total Device Count (Single Cell — for PX Graphics)
```
station:|slot:/Drivers|bql:select count(name) as 'Total Devices' from driver:Device|cell:0,0
```

---

## 2. Device Status Snapshot

### All Devices + Current Status
```
station:|slot:/Drivers|bql:select slotPath as 'Path', displayName as 'Device', status as 'Status', enabled as 'Enabled' from driver:Device
```

### Only DOWN Devices
```
station:|slot:/Drivers|bql:select slotPath, displayName, status from driver:Device where status.down = true
```

### Only FAULTED Devices
```
station:|slot:/Drivers|bql:select slotPath, displayName, status from driver:Device where status.fault = true
```

### DOWN or FAULTED (Combined)
```
station:|slot:/Drivers|bql:select slotPath as 'Path', displayName as 'Name', status as 'Status' from driver:Device where status.down = true or status.fault = true
```

### COUNT of Faulted Devices (for dashboards)
```
station:|slot:/Drivers|bql:select count(name) as 'Faulted' from driver:Device where status.fault = true|cell:0,0
```

### Devices with STALE data
```
station:|slot:/|bql:select slotPath, displayName, status from control:ControlPoint where status.stale = true
```

---

## 3. Network-Scoped Discovery

### All Devices in a Specific Driver Network
```
station:|slot:/Drivers/BacnetNetwork|bql:select * from driver:Device
```
```
station:|slot:/Drivers/ModbusNetwork|bql:select * from driver:Device
```
```
station:|slot:/Drivers/LonWorksNetwork|bql:select * from driver:Device
```

### All Devices with Parent Network Name
```
station:|slot:/Drivers|bql:select parent.displayName as 'Network', name as 'Device', displayName as 'Label', status as 'Status' from driver:Device
```

---

## 4. Device Type / Hardware Identification

### All Driver Networks (not devices — the network containers)
```
station:|slot:/Drivers|bql:select name, displayName, status from driver:DeviceNetwork
```

### All Enabled vs Disabled Devices
```
station:|slot:/Drivers|bql:select slotPath, displayName, enabled, status from driver:Device where enabled = false
```

### Last OK Time (comms last seen alive) — requires LastOkTime extension
```
station:|slot:/Drivers|bql:select slotPath, displayName, lastOkTime, status from driver:Device
```

---

## 5. Remote Station (Fox Protocol)

Prefix any query with `local:|fox:|station:|` for remote JACE/controller access:

```
local:|fox:|station:|slot:/Drivers|bql:select * from driver:Device
```
```
local:|fox:|station:|slot:/Drivers|bql:select slotPath, displayName, status from driver:Device where status.down = true or status.fault = true
```

---

## 6. Notes on Type Resolution

If a query returns no results, the device type may not match. Steps to find the correct type:

1. Navigate to the device in Component Browser
2. Right-click → **Slot Sheet** (AX view) or check **Type** in property sheet
3. The type shown (e.g., `BModbusTcpDevice`) maps to BQL as `modbusTcp:ModbusTcpDevice`
4. Use parent classes for cross-protocol queries (see `docs/Type_Hierarchy.md`)

**Type naming rule:**
```
B[ModulePrefix][ClassName]
→ remove 'B' prefix
→ lowercase module prefix
→ add colon separator
→ result: modulePrefix:ClassName
```

Example:
```
BBacnetDevice → bacnet:BacnetDevice
BModbusTcpDevice → modbusTcp:ModbusTcpDevice
BLonDevice → lon:LonDevice
```
