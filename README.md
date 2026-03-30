# Niagara 4 BQL/NEQL — Asset Discovery & Security Reference

> **Platform:** Tridium Niagara 4 / JENEsys ProBuilder
> **Interface:** Workbench → `CTRL+L` (Open ORD / Query Dialog)
> **Scope:** Asset discovery, device inventory, network topology, real-time status, security posture — NO alarm data, NO trend history

---

## How to Use

1. Open Workbench
2. Press **`CTRL+L`** to open the ORD query bar
3. Paste any query from this repo
4. Replace `[stationName]` with your actual station name where indicated
5. Press Enter — results open in a Collection Table

For remote stations prefix every query with:
```
local:|fox:|station:|
```

---

## Repository Structure

| File | Purpose |
|------|---------|
| [`queries/01_device_inventory.md`](queries/01_device_inventory.md) | Enumerate all devices across every protocol |
| [`queries/02_point_inventory.md`](queries/02_point_inventory.md) | Enumerate all control points, I/O, virtual |
| [`queries/03_network_topology.md`](queries/03_network_topology.md) | Driver/network map, routing, inter-station |
| [`queries/04_security_access.md`](queries/04_security_access.md) | Users, roles, permissions, audit trail |
| [`queries/05_realtime_status.md`](queries/05_realtime_status.md) | Device health: fault, down, stale, comms fail |
| [`queries/06_protocol_specific.md`](queries/06_protocol_specific.md) | BACnet, Modbus, LonWorks, SNMP, KNX, OPC per-protocol queries |
| [`queries/07_neql_tagging.md`](queries/07_neql_tagging.md) | NEQL tag/relation queries for tagged assets |
| [`queries/08_http_api.md`](queries/08_http_api.md) | HTTP ORD endpoints for external tooling/scripts |
| [`docs/BQL_Syntax.md`](docs/BQL_Syntax.md) | Full BQL language reference |
| [`docs/NEQL_Reference.md`](docs/NEQL_Reference.md) | Full NEQL tag/relation reference |
| [`docs/Type_Hierarchy.md`](docs/Type_Hierarchy.md) | Niagara type hierarchy for FROM clauses |
| [`docs/Encoding.md`](docs/Encoding.md) | ORD encoding rules (colons, spaces, special chars) |

---

## Quick Reference — Most Useful Discovery Queries

```
-- ALL devices, all protocols
station:|slot:/Drivers|bql:select * from driver:Device

-- ALL control points, station-wide
station:|slot:/|bql:select slotPath, displayName, out, status from control:ControlPoint

-- ALL BACnet devices with address
station:|slot:/Drivers|bql:select name, address, deviceId, status from bacnet:BacnetDevice

-- ALL Modbus devices (TCP + RTU/Async)
station:|slot:/Drivers|bql:select * from modbusCore:ModbusClientDevice

-- Devices currently DOWN or FAULTED
station:|slot:/Drivers|bql:select slotPath, displayName, status from driver:Device where status.fault = true or status.down = true

-- Count all devices per driver network
station:|slot:/Drivers|bql:select parent.displayName as 'Network', count(name) as 'DeviceCount' from driver:Device

-- All users configured on station
station:|slot:/Services/UserService|bql:select * from user:User

-- NEQL: all tagged n:point assets
station:|slot:|neql:n:point|bql:select displayName, slotPath, ord
```

---

## JENEsys ProBuilder Notes

- JENEsys controllers run Niagara N4 — all BQL/NEQL syntax is identical
- Default station name is typically the controller hostname
- Access via Workbench → `CTRL+L` or `File > Open > Open Query`
- ProBuilder often uses `lonworks:` driver — see `06_protocol_specific.md`
- HTTP ORD queries require Basic Hx Profile enabled on the JACE

---

## Sources

- [Tridium Official Docs](https://docs.niagara-community.com)
- [mrupperman — Niagara Sample BQL Queries](https://gist.github.com/mrupperman/8a0761bbb416b8ef1ca4f51c228f63bf)
- [thomasjupe — NiagaraN4BQLQueries](https://github.com/thomasjupe/NiagaraN4BQLQueries)
- [iC Knowledge Base — Multi-Protocol BQL](https://icdocs.ismacontrolli.com/kb/article/niagara-network-protocols-bql-queries-for-device-i)
- [Tridium Summit — NEQL Tags & Relations PDF](https://pages1.tridium.com/rs/808-SGM-271/images/EntitiesTagsNEQL.pdf)
