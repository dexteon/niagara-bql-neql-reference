# Protocol-Specific Queries

> **Goal:** Per-protocol device discovery with protocol-native fields
> **Interface:** Workbench `CTRL+L`

---

## BACnet

### All BACnet Devices
```
station:|slot:/Drivers|bql:select * from bacnet:BacnetDevice
```

### BACnet Devices with Address + Device ID
```
station:|slot:/Drivers|bql:select name, displayName, address, deviceId, status from bacnet:BacnetDevice
```

### BACnet Devices — Name, Address, Vendor Info
```
station:|slot:/Drivers|bql:select slotPath, displayName, address, deviceId, vendorName, modelName, applicationSoftwareVersion, status from bacnet:BacnetDevice
```

### BACnet Devices DOWN or FAULTED
```
station:|slot:/Drivers|bql:select slotPath, displayName, address, deviceId, status from bacnet:BacnetDevice where status.down = true or status.fault = true
```

### BACnet Device ID Only (for external correlation)
```
station:|slot:/Drivers|bql:select name, deviceId.getInstanceNumber as 'Instance', address as 'Network Address' from bacnet:BacnetDevice
```

### BACnet Points (all)
```
station:|slot:/Drivers/BacnetNetwork|bql:select slotPath, displayName, out, status from control:ControlPoint
```

### BACnet Network Config
```
station:|slot:/Drivers|bql:select * from bacnet:BacnetNetwork
```

---

## Modbus

### All Modbus Devices — TCP
```
station:|slot:/Drivers|bql:select * from modbusTcp:ModbusTcpDevice
```

### All Modbus Devices — Async (RTU/Serial)
```
station:|slot:/Drivers|bql:select * from modbusAsync:ModbusAsyncDevice
```

### All Modbus Devices — Both TCP + Async (common parent)
```
station:|slot:/Drivers|bql:select * from modbusCore:ModbusClientDevice
```

### Modbus Devices with Address
```
station:|slot:/Drivers|bql:select name, displayName, address, unitId, status from modbusCore:ModbusClientDevice
```

### Modbus TCP Devices — IP + Port
```
station:|slot:/Drivers|bql:select name, displayName, ipAddress, port, unitId, status from modbusTcp:ModbusTcpDevice
```

### Modbus Devices DOWN
```
station:|slot:/Drivers|bql:select slotPath, displayName, status from modbusCore:ModbusClientDevice where status.down = true
```

### Modbus Points
```
station:|slot:/Drivers/ModbusTcpNetwork|bql:select slotPath, displayName, out, status from control:ControlPoint
```

---

## LonWorks

### All LonWorks Devices
```
station:|slot:/Drivers|bql:select * from lon:LonDevice
```

### LonWorks Devices + Neuron ID + Status
```
station:|slot:/Drivers|bql:select name, displayName, neuronId, status from lon:LonDevice
```

### LonWorks Network Config
```
station:|slot:/Drivers|bql:select * from lon:LonNetwork
```

### LonWorks Devices DOWN
```
station:|slot:/Drivers|bql:select slotPath, displayName, neuronId, status from lon:LonDevice where status.down = true
```

### LonWorks Points
```
station:|slot:/Drivers/LonNetwork|bql:select slotPath, displayName, out, status from control:ControlPoint
```

---

## SNMP

### All SNMP Devices
```
station:|slot:/Drivers|bql:select * from snmp:SnmpDevice
```

### SNMP Devices with IP + OID
```
station:|slot:/Drivers|bql:select name, displayName, address, status from snmp:SnmpDevice
```

### SNMP Devices DOWN
```
station:|slot:/Drivers|bql:select slotPath, displayName, address, status from snmp:SnmpDevice where status.down = true
```

---

## KNX

### All KNX Devices
```
station:|slot:/Drivers|bql:select * from knx:KnxDevice
```

### KNX Devices + Address + Status
```
station:|slot:/Drivers|bql:select name, displayName, address, status from knx:KnxDevice
```

---

## OPC UA / OPC DA

### All OPC UA Devices
```
station:|slot:/Drivers|bql:select * from opcClient:OpcClientDevice
```

### OPC UA Devices + Endpoint + Status
```
station:|slot:/Drivers|bql:select name, displayName, endpointUrl, status from opcClient:OpcClientDevice
```

### OPC UA Points
```
station:|slot:/Drivers/OpcNetwork|bql:select slotPath, displayName, out, status from control:ControlPoint
```

---

## Niagara-to-Niagara (Fox)

### All Fox Client Connections (remote stations)
```
station:|slot:/Drivers|bql:select * from fox:FoxClientDevice
```

### Fox Connections with Address + Status
```
station:|slot:/Drivers|bql:select name, displayName, address, status from fox:FoxClientDevice
```

### Fox Connections DOWN
```
station:|slot:/Drivers|bql:select slotPath, displayName, address, status from fox:FoxClientDevice where status.down = true
```

---

## Multi-Protocol Cross-Query

### All Devices Across ALL Protocols (universal)
```
station:|slot:/Drivers|bql:select slotPath as 'Path', displayName as 'Name', parent.displayName as 'Network', status as 'Status' from driver:Device
```

### Devices Across Multiple Specific Protocols
BQL does not support UNION — run separately per type or use the universal `driver:Device` query above.

### NEQL-Based Cross-Protocol (requires tagging)
```
station:|slot:|neql:my:deviceForQuery|bql:select displayName, slotPath, status
```
> Tag each device with a custom `my:deviceForQuery` marker tag, then query all at once regardless of protocol.

---

## Type Reference — Common Protocol Types

| Protocol | BQL FROM Type | Notes |
|----------|--------------|-------|
| Any/All | `driver:Device` | Universal parent |
| BACnet | `bacnet:BacnetDevice` | |
| Modbus TCP | `modbusTcp:ModbusTcpDevice` | |
| Modbus RTU | `modbusAsync:ModbusAsyncDevice` | |
| Modbus (both) | `modbusCore:ModbusClientDevice` | Common parent |
| LonWorks | `lon:LonDevice` | JENEsys uses this heavily |
| SNMP | `snmp:SnmpDevice` | |
| KNX | `knx:KnxDevice` | |
| OPC UA | `opcClient:OpcClientDevice` | |
| Fox (N4-N4) | `fox:FoxClientDevice` | |
| Any network | `driver:DeviceNetwork` | Network containers |
| Any component | `baja:Component` | Broadest type |
