# Azure Storage Lab
> **Disclaimer:** All Azure resources created in this lab were deleted immediately after completion. No services were left running. This lab was built purely for learning and documentation purposes.

---

<h2>Description</h2>
This lab focuses on Azure Storage — covering all major storage types, access control methods, data protection features, and lifecycle management.
<br /><br />

| Detail | Value |
|---|---|
| **Resource Group** | `Az-ST-lab` |
| **Region** | Australia East |
| **Storage Accounts** | `nako8klrs` / `nako8kgrs` / `nako8kzrs` |
| **Blob Containers** | `blob-hot` / `blob-cool` / `blob-archive` |
| **File Share** | `st-fileshare` |
| **Virtual Machines** | `Mihini1` (Windows) / `Mihini2` (Windows) |

---

##  What This Lab Covers

- Create storage accounts with LRS, GRS, and ZRS redundancy
- Work with Blob containers — set Hot, Cool, and Archive access tiers
- Configure lifecycle management policies to automate tier transitions
- Create an Azure File Share and provision VMs to mount it
- Generate Shared Access Signatures (SAS tokens) and stored access policies
- Enable soft delete and versioning, then simulate and recover a deleted blob

---

## Lab Structure

```
azure-storage-lab/
├── README.md
├── commands-reference.md
└── screenshots/
    ├── part1-resource-group/
    ├── part2-storage-accounts/
    ├── part3-blob-containers/
    ├── part4-file-shares/
    ├── part5-sas-tokens/
    └── part6-soft-delete/
```

---

## Part 1 — Resource Group

Created the resource group `Az-ST-lab` in **Australia East** to house all lab resources.

**Steps:**
1. Navigate to **Resource groups** → **+ Create**
2. Name: `Az-ST-lab` | Region: `Australia East`
3. Click **Review + create** → **Create**

*Screenshot: `screenshots/part1-resource-group/`*

---

## Part 2 — Storage Accounts (LRS, GRS, ZRS)

Three storage accounts were created to demonstrate the different redundancy options available in Azure.

| Storage Account | Redundancy | Method |
|---|---|---|
| `nako8klrs` | Locally-redundant storage (LRS) | Portal |
| `nako8kgrs` | Geo-redundant storage (GRS) | Azure CLI |
| `nako8kzrs` | Zone-redundant storage (ZRS) | Azure CLI |

**Redundancy Explained:**

- **LRS** — 3 copies within a single datacenter. Lowest cost. Suitable for dev/test.
- **GRS** — Replicates to a secondary region hundreds of miles away. Protects against regional outages.
- **ZRS** — Replicates across 3 availability zones in the same region. Protects against zone-level failures.

**CLI Commands used:**
```bash
az storage account create \
  --name nako8kgrs \
  --resource-group Az-ST-lab \
  --location australiaeast \
  --sku Standard_GRS

az storage account create \
  --name nako8kzrs \
  --resource-group Az-ST-lab \
  --location australiaeast \
  --sku Standard_ZRS
```

📸 *Screenshot: `screenshots/part2-storage-accounts/`*

---

## Part 3 — Blob Containers, Access Tiers & Lifecycle Management

Three blob containers were created inside `nako8klrs`, each assigned a different access tier.

| Container | Access Tier | Use Case |
|---|---|---|
| `blob-hot` | Hot | Frequently accessed data |
| `blob-cool` | Cool | Infrequently accessed, stored 30+ days |
| `blob-archive` | Archive | Rarely accessed, stored 180+ days |

**Lifecycle Management Policy — `tier-transition-rule`:**

A policy was configured on `nako8klrs` to automate tier transitions:

| Condition | Action |
|---|---|
| 30 days since last modification | Move to Cool |
| 90 days since last modification | Move to Archive |
| 365 days since last modification | Delete blob |

This removes the need for manual tier management in production environments.

📸 *Screenshot: `screenshots/part3-blob-containers/`*

---

## Part 4 — Azure File Share & VM Provisioning

### File Share

Created `st-fileshare` inside `nako8klrs` with the **Transaction optimized** tier.

### Virtual Machines

| VM | OS | Purpose |
|---|---|---|
| `Mihini1` | Windows Server 2022 | Mount file share via PowerShell |
| `Mihini2` | Windows Server 2022 | Secondary VM |

> **Note:** "Mihini" means "machine" in te reo Māori.

Both VMs were provisioned in `Az-ST-lab` in **Australia East** using `Standard_B1s` sizing.

### File Share Mounting

The mount script was retrieved via the Azure Portal:
- Navigate to `nako8klrs` → **File shares** → `st-fileshare` → **Connect**
- The **Windows** tab generates a PowerShell script that maps the share as a network drive (e.g. `Z:`)

> **Security Note:** RDP connection to the VMs was not performed due to security restrictions in this environment. The mount scripts were retrieved and documented from the Portal Connect blade. In a production environment, the generated PowerShell script would be executed inside the VM to map the file share as a persistent network drive.

📸 *Screenshot: `screenshots/part4-file-shares/`*

