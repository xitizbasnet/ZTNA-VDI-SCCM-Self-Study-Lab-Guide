# LAB 2: Virtual Desktop Infrastructure (VDI)

## 📘 Overview

**Goal:** Stand up a working **Virtual Desktop Infrastructure (VDI)** pool with a golden image, roaming profiles, and tuned performance.

**Example platform:** Azure Virtual Desktop (AVD)

This lab covers the core components required to build a VDI environment, including the golden image, host pool, session hosts, profile management with FSLogix, and performance optimization.

---

## 📋 Prerequisites

| Prerequisite           | Details                                                                          |
| ---------------------- | -------------------------------------------------------------------------------- |
| **Cloud subscription** | Azure subscription (or Citrix/VMware Horizon environment if using those instead) |
| **Networking**         | A VNet/subnet the session hosts and storage account can reach                    |
| **Storage**            | Azure Files or a file server for FSLogix profile containers                      |
| **Licensing**          | Windows 10/11 Enterprise multi-session + M365/Windows license for the test users |

---

## 🏗️ Part A: Choose the VDI Architecture

### Step 1: Choose the VDI Architecture

Determine which VDI architecture you will use.

* Decide between **hosted VDI** and **on-premises VDI**.

  * **Hosted:** Cloud-managed, such as AVD or Horizon Cloud.
  * **On-premises:** VDI infrastructure hosted within your organization's environment.

Identify the core components that you will build:

| Component             | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| **Session hosts**     | VMs that users connect to for their virtual desktop sessions |
| **Connection broker** | Routes users to an appropriate session host                  |
| **Gateway**           | Secures external connections to the VDI environment          |

---

## 🖼️ Part B: Build and Optimize the Golden Image

### Step 2: Build and Optimize the Golden Image

1. Deploy a base **Windows 10/11 multi-session VM**.

2. Install the required:

   * Line-of-business applications
   * Windows updates

3. Run the **Azure Virtual Desktop optimization script**, or an equivalent optimization tool.

   The optimization process disables unnecessary:

   * Services
   * Scheduled tasks
   * Visual effects

   This helps reduce resource consumption in a multi-user VM.

4. Generalize the VM using **Sysprep**.

5. Capture the VM as either:

   * A managed image
   * A Shared Image Gallery version

> **💡 Tip**
>
> Keep the golden image focused on the applications and configuration required by the users. Every additional component in the image can affect the resources consumed by future session hosts.

---

## 🖥️ Part C: Create the Host Pool and Session Hosts

### Step 3: Create the Host Pool and Session Hosts

1. Create a host pool.

2. Select the appropriate host pool configuration:

   * **Pooled** or **Personal**
   * **Breadth-first** or **Depth-first** load balancing

3. Deploy session host VMs from the golden image into the host pool.

4. Join the session hosts to your:

   * Domain
   * Entra ID

5. Create an **Application Group**.

   Choose either:

   * **Desktop**
   * **RemoteApp**

6. Create a **Workspace**.

7. Assign your test user group to the Application Group.

The resulting logical structure is:

```text
Workspace
    │
    └── Application Group
            │
            └── Session Hosts
                    │
                    └── Golden Image
```

---

## 👤 Part D: Configure Profile Management with FSLogix

### Step 4: Configure Profile Management (FSLogix)

Configure FSLogix so that user profiles can follow users between session hosts.

1. Provision an **Azure Files** share or a file server for FSLogix profile containers.

2. Configure the correct:

   * NTFS permissions
   * Share permissions

   for the user group.

3. Install **FSLogix** on the golden image.

   Alternatively, install it using a startup script.

4. Configure the required registry keys:

   ```text
   VHDLocations
   Enabled=1
   ```

5. Configure `VHDLocations` to point to the profile share.

6. Re-capture or update the golden image after making this change.

7. Redeploy the session hosts.

> **📝 Note**
>
> FSLogix profile containers allow a user's profile to follow them between session hosts. This is essential for maintaining user settings and files in a pooled VDI environment.

---

## ⚡ Part E: Tune Performance

### Step 5: Tune Performance

Configure the appropriate display protocol for the selected VDI platform.

| Platform                        | Display Protocol |
| ------------------------------- | ---------------- |
| **Azure Virtual Desktop (AVD)** | RDP Shortpath    |
| **Citrix**                      | HDX              |
| **VMware Horizon**              | Blast Extreme    |

1. Select the appropriate display protocol for your platform.

2. Enable the protocol on the gateway/host pool.

3. Right-size the VM SKUs based on expected concurrent user density.

4. Consider the following when selecting VM resources:

   * vCPU per session
   * RAM per session

5. Enable GPU acceleration only for graphics-heavy workloads.

> **💡 Tip**
>
> Avoid selecting VM sizes based only on individual-user requirements. VDI capacity must account for the expected number of concurrent sessions running on each session host.

---

## 🧪 Part F: Test and Validate

### Step 6: Test and Validate

1. Log in as a test user from the **Remote Desktop / AVD client**.

2. Confirm that the virtual desktop loads successfully.

3. Verify that the correct user profile is loaded.

4. Log out.

5. Log back in, or connect from a second host.

6. Confirm that the **FSLogix profile follows the user**.

7. Verify that the user's:

   * Settings
   * Files

   remain intact.

This confirms that profile roaming is working correctly.

> **✅ Validation Result**
>
> Successful profile roaming is demonstrated when the user can move between session hosts while retaining their settings and files.

---

## 💡 Best Practice Tips

* **Keep the golden image minimal.** Install only what is needed. Bloated images slow down every future host.
* **Always version your golden image.** Use image gallery versions so you can roll back a bad update.
* **Separate the FSLogix profile share from application data shares** to reduce I/O contention.
* **Load-test session host capacity before rollout.** Undersized hosts are the **#1 cause of VDI complaints**.
