# Azure Storage Lab
> **Disclaimer:** All Azure resources created in this lab were deleted after completion. No services were left running. This lab was built purely for learning and documentation purposes.

---

<h2>Description</h2>
This lab focuses on how I worked with Azure Storage — covering major storage types, access control methods, data protection features, and lifecycle management.
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

##  What I covered in this Lab 

- Created storage accounts with LRS, GRS, and ZRS redundancy zone's
- I Work with Blob containers and set Hot, Cool, and Archive access tiers
- Configured lifecycle management policies to automate tier transitions
- Created an Azure File Share and provision VMs
- I Generated Shared Access Signatures (SAS tokens) and stored access policies
- Enabled soft delete and versioning
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

I created a resource group named `Az-ST-lab` and set the region to **Australia East**.

**Steps:**
1. Navigate to **Resource groups** → **+ Create**
2. Name: `Az-ST-lab` | Region: `Australia East`
3. Click **Review + create** → **Create**

<img src="https://imgur.com/0aFIcEC.png" height="80%" width="80%" alt="Resource Group"/>

---

## Part 2 — Storage Accounts (LRS, GRS, ZRS)

I then created three storage accounts to demonstrate the different redundancy options available in Azure,
one was made on the portal, and two were created in the CLI

| Storage Account | Redundancy | Method |
|---|---|---|
| `nako8klrs` | Locally-redundant storage (LRS) | Portal |
| `nako8kgrs` | Geo-redundant storage (GRS) | Azure CLI |
| `nako8kzrs` | Zone-redundant storage (ZRS) | Azure CLI |

**CLI Commands used:**
```bash
az storage account create \--name nako8kgrs \--resource-group Az-ST-lab \--location australiaeast \--sku Standard_GRS

az storage account create \--name nako8kzrs \--resource-group Az-ST-lab \--location australiaeast \--sku Standard_ZRS
```

<img src="https://imgur.com/UgiaxZs.png" height="80%" width="80%" alt="ST ACCs"/>

<img src="https://imgur.com/HZ2FB85.png" height="80%" width="80%" alt="ST ACCs"/>

<img src="https://imgur.com/Wai1pZp.png" height="80%" width="80%" alt="ST ACCs"/>

<img src="https://imgur.com/HyIiPXL.png" height="80%" width="80%" alt="ST ACCs"/>

---

## Part 3 — Blob Containers, Access Tiers & Lifecycle Management

I had three blob containers created inside `nako8klrs` which I would use as my main storage account for this lab, 
each assigned a different access tier.

| Container | Access Tier | Use Case |
|---|---|---|
| `blob-hot` | Hot | Frequently accessed data |
| `blob-cool` | Cool | occasionally accessed, stored 30+ days |
| `blob-archive` | Archive | Barely accessed, stored 180+ days |

<img src="https://imgur.com/TMOVGqO.png" height="80%" width="80%" alt="Blobs"/>

**Lifecycle Management Policy**

A policy was configured on `nako8klrs` to automate tier transitions:

| Condition | Action |
|---|---|
| 30 days since last modification | Move to Cool |
| 90 days since last modification | Move to Archive |
| 365 days since last modification | Delete blob |

This removes the need for manual tier management in production environments.

<img src="https://imgur.com/k9vAnU2.png" height="80%" width="80%" alt="Blobs"/>

---

## Part 4 — Azure File Share & VM Provisioning

### File Share

Created `st-fileshare` inside `nako8klrs`.

<img src="https://imgur.com/1uPmcVK.png" height="80%" width="80%" alt="FS"/>

### Virtual Machines

| VM | Os | Purpose |
|---|---|---|
| `Mihini1` | Windows | Mount file share via PowerShell |
| `Mihini2` | Windows | Secondary VM |

> **Note:** "Mihini" = "machine" in Māori.

I had `Mihini1` provisioned in `Mihini_group` and `Mihini2` provisioned in `Az-ST-lab` in **Australia East**.

<img src="https://imgur.com/mZBWKtM.png" height="80%" width="80%" alt="FS"/>

