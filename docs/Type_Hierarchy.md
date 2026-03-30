# Niagara Type Hierarchy for BQL FROM Clauses

> Use this to find the right type string for `from <type>` in BQL queries.
> Broader (parent) types return more results. Narrower (child) types are more specific.

---

## How Types Map to BQL

Java class name → BQL type:
```
BModbusTcpDevice   →   modbusTcp:ModbusTcpDevice
BBacnetDevice      →   bacnet:BacnetDevice
BLonDevice         →   lon:LonDevice
```

Rule: remove `B` prefix, lowercase the module prefix, join with `:`.

---

## Core Component Hierarchy

```
baja:Component
└── baja:BObject
    └── driver:Device                    ← ALL devices, all protocols
        ├── bacnet:BacnetDevice          ← BACnet
        ├── modbusTcp:ModbusTcpDevice    ← Modbus TCP
        ├── modbusAsync:ModbusAsyncDevice← Modbus RTU/ASCII
        │   (both inherit from modbusCore:ModbusClientDevice)
        ├── lon:LonDevice                ← LonWorks
        ├── snmp:SnmpDevice              ← SNMP
        ├── knx:KnxDevice                ← KNX
        ├── opcClient:OpcClientDevice    ← OPC UA/DA
        ├── fox:FoxClientDevice          ← Niagara Fox (N4-N4)
        └── [vendor-specific devices]

driver:DeviceNetwork                     ← Network containers
├── bacnet:BacnetNetwork
├── modbusTcp:ModbusTcpNetwork
├── modbusAsync:ModbusAsyncNetwork
├── lon:LonNetwork
├── snmp:SnmpNetwork
└── fox:FoxNetwork
```

---

## Control Point Hierarchy

```
control:ControlPoint                     ← ALL points
├── control:NumericPoint                 ← float/integer values
├── control:BooleanPoint                 ← true/false
├── control:StringPoint                  ← text
├── control:EnumPoint                    ← enumerated states
└── [protocol proxy points]
    ├── bacnet:BacnetProxyExt (ext, not point itself)
    └── modbusTcp:ModbusTcpProxyExt
```

---

## Common Protocol Type Quick Reference

| Protocol | Device Type | Network Type |
|----------|------------|-------------|
| **Any** | `driver:Device` | `driver:DeviceNetwork` |
| **BACnet** | `bacnet:BacnetDevice` | `bacnet:BacnetNetwork` |
| **Modbus TCP** | `modbusTcp:ModbusTcpDevice` | `modbusTcp:ModbusTcpNetwork` |
| **Modbus RTU** | `modbusAsync:ModbusAsyncDevice` | `modbusAsync:ModbusAsyncNetwork` |
| **Modbus (both)** | `modbusCore:ModbusClientDevice` | — |
| **LonWorks** | `lon:LonDevice` | `lon:LonNetwork` |
| **SNMP** | `snmp:SnmpDevice` | `snmp:SnmpNetwork` |
| **KNX** | `knx:KnxDevice` | `knx:KnxNetwork` |
| **OPC UA** | `opcClient:OpcClientDevice` | `opcClient:OpcClientNetwork` |
| **Fox (N4→N4)** | `fox:FoxClientDevice` | `fox:FoxNetwork` |
| **M-Bus** | `mbus:MbusDevice` | `mbus:MbusNetwork` |
| **Ethernet/IP** | `ethernetIp:EthernetIpDevice` | `ethernetIp:EthernetIpNetwork` |
| **DNP3** | `dnp3:Dnp3Device` | `dnp3:Dnp3Network` |
| **DALI** | `dali:DaliDevice` | `dali:DaliNetwork` |

---

## Finding Unknown Types

When a device type is unknown:

1. Navigate to the device in Workbench Component Browser
2. Open the **Slot Sheet** view (right-click → Slot Sheet)
3. The type field shows the Java class name
4. Convert: `BFooBarDevice` → `fooBar:FooBarDevice`

Or use Bajadoc (Help → Bajadoc) to search class names and find parent/child relationships.

---

## Service Types

```
station:|slot:/Services|bql:select * from baja:Component

Common service types:
user:UserService          ← user management
alarm:AlarmService        ← alarm service
history:HistoryService    ← history/logging
fox:FoxService            ← Fox inter-station comms
web:WebService            ← HTTP/web access
schedule:ScheduleService  ← scheduling
program:ProgramService    ← program objects
```
