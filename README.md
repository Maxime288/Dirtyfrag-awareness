# DirtyFrag Awareness

> Sensibilisation à la vulnérabilité **Dirty Frag** — Analyse d’Élévation Locale de Privilèges (LPE) du noyau Linux.

---

## ⚠️ Avertissement

Ce projet est fourni exclusivement à des fins :

- de sensibilisation,
- de recherche en sécurité,
- de défense préventive.

Aucun exploit weaponized n’est inclus dans ce dépôt.

N’utilisez jamais ces informations sur des infrastructures de production sans autorisation explicite.

> ⚠️ Les éléments présentés dans ce projet constituent un scénario de recherche en sécurité à vocation pédagogique. Les identifiants CVE utilisés servent à structurer l’analyse technique et ne préjugent pas nécessairement d’une publication officielle.

---

## 📌 Présentation

DirtyFrag est une vulnérabilité de type **Local Privilege Escalation (LPE)** affectant le noyau Linux.

Elle peut permettre à un utilisateur local authentifié — même fortement restreint — d’obtenir une exécution de code avec des privilèges élevés, potentiellement jusqu’à `root`, dans les environnements concernés.

Le projet a pour objectif de :

- vulgariser le fonctionnement interne de la faille,
- documenter les composants impactés,
- expliquer la chaîne d’exploitation,
- proposer des mesures de mitigation immédiates.

---

# 🔍 Qu’est-ce que Dirty Frag ?

DirtyFrag est une vulnérabilité affectant plusieurs sous-systèmes du noyau Linux, notamment :

- la gestion mémoire,
- le `page-cache`,
- certains composants réseau.

Elle est conceptuellement proche de vulnérabilités telles que :

- Dirty Pipe (2022),
- Copy Fail (2024).

La vulnérabilité exploite une corruption liée au membre `frag` de la structure réseau Linux `sk_buff`.

---

# ✨ Caractéristiques principales

## ✅ Déterminisme logique

Aucune condition de compétition (race condition) n’est requise dans le modèle analysé.

## ✅ Stabilité élevée

Les tentatives échouées n’entraînent généralement pas de :

- kernel panic,
- crash système,
- redémarrage immédiat.

## ✅ Contournement de certaines mitigations

Dirty Frag peut contourner certaines protections mises en place contre :

- Copy Fail,
- `algif_aead`,
- blacklist de modules spécifiques.

## ✅ Présence historique

Le code vulnérable serait présent dans la branche principale du noyau Linux depuis environ **9 ans**, selon l’analyse du projet.

---

# 🧩 Chaîne d’exploitation

La chaîne combine deux vulnérabilités distinctes afin de couvrir différents environnements système.

---

## CVE-2026-43284 — xfrm-ESP Page-Cache Write

| Élément | Détails |
|---|---|
| Composant affecté | Sous-système IPsec / XFRM |
| Primitive | Écriture contrôlée de 4 octets dans le page-cache |
| Introduction du bug | `cac2661c53f3` (17 janvier 2017) |
| Correctif officiel | `f4c50a4034e6` (5 mai 2026) |
| Condition requise | User namespace |

---

## CVE-2026-43500 — RxRPC Page-Cache Write

| Élément | Détails |
|---|---|
| Composant affecté | `rxrpc.ko` |
| Primitive | Écriture contrôlée dans le page-cache |
| Introduction du bug | `2dc334f1a63a` (8 juin 2023) |
| Correctif officiel | `aa54b1d27fe0` (10 mai 2026) |
| Condition requise | Aucun privilège namespace |

---

# 🎯 Logique de ciblage croisé

Ubuntu → RxRPC  
RHEL → xfrm + namespaces  

Résultat : élévation de privilèges observée dans plusieurs environnements Linux testés.

---

# 🛡️ Mitigation immédiate

```bash
sudo sh -c "printf 'install esp4 /bin/false
install esp6 /bin/false
install rxrpc /bin/false
' \
 > /etc/modprobe.d/dirtyfrag.conf; \
 rmmod esp4 esp6 rxrpc 2>/dev/null; \
 sync && echo 3 > /proc/sys/vm/drop_caches; true"
```

---

# 🧹 Purge du page-cache

```bash
sudo sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

---

# 📅 Chronologie

- 2017 : introduction xfrm-ESP
- 2023 : introduction RxRPC
- 2026 : correctifs kernel

---

# 👤 Crédits

- Hyunwoo Kim (@v4bel)
- @Maxime288

---

# 📚 Objectif

Documentation, sensibilisation et recherche défensive.

Aucun exploit offensif fourni.
