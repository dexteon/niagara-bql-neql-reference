# Network Topology Queries

> **Goal:** Map all driver networks, inter-station connections, routing
> **Interface:** Workbench `CTRL+L`

---

## 1. Driver / Network Discovery

### All Driver Networks
```
station:|slot:/Drivers|bql:select name, displayName, status from driver:DeviceNetwork
```

### All Networks + Device Count
```
station:|slot:/Drivers|bql:select parent.displayName as 'Network', count(name) as 'Devices' from driver:Device
```

### All Components Under Drivers (full tree)
```
station:|slot:/Drivers|bql:select * from baja:Component
```

---

## 2. Inter-Station / Fox Connections

### All Fox Connections (remote stations)
```
station:|slot:/Drivers|bql:select * from fox:FoxClientDevice
```

### Fox Connections with Address + Status
```
station:|slot:/Drivers|bql:select name, displayName, address, status from fox:FoxClientDevice
```

---

## 3. Station Services Map

### All Services Running on Station
```
station:|slot:/Services|bql:select * from baja:Component
```

### Services + Status
```
station:|slot:/Services|bql:select name, displayName, status from baja:BObject
```

---

## 4. Platform / Station Info

### Station Config Components
```
station:|slot:/Config|bql:select * from baja:Component
```

### All Modules / Installed Software
```
station:|slot:/|bql:select * from baja:ModuleComponent
```
