# DirtyFrag Awareness

> Sensibilisation à la vulnérabilité **Dirty Frag** — Analyse d’Élévation Locale de Privilèges (LPE) du noyau Linux.

## ⚠️ Avertissement

Ce projet est fourni exclusivement à des fins :

- de sensibilisation,
- de recherche en sécurité,
- de défense préventive.

Aucun exploit weaponized n’est inclus dans ce dépôt.

N’utilisez jamais ces informations sur des infrastructures de production sans autorisation explicite.

---

## 📌 Présentation

DirtyFrag est une vulnérabilité de type **Local Privilege Escalation (LPE)** affectant le noyau Linux.

Elle permet à un utilisateur local authentifié — même fortement restreint — d’obtenir une exécution de code avec les privilèges `root`.

Le projet a pour objectif de :

- vulgariser le fonctionnement interne de la faille,
- documenter les composants impactés,
- expliquer la chaîne d’exploitation,
- proposer des mesures de mitigation immédiates.

---

# 🔍 Qu’est-ce que Dirty Frag ?

Dirty Frag est une faille impactant profondément :

- la gestion mémoire du noyau Linux,
- le `page-cache`,
- le sous-système réseau.

Elle est considérée comme un descendant direct de :

- Dirty Pipe (2022),
- Copy Fail (2024).

La vulnérabilité exploite une corruption liée au membre `frag` de la structure réseau Linux `sk_buff`.

---

# ✨ Caractéristiques principales

## ✅ Déterminisme logique

Aucune race condition.

L’exploitation réussit de manière fiable sans dépendre d’aléas temporels.

## ✅ Stabilité élevée

Les tentatives échouées ne provoquent généralement pas de :

- kernel panic,
- crash système,
- redémarrage.

## ✅ Contournement de défenses

Dirty Frag contourne certaines mitigations mises en place contre :

- Copy Fail,
- `algif_aead`,
- blacklist de modules.

## ✅ Présence historique

Le code vulnérable était présent dans la branche principale du noyau Linux depuis environ **9 ans**.

---

# 🧩 Chaîne d’exploitation

La chaîne combine deux vulnérabilités distinctes.

---

## 1. CVE-2026-43284 — xfrm-ESP Page-Cache Write

| Élément | Détails |
|---|---|
| Composant affecté | Sous-système IPsec / XFRM |
| Primitive | Écriture arbitraire ciblée de 4 octets |
| Introduction du bug | `cac2661c53f3` (17 janvier 2017) |
| Correctif officiel | `f4c50a4034e6` (5 mai 2026) |
| Condition requise | User namespace |

### Particularité

Certaines distributions comme Ubuntu bloquent l’accès XFRM via AppArmor.

La seconde faille permet alors de contourner cette restriction.

---

## 2. CVE-2026-43500 — RxRPC Page-Cache Write

| Élément | Détails |
|---|---|
| Composant affecté | `rxrpc.ko` |
| Primitive | Écriture arbitraire dans le page-cache |
| Introduction du bug | `2dc334f1a63a` (8 juin 2023) |
| Correctif officiel | `aa54b1d27fe0` (10 mai 2026) |
| Condition requise | Aucun privilège namespace |

### Particularité

Le module RxRPC :

- est souvent absent sur les serveurs RHEL minimaux,
- mais présent par défaut sur Ubuntu Desktop.

---

# 🎯 Logique de ciblage croisé

```text
Ubuntu (AppArmor bloque xfrm-ESP)
    └──► Exploitation via RxRPC

RHEL / Distros Enterprise (RxRPC absent)
    └──► Exploitation via xfrm-ESP + user namespace
```

### Résultat

Accès `ROOT` possible sur une large variété de distributions Linux.

---

# 🧪 Variante : Fragnesia

Après publication initiale, une variante appelée **Fragnesia** (`CVE-2026-46300`) a démontré que certains correctifs partiels laissaient subsister des vecteurs d’attaque résiduels.

---

# 🖥️ Distributions validées vulnérables

Versions d’origine non corrigées :

- Ubuntu 24.04.4 LTS
- RHEL 10.1
- Fedora 44
- openSUSE Tumbleweed
- CentOS Stream 10
- AlmaLinux 10

---

# 🛡️ Mitigation immédiate

## Neutralisation des modules vulnérables

```bash
sudo sh -c "printf 'install esp4 /bin/false\ninstall esp6 /bin/false\ninstall rxrpc /bin/false\n' \
 > /etc/modprobe.d/dirtyfrag.conf; \
 rmmod esp4 esp6 rxrpc 2>/dev/null; \
 sync && echo 3 > /proc/sys/vm/drop_caches; true"
```

## ⚠️ Impact

Cette mitigation désactive temporairement :

- IPsec / ESP,
- certains VPN,
- AFS / RxRPC.

---

# 🧹 Purge du page-cache

Après audit ou test :

```bash
sudo sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

---

# ✅ Résolution définitive

La seule solution pérenne consiste à :

- appliquer les mises à jour de sécurité,
- installer les correctifs fournis par les distributions Linux.

---

# 📅 Chronologie technique

| Date | Événement |
|---|---|
| 17 Jan 2017 | Introduction du bug xfrm-ESP |
| 8 Juin 2023 | Introduction du bug RxRPC |
| 5 Mai 2026 | Correctif xfrm-ESP |
| 7 Mai 2026 | Coordination linux-distros |
| 10 Mai 2026 | Correctif RxRPC |

---

# 👤 Crédits

## Auteur original & découverte

**Hyunwoo Kim (@v4bel)**

Référence technique :
`V4bel/dirtyfrag`

## Dépôt de sensibilisation francophone

Maintenu par :

**@Maxime288**

Projet :
`Dirtyfrag-awareness`

---

# 📚 Objectif du dépôt

Ce dépôt est destiné à :

- la documentation technique,
- la sensibilisation cybersécurité,
- la recherche défensive,
- l’analyse du noyau Linux.

Aucun exploit offensif n’est fourni.
