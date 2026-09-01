# LAB 3: SCCM / MECM (Endpoint Configuration Manager)

## 📘 Overview

**Goal:** Stand up a working **SCCM/MECM** site, deploy an application, patch clients, and build a basic **OS Deployment (OSD)** task sequence.

This lab covers the core capabilities of Microsoft Endpoint Configuration Manager, including site infrastructure, client deployment, application deployment, patch management, operating system deployment, and co-management with Microsoft Intune.

---

## 📋 Prerequisites

| Prerequisite  | Details                                                |
| ------------- | ------------------------------------------------------ |
| **Server OS** | Windows Server (2019/2022) for the site server         |
| **Database**  | SQL Server (Standard/Express for lab use)              |
| **Roles**     | IIS and WSUS role installed on the appropriate servers |
| **Domain**    | Active Directory domain with schema extended for SCCM  |

---

## 🏗️ Part A: Plan Site Infrastructure

### Step 1: Plan Site Infrastructure

1. Decide the site type for the lab.

   A **single Primary Site** is sufficient for this lab.

   A **Central Administration Site (CAS)** is only needed at multi-primary/enterprise scale.

2. Identify the Site System Roles required at minimum:

   * **Management Point**
   * **Distribution Point**
   * **Software Update Point**

The basic site architecture is:

```text
                    SCCM / MECM
                         │
              ┌──────────┼──────────┐
              │          │          │
        Management   Distribution  Software
           Point        Point       Update Point
              │          │          │
              └──────────┼──────────┘
                         │
                       Clients
```

---

## ⚙️ Part B: Install Prerequisites and the Site

### Step 2: Install Prerequisites and the Site

1. Install and configure **SQL Server**.

2. Install **IIS** with the required role services.

3. Use the **SCCM prerequisite checker** to identify any missing prerequisites.

4. Extend the **Active Directory schema**.

5. Create the **System Management** container.

6. Run the **SCCM setup wizard**.

7. Install the **Primary Site**.

> **📝 Note**
>
> The SCCM prerequisite checker will flag missing prerequisites. Resolve prerequisite issues before continuing with the site installation.

---

## 🌐 Part C: Configure Boundaries and Client Deployment

### Step 3: Configure Boundaries and Client Deployment

1. Create a **Boundary** that matches your test network.

   The boundary can be based on:

   * IP subnet
   * Active Directory site

2. Add the boundary to a **Boundary Group**.

3. Enable a **client push installation** method.

   Alternatively, use:

   * GPO
   * Manual installation

   for the lab.

4. Push the SCCM client to a test machine.

5. Confirm client health in the console.

   Navigate to the **Devices** node and verify that the client shows:

   ```text
   Active
   Client = Yes
   ```

> **⚠️ Warning**
>
> Misconfigured boundaries or boundary groups can cause clients to communicate with or download content from an incorrect Distribution Point.

---

## 📦 Part D: Package and Deploy an Application

### Step 4: Package and Deploy an Application

1. Create an **Application** in the SCCM/MECM console.

2. Choose an appropriate deployment type:

   * MSI
   * Script installer
   * App-V

3. Define the application's:

   * Detection method
   * Requirements

4. Distribute the application content to your **Distribution Point**.

5. Deploy the application to a test device collection.

6. Confirm that the installation succeeds in **Software Center**.

> **📝 Note**
>
> **Supersedence** lets a newer application deployment automatically replace an older version. Configure supersedence when you have two versions of the same application.

---

## 🔄 Part E: Configure Patch Management

### Step 5: Configure Patch Management

1. Confirm that the **Software Update Point** is synchronizing with:

   * WSUS
   * Microsoft Update

2. Create a **Software Update Group** for a set of patches.

3. Build an **Automatic Deployment Rule (ADR)**.

4. Configure the ADR to deploy monthly patches automatically to a **pilot collection**.

5. Set up an **Orchestration Group** if you need to sequence patch reboots across dependent servers.

The basic patch-management flow is:

```text
Microsoft Update
       ↓
      WSUS
       ↓
Software Update Point
       ↓
Software Update Group
       ↓
Automatic Deployment Rule
       ↓
Pilot Collection
       ↓
Clients
```

> **💡 Tip**
>
> Use a pilot collection when introducing automated monthly patch deployments. This allows you to validate patches before broader deployment.

---

## 💿 Part F: Build OS Deployment (OSD)

### Step 6: Build OS Deployment (OSD)

1. Import a **Windows boot image**.

   Alternatively, use the default **x64 boot image**.

2. Distribute the boot image to your **Distribution Point**.

3. Create a **Task Sequence**.

4. Choose an appropriate task sequence type:

   * **Install an existing image**
   * **Build and Capture**

5. Include **driver package** steps for your hardware model.

6. Deploy the task sequence to a:

   * Bare-metal target
   * PXE-less target using boot media/USB

7. Validate that the deployment successfully completes:

   * Image deployment
   * Domain join

The basic OSD workflow is:

```text
Boot Image
    ↓
Task Sequence
    ↓
Windows Image
    ↓
Drivers
    ↓
Configuration
    ↓
Domain Join
    ↓
Completed Client
```

---

## ☁️ Part G: Enable Co-Management with Intune

### Step 7: Enable Co-Management with Intune

1. In the SCCM/MECM console, enable **Co-management**.

2. Connect the SCCM/MECM tenant to **Microsoft Intune**.

3. Choose which workloads should shift to Intune.

   Available examples include:

   * Compliance policies
   * Windows Update
   * Endpoint Protection

4. Start by shifting **one workload** for the pilot group.

   For example:

   ```text
   Compliance policies
   ```

5. Monitor the pilot group before shifting additional workloads.

> **💡 Tip**
>
> Move workloads gradually instead of shifting all workloads at once. This makes it easier to identify and troubleshoot issues during the transition to Intune.

---

## 💡 Best Practice Tips

* **Always test ADRs and task sequences on a small pilot collection** before deploying broadly.
* **Keep application detection methods accurate.** A bad detection rule can cause endless application re-installs.
* **Document your boundary groups carefully.** Misconfigured boundaries are the top cause of clients downloading content from the wrong Distribution Point.
* **Move co-management workloads to Intune gradually, one at a time**, and monitor compliance before shifting the next workload.
