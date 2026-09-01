# LAB 4: Integration & Capstone Project

## 📘 Overview

**Goal:** Combine Labs 1–3 into one secure, monitored environment where:

* VDI is reachable only through **ZTNA**.
* SCCM patch compliance is enforced over the same **ZTNA path**.
* Identity, ZTNA, VDI, SCCM, and monitoring data are integrated into a centralized environment.

---

## 📋 Prerequisites

| Prerequisite       | Details                                                                 |
| ------------------ | ----------------------------------------------------------------------- |
| **Completed labs** | Lab 1 (ZTNA), Lab 2 (VDI), and Lab 3 (SCCM) environments up and running |
| **Logging target** | A SIEM or log destination, such as Microsoft Sentinel, Splunk, or ELK   |

---

## 🔐 Part A: Secure the VDI Landing Zone with ZTNA

### Step 1: Secure the VDI Landing Zone with ZTNA

1. Remove or disable the legacy VPN or public RDP/gateway rule previously used to reach the VDI environment.

2. Publish the **VDI connection broker/gateway** as an application behind the **Lab 1 ZTNA connector**.

3. Apply the same **MFA + device-compliance** access policy from Lab 1 to the VDI application.

4. Scope the policy to your **VDI user group**.

The expected access flow is:

```text
User
  ↓
Identity Verification
  ↓
MFA + Device Compliance
  ↓
ZTNA Connector
  ↓
VDI Connection Broker/Gateway
  ↓
VDI Environment
```

> **⚠️ Warning**
>
> Do not remove the legacy VPN path until the ZTNA path has been fully validated and a rollback plan is available.

---

## ☁️ Part B: Configure SCCM Cloud Management Gateway (CMG)

### Step 2: Configure SCCM Cloud Management Gateway (CMG)

1. In the SCCM console, create a **Cloud Management Gateway (CMG)**.

   Use an **Azure App Service or container-based** deployment so that SCCM clients can check in without being on the corporate network.

2. Install and configure a **Cloud Management Gateway connector point** on-premises.

3. Add the CMG as a **Service Connection Point**.

The basic management flow is:

```text
Remote SCCM Client
        ↓
Cloud Management Gateway
        ↓
CMG Connector Point
        ↓
SCCM / MECM
```

---

## 🛡️ Part C: Tunnel CMG Traffic Through ZTNA

### Step 3: Tunnel CMG Traffic Through ZTNA

1. Publish the **CMG endpoint** through the ZTNA gateway rather than exposing it directly.

2. Apply device posture checks consistent with your other **Zero Trust policies**.

3. Update client **Boundary Groups** so remote clients prefer the **CMG** for content and management traffic.

The intended architecture is:

```text
Remote Client
     ↓
Device Posture Check
     ↓
ZTNA Gateway
     ↓
CMG
     ↓
SCCM / MECM
```

> **📝 Note**
>
> The CMG should use the same Zero Trust approach as the rest of the environment, with access controlled by identity and device posture rather than relying solely on network location.

---

## 🧪 Part D: Validate Remote Patch Compliance

### Step 4: Validate Remote Patch Compliance

1. From a remote/off-network test device behind ZTNA, confirm that the **SCCM client checks in**.

2. Confirm that the client reports its **compliance state**.

3. Re-run the **ADR/Software Update Group** configured in Lab 3.

4. Confirm that the remote device receives and installs patches through the:

   ```text
   CMG / ZTNA path
   ```

The expected workflow is:

```text
Remote Device
     ↓
ZTNA
     ↓
CMG
     ↓
SCCM
     ↓
ADR / Software Update Group
     ↓
Required Patches
     ↓
Client Compliance
```

> **✅ Validation Result**
>
> The remote test device should successfully check in to SCCM, receive the required patches, install them, and report its updated compliance state without requiring a traditional corporate-network connection.

---

## 📊 Part E: Set Up Centralized Logging and SIEM Integration

### Step 5: Set Up Centralized Logging and SIEM Integration

Forward the following logs to your chosen **SIEM or log destination**:

* **ZTNA access logs**
* **SCCM client health/compliance logs**
* **VDI session logs**

Create a basic correlation rule.

**Example:**

```text
Alert on repeated ZTNA access denials from the same user
```

The centralized monitoring flow is:

```text
ZTNA Logs ───────────────┐
                         │
SCCM Logs ───────────────┼──→ SIEM / Log Destination
                         │
VDI Session Logs ────────┘
```

---

## 📈 Part F: Build a Cross-Platform Dashboard

### Step 6: Build a Cross-Platform Dashboard

Build a single dashboard using a platform such as:

* **Sentinel workbook**
* **Grafana**
* **Similar dashboard platform**

The dashboard should show:

| Metric                               | Purpose                                       |
| ------------------------------------ | --------------------------------------------- |
| **ZTNA access success/failure rate** | Monitor successful and denied access attempts |
| **VDI session host utilization**     | Monitor VDI host resource utilization         |
| **SCCM patch compliance percentage** | Monitor endpoint patch compliance             |

Walk through the complete flow end-to-end and confirm that every layer is visible in one place:

```text
Identity
   ↓
ZTNA
   ↓
VDI / CMG
   ↓
SCCM Compliance
   ↓
Dashboard
```

The final environment should provide visibility across the complete access and management path.

---

## 💡 Best Practice Tips

* **Decommission the legacy VPN path only after the ZTNA path is fully validated.** Keep a rollback plan available.
* **Treat the CMG like any internet-facing service.** Keep it patched and monitor its App Service logs.
* **Correlate identity, ZTNA, and SCCM logs by username/device ID** so incidents can be traced end-to-end.
* **Re-run this capstone checklist periodically.** Zero Trust is a continuous posture, not a one-time setup.
