<div align="center">

# 🚨 DirtyFrag Awareness

### Linux Kernel Local Privilege Escalation (LPE) Research & Awareness

<p align="center">
  <img src="https://img.shields.io/badge/Linux-Kernel-critical?style=for-the-badge&logo=linux&logoColor=white" />
  <img src="https://img.shields.io/badge/Security-Research-red?style=for-the-badge&logo=hackthebox&logoColor=white" />
  <img src="https://img.shields.io/badge/Status-Educational-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Focus-Defensive-success?style=for-the-badge" />
</p>

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:0f172a,100:dc2626&height=240&section=header&text=DirtyFrag&fontSize=58&fontColor=ffffff&animation=fadeIn&fontAlignY=38&desc=Linux%20Kernel%20LPE%20Awareness%20Project&descAlignY=60" />
</p>

</div>

---

# 📖 Overview

**DirtyFrag** is a Linux kernel Local Privilege Escalation (LPE) research and awareness project focused on page-cache corruption and modern kernel attack surfaces.

This repository aims to:

- document the internal mechanics behind the vulnerability,
- explain the exploitation chain,
- analyze impacted Linux subsystems,
- provide mitigation and defensive guidance,
- help defenders and researchers understand kernel attack vectors.

> ⚠️ This repository is strictly educational and defensive.  
> No weaponized exploit or offensive tooling is provided.

---

# 🔍 Technical Summary

DirtyFrag targets several low-level Linux kernel components:

- memory management,
- page-cache handling,
- networking internals,
- packet fragmentation mechanisms,
- `sk_buff` structures.

The vulnerability family is conceptually related to:

- Dirty Pipe (2022),
- Copy Fail (2024),
- page-cache corruption primitives.

---

# 🧠 Key Characteristics

| Feature | Description |
|---|---|
| Deterministic behavior | No race condition required |
| Stable execution model | Reduced crash probability |
| Page-cache corruption | Arbitrary controlled writes |
| Namespace interaction | User namespace implications |
| Multi-distribution impact | Ubuntu / RHEL / Fedora / SUSE |
| Defensive bypasses | Certain mitigations can be bypassed |

---

# 🧩 Exploitation Chain

## CVE-2026-43284 — xfrm-ESP Page-Cache Write

| Property | Value |
|---|---|
| Component | IPsec / XFRM |
| Primitive | Controlled 4-byte page-cache write |
| Introduced | `cac2661c53f3` |
| Fixed | `f4c50a4034e6` |
| Requirement | User namespace |

---

## CVE-2026-43500 — RxRPC Page-Cache Write

| Property | Value |
|---|---|
| Component | `rxrpc.ko` |
| Primitive | Arbitrary page-cache write |
| Introduced | `2dc334f1a63a` |
| Fixed | `aa54b1d27fe0` |
| Requirement | None |

---

# 🎯 Cross-Distribution Targeting Logic

```text
Ubuntu
 └── RxRPC exploitation path

RHEL / Enterprise Linux
 └── xfrm + namespace exploitation path
```

Result:

- broader kernel attack surface coverage,
- privilege escalation scenarios across multiple Linux ecosystems.

---

# 🖥️ Potentially Impacted Environments

- Ubuntu 24.04 LTS
- Fedora 44
- RHEL 10.x
- AlmaLinux 10
- CentOS Stream 10
- openSUSE Tumbleweed

---

# 🛡️ Immediate Mitigation

## Disable vulnerable modules

```bash
sudo sh -c "printf 'install esp4 /bin/false\ninstall esp6 /bin/false\ninstall rxrpc /bin/false\n' \
 > /etc/modprobe.d/dirtyfrag.conf; \
 rmmod esp4 esp6 rxrpc 2>/dev/null; \
 sync && echo 3 > /proc/sys/vm/drop_caches; true"
```

---

# 🧹 Flush Page Cache

```bash
sudo sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

---

# ✅ Permanent Remediation

Apply official kernel security updates provided by your Linux distribution vendor.

Recommended:

- update kernel packages,
- reboot affected systems,
- validate mitigations,
- review namespace policies,
- monitor unusual page-cache behavior.

---

# 📅 Timeline

| Date | Event |
|---|---|
| Jan 2017 | xfrm vulnerability introduced |
| Jun 2023 | RxRPC issue introduced |
| May 2026 | Official fixes integrated |

---

# 📚 Research Goals

This repository focuses on:

- Linux kernel internals,
- defensive security research,
- vulnerability awareness,
- mitigation engineering,
- page-cache exploitation theory,
- Linux hardening practices.

---

# ⚠️ Legal & Ethical Notice

This project is intended exclusively for:

- defensive research,
- educational purposes,
- authorized laboratory environments,
- cybersecurity awareness.

Do **NOT** use these concepts against systems without explicit authorization.

---

# 👤 Credits

## Original Research

- **Hyunwoo Kim (@v4bel)**

## French Awareness Project

- **@Maxime288**

---

<div align="center">

## ⭐ Support the Project

If this repository helped your research or awareness work:

⭐ Star the repository  
🔁 Share with the security community  
🛡️ Promote defensive security research

---

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:dc2626,100:0f172a&height=120&section=footer"/>

</div>
