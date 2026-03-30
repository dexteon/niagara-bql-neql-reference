# Security & Access Queries

> **Goal:** Enumerate users, roles, permissions, audit trail, login/logout events
> **Interface:** Workbench `CTRL+L`
> **Note:** No alarm data — security posture and access control only

---

## 1. User Enumeration

### All Users on Station
```
station:|slot:/Services/UserService|bql:select * from user:User
```

### Users with Key Fields
```
station:|slot:/Services/UserService|bql:select name as 'Username', displayName as 'Display Name', enabled as 'Enabled', roles as 'Roles' from user:User
```

### Enabled Users Only
```
station:|slot:/Services/UserService|bql:select name, displayName, roles from user:User where enabled = true
```

### Disabled / Locked Out Users
```
station:|slot:/Services/UserService|bql:select name, displayName, enabled from user:User where enabled = false
```

### Count of All Users
```
station:|slot:/Services/UserService|bql:select count(name) as 'Total Users' from user:User|cell:0,0
```

---

## 2. Role Enumeration

### All Roles Configured
```
station:|slot:/Services/UserService|bql:select * from user:UserRole
```

### Roles + Permissions
```
station:|slot:/Services/UserService|bql:select name, displayName, permissions from user:UserRole
```

---

## 3. Audit Log — Access & Change Events

### Full Audit Log (All Events)
```
history:/[stationName]/AuditHistory|bql:select *
```

### Login & Logout Events — Last 7 Days
```
station:|history:/[stationName]/SecurityHistory?period=last7Days|bql:select timestamp.toDateString as 'Date', timestamp.toTimeString as 'Time', operation as 'Operation', userName as 'User' where operation like 'Login' or operation like 'Logout'
```

### All Write / Override Operations
```
history:/[stationName]/AuditHistory|bql:select * where operation like 'Invoked'
```

### All Override Operations with User + Target
```
history:/[stationName]/AuditHistory|bql:select timestamp as 'Time', userName as 'User', operation as 'Op', target as 'Target', value as 'Value' where operation like 'Invoked'
```

### Changes by a Specific User
```
history:/[stationName]/AuditHistory|bql:select timestamp, operation, target, value where userName = 'admin'
```

### All Config Changes (Modified operations)
```
history:/[stationName]/AuditHistory|bql:select timestamp, userName, operation, target, value where operation like 'Modified'
```

### Changes to a Specific Component Path
```
history:/[stationName]/AuditHistory|bql:select timestamp, userName, value where operation like 'Invoked' and target like '/Drivers/BacnetNetwork*'
```

### Audit Events in a Date Range
```
history:/[stationName]/AuditHistory|bql:select timestamp, userName, operation, target where timestamp >= AbsTime '2024-01-01T00:00:00.000-0' and timestamp <= AbsTime '2024-12-31T23:59:59.000-0'
```

---

## 4. Security Service State

### Security Service Config
```
station:|slot:/Services/SecurityService|bql:select * from baja:Component
```

### Certificate / TLS Components
```
station:|slot:/Services|bql:select * from baja:Component where name like '*Certificate*' or name like '*Tls*' or name like '*Ssl*'
```

---

## 5. HTTP / Web Service Access

### All Web Service Components
```
station:|slot:/Services|bql:select name, displayName, status from baja:Component where name like '*Web*' or name like '*Http*' or name like '*Fox*'
```

### Fox Service Config (inter-station comms)
```
station:|slot:/Services/FoxService|bql:select * from baja:Component
```