---

## Part 5 — SAS Tokens & Stored Access Policies

### Shared Access Signature (SAS) Token

A SAS token was generated on a blob inside `blob-hot` to demonstrate scoped, time-limited access without sharing storage account keys.

**Settings used:**
- **Permissions:** Read only
- **Expiry:** 24 hours
- **Protocol:** HTTPS only

The generated SAS URL was tested in a browser — the blob loaded successfully, confirming the token worked correctly.

### Stored Access Policy — `blob-hot-policy`

A stored access policy was created on the `blob-hot` container to provide revokable SAS token management.

| Setting | Value |
|---|---|
| **Identifier** | `blob-hot-policy` |
| **Permissions** | Read, Write, List |
| **Expiry** | 30 days |

**Key advantage:** A stored access policy can be modified or deleted at any time, instantly revoking all SAS tokens tied to it — without needing to rotate the storage account key.

📸 *Screenshot: `screenshots/part5-sas-tokens/`*

---

## Part 6 — Soft Delete, Versioning & Blob Recovery

### Configuration

Data protection settings were enabled on `nako8klrs`:

| Feature | Setting |
|---|---|
| Blob versioning | Enabled |
| Blob soft delete | Enabled — 7 day retention |
| Container soft delete | Enabled — 7 day retention |

### Simulated Deletion & Recovery

1. A blob inside `blob-hot` was deleted via the Portal
2. **Show deleted blobs** toggle revealed the blob in a soft-deleted state
3. The **Undelete** option restored the blob successfully
4. **Version history** confirmed multiple versions were retained with timestamps
5. A previous version was restored using **Make current version**

This demonstrates that even after accidental or malicious deletion, blobs remain recoverable within the retention window.

📸 *Screenshot: `screenshots/part6-soft-delete/`*

---

## Decommissioning & Cleanup

> **Important Learning:** Decommissioning Azure resources cleanly is more complex than it appears and is rarely covered in standard lab guides. The following was encountered and resolved during teardown.

### Cleanup Order

Resources must be deleted in the correct order to avoid orphaned dependencies:

1. **Stop VMs** before deleting them to prevent orphaned resources
2. **Delete VMs** — ensure OS disks, network interfaces, and public IPs are checked for deletion
3. **Check for leftover VM resources** — disks, NICs, public IPs, and NSGs can survive VM deletion
4. **Delete storage accounts** — this removes all blobs, containers, file shares, and policies inside
5. **Delete the resource group** — acts as a final sweep for anything remaining

### Recovery Services Vault — Lesson Learned

**Issue:** After enabling soft delete on the Azure File Share, Azure automatically created a **Recovery Services Vault** inside `Az-ST-lab`. This vault blocked deletion of the resource group with the following error:

```
BMSUserErrorVaultDeletionNotAllowed: Recovery Services Vault cannot be deleted 
as there are existing resources within the vault.
```

**Root Cause:** The file share backup item existed in a soft-deleted state inside the vault, preventing vault deletion even though it appeared removed.

**Resolution — must be done in this exact order:**

1. Navigate to the vault → **Properties** → **Security Settings** → disable **Soft Delete**
2. Go to **Backup items** → **Azure Storage (Azure Files)** → toggle **Show deleted items**
3. The soft-deleted file share reappears — click **Undelete** to restore it to active state
4. Once active, click **Stop Backup** → **Delete Backup Data** → confirm
5. Go to **Backup Infrastructure** → **Storage Accounts** → unregister any listed accounts
6. Wait 2–3 minutes for Azure to process, then delete the vault
7. Delete the resource group as normal

> **Key insight:** Azure requires you to *undelete* a soft-deleted backup item before you can permanently remove it. This is counterintuitive but is by design — it prevents accidental permanent deletion of backup data.

> **Also note:** A resource lock (Delete lock) was found on `nako8klrs`. This must be removed first via **Storage account** → **Settings** → **Locks** → delete the lock, before the storage account can be deleted.

---

## Key Takeaways

- Azure offers three redundancy tiers — LRS, GRS, and ZRS — each suited to different availability and cost requirements
- Blob access tiers (Hot, Cool, Archive) directly impact storage costs and retrieval latency — lifecycle policies automate this in production
- SAS tokens provide scoped, time-limited access to storage without exposing account keys — stored access policies make them revokable
- Soft delete and versioning are critical data protection features — they protect against accidental and malicious deletion within a configurable retention window
- Azure sometimes creates background resources (like Recovery Services Vaults) automatically — understanding what triggers them is important for clean resource management and cost control
- Always delete VMs in a stopped/deallocated state and verify all dependent resources are removed

---
<h2>Lessons Learned</h2>

- <b>Lock and load</b> — When working with storage 
<br /><br />
- <b>Groups over individuals</b> —
<br /><br />
- <b>Least privilege in practice</b> — 
<br /><br />
- <b>key security features</b> —
<br /><br />
- <b>CLI vs Portal</b> — 
---

*Lab completed by [@Nako8k](https://github.com/Nako8k)*
