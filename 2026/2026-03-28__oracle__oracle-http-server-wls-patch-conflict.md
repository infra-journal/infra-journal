# OPatch Conflict: WLS PSU 38792523 vs FMW Thirdparty Bundle Patch

## Applies To

- **Product:** Oracle WebLogic Server 12.2.1.4.0
- **Component:** OPatch / Oracle Home Patching
- **Platform:** All platforms (Windows / Linux)

---

## Problem Description

While applying the **WLS Patch Set Update 12.2.1.4.251223 (Patch 38792523)**, OPatch fails with a conflict against the previously installed **FMW Thirdparty Bundle Patch 12.2.1.4.250314 (Patch 37710654)**.

### Error Message

```
Following patches have conflicts: [   37710654   38792523 ]
Use the MOS Patch Conflict Checker "https://support.oracle.com/epmos/faces/PatchConflictCheck" to resolve.
See MOS documents 1941934.1 and 1299688.1 for additional information and resolution methods.
UtilSession failed: Inter-conflict checking failed in apply incoming patches
Log file location: <ORACLE_HOME>\cfgtoollogs\opatch\opatch2026-03-27_05-00-57AM_1.log
OPatch failed with error code 73
```

### OPatch Log Detail

```
[INFO]    Details of Conflicting Patches:

          Installed Patches:
          37710654;FMW Thirdparty Bundle Patch 12.2.1.4.250314

          Incoming Patches:
          38792523;WLS PATCH SET UPDATE 12.2.1.4.251223

[INFO]    Conflicts/Supersets for each patch are:
          Patch : 38792523
            Conflict with 37710654
            Conflict details:
            <Oracle_Home>/oracle_common/modules/thirdparty/log4j-2.11.1.jar
            Bug Superset of 38412437
            Super set bugs are: ...

[SEVERE]  OUI-67073:UtilSession failed: Inter-conflict checking failed in apply incoming patches
```

---

## Root Cause

The conflict occurs on `log4j-2.11.1.jar`. Both the incoming WLS PSU (38792523) and the installed FMW Thirdparty Bundle Patch (37710654) modify this same file.

Applying them separately — in any order — triggers an OPatch inter-conflict error because OPatch detects a file-level collision.

> **Known Bug:** This is a documented issue tracked under unpublished Oracle Bug **38871681**:  
> *"WEBLOGIC PSU PATCH 38792523 CONFLICTS WITH PREVIOUS TPP PATCH"*

---

## What Does NOT Work

| Approach | Outcome |
|---|---|
| Apply WLS PSU 38792523 alone | ❌ Conflicts with installed Thirdparty Bundle |
| Roll back Thirdparty Bundle, then apply PSU | ❌ Causes conflicts with older installed patches |
| Apply newer Thirdparty Bundle 38759265 first | ❌ Conflicts with existing WLS PSU patch 38412437 |

---

## Solution

Apply the **WLS PSU** and the **latest FMW Thirdparty Bundle Patch** together in a **single `opatch napply` command**.

### Patches Required

| Patch Number | Description |
|---|---|
| **38792523** | WLS Patch Set Update 12.2.1.4.251223 (Dec 2025) |
| **38759265** | FMW Thirdparty Bundle Patch (latest, compatible with above PSU) |

### Step-by-Step Instructions

**Step 1 — Backup your inventory**

```bat
opatch lsinventory -detail > <ORACLE_HOME>\inventory_backup_%date%.txt
```

**Step 2 — Download both patches from MOS**

- Patch **38792523** — WLS PSU 12.2.1.4.251223
- Patch **38759265** — FMW Thirdparty Bundle Patch (latest)

**Step 3 — Extract both patches into the same directory**

```
W:\patches\
    ├── 38759265\
    └── 38792523\
```

**Step 4 — Navigate to the patches directory**

```bat
cd D:\patches
```

**Step 5 — Apply both patches together using napply**

```bat
opatch napply -id 38759265,38792523
```

> ⚠️ **Critical:** The comma between patch IDs is **mandatory** with **no spaces**.  
> Applying the patches separately will reproduce the same conflict error.

---

## Why This Works

When both patches are passed to `opatch napply` in a single command, OPatch resolves the `log4j-2.11.1.jar` file conflict internally by processing them as a combined transaction. This avoids the collision that occurs when OPatch compares an already-installed patch against an incoming one independently.

---

## References

| Reference | Description |
|---|---|
| MOS Note **1941934.1** | OPatch Conflict Resolution — General Guidance |
| MOS Note **1299688.1** | How to Request a Merged Patch from Oracle Support |
| Oracle Bug **38871681** | Unpublished bug: WLS PSU 38792523 conflicts with previous TPP patch |
| [MOS Patch Conflict Checker](https://support.oracle.com/epmos/faces/PatchConflictCheck) | Oracle patch conflict validation tool |

---

## Environment

| Item | Value |
|---|---|
| Product Version | WebLogic 12.2.1.4.0 |
| Installed Patch | 37710654 — FMW Thirdparty Bundle 12.2.1.4.250314 |
| Incoming Patch | 38792523 — WLS PSU 12.2.1.4.251223 |
| Resolution Patch | 38759265 — FMW Thirdparty Bundle (latest) |
| OPatch Error Code | 73 |
| Conflicting File | `oracle_common/modules/thirdparty/log4j-2.11.1.jar` |
