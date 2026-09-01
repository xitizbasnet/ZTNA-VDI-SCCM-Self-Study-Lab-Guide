# LAB 1: Zero Trust Network Access (ZTNA)

## 📘 Overview

**Goal:** Replace a legacy VPN-style access model with an identity-aware, per-application **Zero Trust Network Access (ZTNA)** flow.

In this lab, you will implement a Zero Trust access model that verifies the user's identity and device posture before granting access to a specific internal application. Access is limited to the application and the ports or protocols that the application requires.

---

## 📋 Prerequisites

Before starting the lab, prepare the following components:

| Prerequisite            | Details                                                                                                |
| ----------------------- | ------------------------------------------------------------------------------------------------------ |
| **Identity provider**   | Microsoft Entra ID (Azure AD) tenant with Global Admin access                                          |
| **ZTNA broker/gateway** | A ZTNA product trial — for example, Zscaler Private Access, Cloudflare Access, or Entra Private Access |
| **Test application**    | One internal web app or RDP/SSH host to publish. A small VM works.                                     |
| **Test device**         | A laptop/VM to install the ZTNA client/connector on                                                    |

---

## 🔐 Part A: Review Zero Trust Foundations

### Step 1: Review Zero Trust Foundations

Compare the traditional perimeter/VPN access model with the Zero Trust model.

* **Traditional perimeter/VPN access:** Trusts the network.
* **Zero Trust:** Trusts nothing by default and verifies every request.

Identify the three core pillars that you will implement:

1. **Identity verification**
2. **Device posture**
3. **Least-privilege, per-application access**

> **💡 Tip**
>
> The goal of ZTNA is to move access decisions away from network location and toward verified identity, device state, and application-specific authorization.

---

## 🗺️ Part B: Map the ZTNA Architecture

### Step 2: Map the ZTNA Architecture

Identify the three logical components in your chosen ZTNA product:

| Component                            | Description                                                     |
| ------------------------------------ | --------------------------------------------------------------- |
| **Policy Decision Point (PDP)**      | Usually the cloud control plane that evaluates access policies. |
| **Policy Enforcement Point (PEP)**   | The gateway or connector that enforces the access decision.     |
| **Software-Defined Perimeter (SDP)** | Hides the application from the public internet.                 |

Sketch the expected traffic flow:

```text
Client
   ↓
Identity Check
   ↓
PDP Evaluates Policy
   ↓
PEP Allows / Denies
   ↓
App
```

The resulting access flow should be:

**Client → Identity check → PDP evaluates policy → PEP allows/denies → App**

---

## 🔑 Part C: Configure Identity and MFA

### Step 3: Configure Identity and MFA

Configure Microsoft Entra ID to provide identity verification and additional access controls.

1. In **Entra ID**, register the test application.

   Navigate to:

   ```text
   App registrations → New registration
   ```

2. Enable **Conditional Access** with **Multi-Factor Authentication (MFA)** required for that application.

3. Add a device compliance/posture check.

   For example, require the user to access the application from a **managed or compliant device** as a Conditional Access condition.

> **📝 Note**
>
> The identity and device posture checks form part of the Zero Trust decision before access to the application is granted.

---

## ⚙️ Part D: Deploy the ZTNA Connector/Gateway

### Step 4: Deploy the ZTNA Connector/Gateway

1. Install the lightweight connector on a VM in the same network segment as your test application.

   This connector replaces the need for an **inbound firewall rule/VPN listener**.

2. Register the connector with your ZTNA tenant/control plane.

3. Confirm that the connector status shows:

   ```text
   Online
   ```

> **📝 Note**
>
> The connector makes an **outbound-only connection** to the broker. No inbound ports need to be opened.

---

## 🛡️ Part E: Define Application-Specific Access Policies

### Step 5: Define Application-Specific Access Policies

1. Publish the internal application through the ZTNA console.

2. Point the published application to the deployed connector.

3. Create an access policy scoped to a specific user group.

4. Configure the policy to require:

   * **MFA**
   * **Compliant device**

5. Limit access to only the ports and protocols that the application requires.

   **Never use "allow all".**

> **⚠️ Warning**
>
> Do not grant access to an entire subnet when the application requires only specific ports or protocols. Follow the principle of least privilege and scope access to the minimum required resources.

---

## 🧪 Part F: Test End-User Access

### Step 6: Test End-User Access

1. Install the ZTNA client on the test device.

2. Log in with a test user.

3. Confirm that the application is reachable only after successful:

   * Identity verification
   * Device posture checks

4. Confirm that the application is **not reachable** under the following conditions:

   * The device is unmanaged.
   * The device is non-compliant.
   * The ZTNA client is not installed or connected.

The expected access model is:

```text
Identity + Device Posture + ZTNA Client
                    ↓
             Access Granted
```

If any required condition fails, access should be denied.

---

## 🔧 Part G: Troubleshoot Common Issues

### Step 7: Troubleshoot Common Issues

#### Connection Failures

Check the following:

* The connector's outbound connectivity to the broker.
* DNS resolution from the connector to the broker.

#### Policy Mismatches

Verify the following:

* The user is a member of the correct group.
* The application scope is correct.
* The port scope is correct.

#### Latency

Check the following:

* Confirm that the connector is deployed close to the application, preferably in the same region/segment.
* Check the nearest broker point-of-presence for the client.

---

## 💡 Best Practice Tips

* **Start with one non-critical application** before migrating production VPN users. This validates the policy logic safely.
* **Log every access decision** (`allow`/`deny`) from day one. You will need this for the **Lab 4 SIEM integration**.
* **Use groups, not individual users**, in access policies. This makes audits and offboarding far easier.
* **Always scope policies to the minimum port/protocol** the application needs. Never grant access to a full subnet.

---

## 📝 Notes

> **Note**
>
> The ZTNA product used in this lab may use different terminology or configuration screens for the PDP, PEP, SDP, connector, Conditional Access, and application publishing components. Map the logical components described in this lab to the equivalent components in your chosen ZTNA product.
