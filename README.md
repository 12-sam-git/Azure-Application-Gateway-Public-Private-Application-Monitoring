# Azure-Application-Gateway-Public-Private-Application-Monitoring
# 🚀 Azure Application Gateway – Public & Private Application Monitoring 

## 📌 Project Overview

This project demonstrates a **production-style Azure architecture** where:

* Public and Private web applications are hosted inside the same Azure Virtual Network (VNet).
* All traffic is routed through a **single Azure Application Gateway**.
* Applications communicate with API servers and a shared Azure SQL Database.
* Monitoring is implemented using **Application Insights**.
* Private application availability is achieved using **custom telemetry monitoring**.

---

## 🎯 Objective

Implement an Azure architecture that supports:

* 🌐 Public Web Application access
* 🔒 Private Internal Application access
* ⚙️ API Layer for business logic
* 🗄 Shared SQL Database
* 📊 Centralized monitoring dashboard

---

## 🏗 Architecture

```
Internet Users
        ↓
Azure Application Gateway
   ├── Public Listener → Public Web VM → Public API VM
   └── Private Listener → Private Web VM → Private API VM
                               ↓
                         Azure SQL Database

Monitoring:
Application Insights
   ├── Standard Availability Test (Public)
   └── Custom Availability Monitoring (Private)
```

---

## 🧱 Azure Resources Used

### Resource Group

* `RG-AppGateway-Project`

### Networking

* Virtual Network: `VNet-AppGateway`
* Subnets:

  * AppGatewaySubnet
  * PublicSubnet
  * PrivateSubnet
  * APISubnet

### Virtual Machines

| VM Name       | Purpose                 |
| ------------- | ----------------------- |
| VM-PublicWeb  | Public Web Application  |
| VM-PrivateWeb | Private Web Application |
| VM-PublicAPI  | Public API              |
| VM-PrivateAPI | Private API             |

### Database

* Azure SQL Database: `SharedDB`

### Networking & Routing

* Azure Application Gateway (Standard v2)
* Public & Private Frontend IPs
* Dedicated listeners and routing rules

### Monitoring

* Application Insights
* Availability Tests
* Workbooks Dashboard
* Custom Azure Function Monitoring

---

## ⚙️ Implementation Steps

---

### 1️⃣ Virtual Network Setup

Created VNet with address space:

```
10.0.0.0/16
```

Subnets:

```
10.0.0.0/24  → AppGatewaySubnet
10.0.1.0/24  → PublicSubnet
10.0.2.0/24  → PrivateSubnet
10.0.3.0/24  → APISubnet
10.0.4.0/24    DBsubnet
10.0.5.0/24    func-subnet
```

---

### 2️⃣ VM Deployment

* Windows Server VMs deployed.
* IIS installed:

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

Created test files:

```
C:\inetpub\wwwroot\index.html
C:\inetpub\wwwroot\health.html
```

---

### 3️⃣ API Layer Setup

Two API VMs created:

* Public API
* Private API

Backend flow:

```
Web Application → API VM → SQL Database
```

---

### 4️⃣ Azure SQL Database

Created:

```
SharedDB
```

Connection string copied from:

```
SQL Database → Connection Strings → ADO.NET
```

Example:

```
Server=tcp:sqlsharedserver.database.windows.net,1433;
Initial Catalog=SharedDB;
User ID=<username>;
Password=<password>;
Encrypt=True;
```

---

### 5️⃣ Application Gateway Configuration

#### Basics

| Setting | Value            |
| ------- | ---------------- |
| Tier    | Standard v2      |
| Mode    | Autoscaling      |
| Subnet  | AppGatewaySubnet |

---

#### Frontend IPs

* Public Frontend IP
* Private Frontend IP

---

#### Backend Pools

* Backend-PublicWeb
* Backend-PrivateWeb

---

#### HTTP Settings

| Setting         | Value    |
| --------------- | -------- |
| Protocol        | HTTP     |
| Port            | 80       |
| Cookie Affinity | Disabled |

---

#### Health Probe

```
Path: /health.html
```

---

#### Listeners

* Public Listener (Public IP)
* Private Listener (Private IP)

---

#### Routing Rules

| Rule         | Listener         | Backend         |
| ------------ | ---------------- | --------------- |
| Public Rule  | Public Listener  | Public Backend  |
| Private Rule | Private Listener | Private Backend |

---

## 📊 Application Insights Monitoring

Created:

```
AppInsights-POC
```

### Availability Tests

| Test         | Target                 |
| ------------ | ---------------------- |
| Test-Public  | Public App Gateway IP  |
| Test-Private | Private App Gateway IP |

---

## ⚠️ Challenge Faced

Standard availability tests run from public Microsoft locations.

Result:

```
Private Endpoint → Not reachable → Failed availability
```

---

## 💡 Solution — Custom Private Monitoring

Implemented custom monitoring using:

* Azure Function (Timer Trigger)
* VNet Integration
* TrackAvailability() telemetry

---

### Monitoring Flow

```
Azure Function (Timer)
        ↓
Call Private URL
        ↓
Check Response
        ↓
Send Telemetry
        ↓
Application Insights (GREEN)
```

---

## 🧠 Core Monitoring Logic (Concept)

```csharp
var response = await httpClient.GetAsync(privateUrl);

var telemetry = new AvailabilityTelemetry
{
    Name = "Private-App-Test",
    Success = response.IsSuccessStatusCode,
    Timestamp = startTime,
    Duration = DateTimeOffset.UtcNow - startTime
};

telemetryClient.TrackAvailability(telemetry);
```

---

## 📈 Workbook Dashboard

Dashboard includes:

* Public Availability Tile
* Private Application Health
* Monitoring Explanation Section

Sample Query:

```kusto
availabilityResults
| where timestamp > ago(30m)
| summarize Availability = avg(toint(success)) * 100 by name
```

---

## 🏆 Final Outcome

✔ Public Application routed successfully
✔ Private Application internal routing working
✔ API → SQL communication successful
✔ Health probes healthy
✔ Monitoring dashboard created
✔ Custom private monitoring implemented

---

## 🧩 Key Learnings

* Standard availability tests cannot monitor private endpoints.
* Internal monitoring requires VNet-based telemetry.
* Application Gateway simplifies multi-app routing.
* Workbooks provide centralized observability.

---

## 🎯 30-Second Architect Summary

> A secure multi-tier Azure architecture was implemented using Application Gateway to route public and private applications within a single VNet. Web applications communicate with API servers backed by Azure SQL Database. Public monitoring uses standard availability tests, while private monitoring uses a VNet-integrated Azure Function that sends custom telemetry to Application Insights, enabling reliable internal availability tracking.

---

## 📂 Future Improvements

* HTTPS + SSL termination
* WAF-enabled Application Gateway
* Private Link monitoring
* Auto-scaling backend pools
* CI/CD deployment pipeline

---

## 👩‍💻 Author

**Sameeksha Y S**

Azure | Cloud | DevOps Enthusiast

---
