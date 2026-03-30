# HTTP ORD API Queries

> **Goal:** Run BQL/NEQL queries over HTTP for external tooling, scripts, SIEM integration
> **Requires:** Basic Hx Profile enabled on the JACE/supervisor
> **Format:** `http://[ip]/ord?[url-encoded-ORD]`

---

## URL Encoding Reference

| Character | Encoded |
|-----------|---------|
| space | `%20` |
| `|` | `%7C` (sometimes needed) |
| `'` | `%27` |
| `*` | `%2A` |
| `:` | `%3A` (in values) |

---

## 1. Device Inventory via HTTP

### All Devices
```
http://[ip]/ord?station:|slot:/Drivers|bql:select%20*%20from%20driver:Device
```

### All Devices — Key Fields
```
http://[ip]/ord?station:|slot:/Drivers|bql:select%20slotPath,displayName,status%20from%20driver:Device
```

### Faulted or Down Devices
```
http://[ip]/ord?station:|slot:/Drivers|bql:select%20slotPath,displayName,status%20from%20driver:Device%20where%20status.fault%20=%20true%20or%20status.down%20=%20true
```

---

## 2. Point Inventory via HTTP

### All Control Points
```
http://[ip]/ord?station:|slot:/|bql:select%20slotPath,displayName,out,status%20from%20control:ControlPoint
```

### Points in a Specific Network
```
http://[ip]/ord?station:|slot:/Drivers/BacnetNetwork|bql:select%20slotPath,displayName,out.value,status%20from%20control:NumericPoint
```

---

## 3. BACnet Devices via HTTP

### All BACnet Devices
```
http://[ip]/ord?station:|slot:/Drivers|bql:select%20name,displayName,address,deviceId,status%20from%20bacnet:BacnetDevice
```

---

## 4. Modbus Devices via HTTP

### All Modbus Devices (TCP + RTU)
```
http://[ip]/ord?station:|slot:/Drivers|bql:select%20name,displayName,address,status%20from%20modbusCore:ModbusClientDevice
```

---

## 5. User / Security via HTTP

### All Users
```
http://[ip]/ord?station:|slot:/Services/UserService|bql:select%20name,displayName,enabled,roles%20from%20user:User
```

---

## 6. Python Example (requests)

```python
import requests

JACE_IP = "192.168.1.100"
USERNAME = "admin"
PASSWORD = "your_password"

query = "station:|slot:/Drivers|bql:select slotPath,displayName,status from driver:Device"
url = f"http://{JACE_IP}/ord?{query}"

response = requests.get(url, auth=(USERNAME, PASSWORD), verify=False)
print(response.text)
```

---

## 7. curl Example

```bash
# All devices
curl -u admin:password -k "http://192.168.1.100/ord?station:|slot:/Drivers|bql:select%20*%20from%20driver:Device"

# BACnet devices
curl -u admin:password -k "http://192.168.1.100/ord?station:|slot:/Drivers|bql:select%20name,address,deviceId,status%20from%20bacnet:BacnetDevice"
```