### File Share Mounting

The mount script was retrieved via the Azure Portal where I had generated a PowerShell script

> **Important Note:** I did not perform a RDP connection to the VMs due to security restrictions for my environment. The mount scripts were retrieved and documented from the Portal . However in a real environment, the PowerShell script would have been executed.

---

## Part 5 — SAS Tokens & Stored Access Policies

### Shared Access Signature (SAS) Token

A SAS token was generated on a blob inside `blob-hot` to demonstrate the time-limited access without sharing storage account keys.

**Settings used:**
- **Permissions:** Read only
- **Expiry:** 24 hours
- **Protocol:** HTTPS only

The generated SAS URL was tested in a browser — the blob loaded successfully, confirming the token worked correctly.

<img src="https://imgur.com/JdgIuEK.png" height="80%" width="80%" alt="SAS"/>

Blob test output

<img src="https://imgur.com/Xr6KK1D.png" height="80%" width="80%" alt="SAS"/>

### Stored Access Policy — `blob-hot-policy`

A stored access policy was created on the `blob-hot` container to provide revokable SAS token management.

| Setting | Value |
|---|---|
| **Identifier** | `blob-hot-policy` |
| **Permissions** | Read, Write, List |
| **Expiry** | 30 days |

<img src="https://imgur.com/oEnXTil.png" height="80%" width="80%" alt="SAS"/>

---

## Part 6 — Soft Delete, Versioning & Blob Recovery

### Configuration

Data protection settings were enabled on `nako8klrs`:

| Feature | Setting |
|---|---|
| Blob versioning | Enabled |
| Blob soft delete | Enabled — 7 day retention |
| Container soft delete | Enabled — 7 day retention |

<img src="https://imgur.com/6pUoTIm.png" height="80%" width="80%" alt="Soft del"/>

### Simulated Deletion & Recovery

1. A blob inside `blob-hot` was deleted via the Portal
2. **Show deleted blobs** toggle revealed the blob in a soft-deleted state
3. The **Undelete** option restored the blob successfully

This demonstrates that even after deletion, blobs remain recoverable within the retention window.

<img src="https://imgur.com/X3BXsen.png" height="80%" width="80%" alt="Soft del"/>

<img src="https://imgur.com/ztgYxYB.png" height="80%" width="80%" alt="Soft del"/>

---

## Key Takeaways

- Azure offers three redundancy tiers — LRS, GRS, and ZRS 
- Blob access tiers (Hot, Cool, Archive) — When using Lifecyle polices, they can automaticly assinge which tier you files are in
- SAS tokens are time-limited access to storage without exposing account keys — which creates a level of security 
- Soft delete and versioning are data protection features — they protect against deletion within a configurable retention windows, I understood this when attemting to delete my lab
- Azure sometimes creates background resources (like Recovery Services Vaults) automatically — understanding this and noting it will help with cost management and awearness within my tenant
- Always delete VMs in a stopped/deallocated state and verify all dependent resources are removed, I do this in labs to prevent acumilating cost, as housing vm's may lead to charges 

---
<h2>Lessons Learned</h2>

- <b>Awearness is key</b> — When working with Resource Groups and Storage Containers, it is important to be aware of the services and systems running within them.
I learned this while attempting to remove all the resources in my lab. A simple Azure Key Vault was preventing me from deleting the Resource Group (RG). Although it was an unintentional accident, learning how to identify and resolve the issue turned out to be a valuable lesson that I will apply to future projects.
<br /><br />
- <b>CLI or Portal?</b> — In this lab, working in both the Azure Portal and the CLI helped broaden my experience and perspective. In a previous lab, I learned that the CLI can provide a much quicker method of configuration. However, I was unable to properly configure my SAS tokens through the CLI, which ultimately slowed my progress down. This may have been due to human error on my part or a software-related issue, but it highlighted an important lesson: while the CLI is often faster and more efficient, there are situations where the Azure Portal can be the better option.

---

*Lab completed by [@Nako8k](https://github.com/Nako8k)*
