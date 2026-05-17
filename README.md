<div align="center">

# 🚨 DirtyFrag Awareness

### Analyse & Sensibilisation à une élévation locale de privilèges (LPE) du noyau Linux

<p align="center">
  <img src="https://img.shields.io/badge/Linux-Kernel-critical?style=for-the-badge&logo=linux&logoColor=white" />
  <img src="https://img.shields.io/badge/Cybersécurité-Recherche-red?style=for-the-badge&logo=hackthebox&logoColor=white" />
  <img src="https://img.shields.io/badge/Statut-Éducatif-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Orientation-Défensive-success?style=for-the-badge" />
</p>

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:0f172a,100:dc2626&height=240&section=header&text=DirtyFrag&fontSize=58&fontColor=ffffff&animation=fadeIn&fontAlignY=38&desc=Linux%20Kernel%20LPE%20Awareness%20Project&descAlignY=60" />
</p>

</div>

---

# 📖 Présentation

**DirtyFrag** est un projet de recherche et de sensibilisation autour d’une vulnérabilité de type **Local Privilege Escalation (LPE)** affectant le noyau Linux.

Ce dépôt a pour objectif de :

- documenter les mécanismes internes de la vulnérabilité,
- expliquer la chaîne d’exploitation,
- analyser les sous-systèmes Linux impactés,
- proposer des mesures de mitigation,
- aider les chercheurs et défenseurs à mieux comprendre les surfaces d’attaque du noyau Linux.

> ⚠️ Ce dépôt est strictement éducatif et défensif.  

# 🔍 Résumé Technique

DirtyFrag cible plusieurs composants bas niveau du noyau Linux :

- gestion mémoire,
- page-cache,
- sous-système réseau,
- fragmentation réseau,
- structures `sk_buff`.

La vulnérabilité est conceptuellement proche de :

- Dirty Pipe (2022),
- Copy Fail (2024),
- primitives modernes de corruption du page-cache.

---

# 🧠 Caractéristiques Principales

| Fonctionnalité | Description |
|---|---|
| Déterminisme logique | Aucune race condition requise |
| Stabilité élevée | Faible probabilité de crash système |
| Corruption du page-cache | Écritures contrôlées possibles |
| Interaction namespaces | Impact des user namespaces |
| Impact multi-distributions | Ubuntu / RHEL / Fedora / SUSE |
| Contournement défensif | Certaines mitigations peuvent être contournées |

---

# 🧩 Chaîne d’Exploitation

## CVE-2026-43284 — xfrm-ESP Page-Cache Write

| Élément | Valeur |
|---|---|
| Composant | IPsec / XFRM |
| Primitive | Écriture contrôlée de 4 octets |
| Introduction | `cac2661c53f3` |
| Correctif | `f4c50a4034e6` |
| Condition | User namespace |

---

## CVE-2026-43500 — RxRPC Page-Cache Write

| Élément | Valeur |
|---|---|
| Composant | `rxrpc.ko` |
| Primitive | Écriture arbitraire dans le page-cache |
| Introduction | `2dc334f1a63a` |
| Correctif | `aa54b1d27fe0` |
| Condition | Aucune |

---

# 🎯 Logique de Ciblage Croisé

```text
Ubuntu
 └── Exploitation via RxRPC

RHEL / Enterprise Linux
 └── Exploitation via xfrm + namespaces
```

Résultat :

- surface d’attaque élargie,
- élévation de privilèges possible sur plusieurs distributions Linux.

---

# 🖥️ Environnements Potentiellement Impactés

- Ubuntu 24.04 LTS
- Fedora 44
- RHEL 10.x
- AlmaLinux 10
- CentOS Stream 10
- openSUSE Tumbleweed

---

# 🛡️ Mitigation Immédiate

## Désactivation des modules vulnérables

```bash
sudo sh -c "printf 'install esp4 /bin/false\ninstall esp6 /bin/false\ninstall rxrpc /bin/false\n' \
 > /etc/modprobe.d/dirtyfrag.conf; \
 rmmod esp4 esp6 rxrpc 2>/dev/null; \
 sync && echo 3 > /proc/sys/vm/drop_caches; true"
```

---

# 🧹 Purge du Page-Cache

```bash
sudo sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

---

# ✅ Remédiation Définitive

Appliquez les correctifs de sécurité officiels fournis par votre distribution Linux.

Recommandations :

- mettre à jour le noyau Linux,
- redémarrer les systèmes concernés,
- valider les mitigations,
- revoir les politiques namespaces,
- surveiller les comportements anormaux du page-cache.

---

# 📅 Chronologie

| Date | Événement |
|---|---|
| Janvier 2017 | Introduction de la faille xfrm |
| Juin 2023 | Introduction de la faille RxRPC |
| Mai 2026 | Intégration des correctifs officiels |

---

# 📚 Objectifs du Projet

Ce dépôt est consacré à :

- la recherche défensive,
- l’analyse du noyau Linux,
- la sensibilisation cybersécurité,
- l’étude du page-cache,
- les mécanismes de mitigation,
- le hardening Linux.

---

# ⚠️ Notice Légale & Éthique

Ce projet est destiné exclusivement :

- à la recherche défensive,
- à des environnements de laboratoire autorisés,
- à la sensibilisation cybersécurité,
- à l’éducation technique.

N’utilisez jamais ces concepts contre des systèmes sans autorisation explicite.

---

# 👤 Crédits

## Recherche Originale

- **Hyunwoo Kim (@v4bel)**

## Projet Francophone de Sensibilisation

- **@Maxime288**

---

<div align="center">

## ⭐ Soutenir le Projet

Si ce dépôt vous a aidé :

⭐ Ajoutez une étoile au projet  
🔁 Partagez-le avec la communauté sécurité  
🛡️ Encouragez la recherche défensive

---

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:dc2626,100:0f172a&height=120&section=footer"/>

</div>
