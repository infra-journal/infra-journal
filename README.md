# Infrastructure Journal

This repository contains my personal notes from working on
Linux and Oracle-based systems.

Entries are written as postmortems:
what broke, what was checked, and what fixed it.

It also includes notes on setting up, installing,
and configuring software and infrastructure components.

---

## File Naming Convention

**Format**

```
YYYY-MM-DD__area__short-description.md
```

**Example**

```
2026-02-14__ords__service-fails-after-windows-patch.md
```

This format ensures:
- chronological ordering
- easy scanning by area
- long-term maintainability

---

## Folder Structure

```
infra-journal/
└── 2026/
    ├── 2026-02-14__ords__service-fails-after-windows-patch.md
    ├── 2026-02-20__linux__high-io-wait.md
    └── 2026-03-01__weblogic__ssl-cert-expiry.md
```

Each year contains its own entries and an `INDEX.md`
for quick navigation.
