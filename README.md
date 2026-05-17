# 🔴 DirtyFrag — Sensibilisation à la Vulnérabilité

> **Dépôt éducatif** — Sensibilisation à la faille de sécurité Dirty Frag (CVE-2026-43284 + CVE-2026-43500).  
> Source originale : [V4bel/dirtyfrag](https://github.com/V4bel/dirtyfrag) — Découverte par Hyunwoo Kim ([@v4bel](https://x.com/v4bel))

---

## ⚠️ Avertissement

Ce dépôt est fourni **à des fins de sensibilisation et de recherche en sécurité uniquement**. N'utilisez jamais ces informations sur des systèmes que vous n'êtes pas autorisé à tester. La divulgation a été coordonnée avec les mainteneurs du noyau Linux via `linux-distros@vs.openwall.org`.

---

## 🧠 Qu'est-ce que Dirty Frag ?

**Dirty Frag** est une vulnérabilité de classe **Local Privilege Escalation (LPE)** affectant le noyau Linux. Elle permet à un utilisateur local non privilégié d'obtenir les **droits root** sur les distributions Linux majeures.

Elle appartient à la même famille de bugs que [Dirty Pipe](https://dirtypipe.cm4all.com/) et [Copy Fail](https://copy.fail/), exploitant le **page-cache** du noyau Linux, mais en ciblant le membre `frag` de la structure `sk_buff`.

### Caractéristiques clés

- ✅ **Bug logique déterministe** — aucune race condition nécessaire
- ✅ **Taux de succès très élevé**
- ✅ **Pas de panic kernel** en cas d'échec
- ✅ **Bypass des mitigations Copy Fail** (blacklist algif_aead sans effet)
- ✅ **Affecte toutes les distributions majeures**
- ⏳ **Présente dans le noyau depuis ~9 ans**

---

## 🔍 Les deux CVE en chaîne

Dirty Frag enchaîne deux vulnérabilités pour couvrir les angles morts de chacune :

### CVE-2026-43284 — xfrm-ESP Page-Cache Write

| Propriété | Détail |
|-----------|--------|
| Composant | Sous-système IPsec/xfrm |
| Primitive | Écriture arbitraire de 4 octets dans le page-cache |
| Depuis | `cac2661c53f3` — 17 janvier 2017 |
| Corrigé | `f4c50a4034e6` — 5 mai 2026 |
| Contrainte | Requiert la création d'un namespace utilisateur |

> ⚠️ Ubuntu peut bloquer cette CVE via une politique AppArmor. C'est pour ça que CVE-2026-43500 est nécessaire.

### CVE-2026-43500 — RxRPC Page-Cache Write

| Propriété | Détail |
|-----------|--------|
| Composant | Module RxRPC |
| Primitive | Écriture dans le page-cache |
| Depuis | `2dc334f1a63a` — 8 juin 2023 |
| Corrigé | `aa54b1d27fe0` — 10 mai 2026 |
| Contrainte | **Pas de privilege de namespace requis** — mais `rxrpc.ko` absent sur beaucoup de distros |

> 💡 Sur Ubuntu, `rxrpc.ko` est chargé par défaut — ce qui en fait la cible idéale quand xfrm-ESP est bloqué.

### Pourquoi les chaîner ?

```
Ubuntu (AppArmor bloque xfrm-ESP)  →  RxRPC fonctionne
Autres distros (rxrpc.ko absent)   →  xfrm-ESP fonctionne

Résultat : ROOT sur TOUTES les distributions majeures
```

---

## 🖥️ Distributions confirmées vulnérables

- Ubuntu 24.04.4 — kernel `6.17.0-23-generic`
- RHEL 10.1 — kernel `6.12.0-124.49.1.el10_1.x86_64`
- openSUSE Tumbleweed — kernel `7.0.2-1-default`
- CentOS Stream 10 — kernel `6.12.0-224.el10.x86_64`
- AlmaLinux 10 — kernel `6.12.0-124.52.3.el10_1.x86_64`
- Fedora 44 — kernel `6.19.14-300.fc44.x86_64`
- Et d'autres...

---

## 🛡️ Mitigation

### Solution immédiate (désactiver les modules vulnérables)

```bash
sudo sh -c "printf 'install esp4 /bin/false\ninstall esp6 /bin/false\ninstall rxrpc /bin/false\n' \
  > /etc/modprobe.d/dirtyfrag.conf; \
  rmmod esp4 esp6 rxrpc 2>/dev/null; \
  echo 3 > /proc/sys/vm/drop_caches; true"
```

### Après une exploitation (nettoyage du page-cache)

```bash
# Vider le page-cache pollué
echo 3 > /proc/sys/vm/drop_caches

# Ou simplement redémarrer le système
sudo reboot
```

### Solution définitive

Mettre à jour le noyau Linux dès que votre distribution publie un backport des correctifs.

---

## 📅 Chronologie

| Date | Événement |
|------|-----------|
| 17 jan. 2017 | Introduction du bug xfrm-ESP (`cac2661c53f3`) |
| 8 juin 2023 | Introduction du bug RxRPC (`2dc334f1a63a`) |
| 5 mai 2026 | Patch xfrm-ESP mergé dans le mainline (`f4c50a4034e6`) |
| 7 mai 2026 | Publication anticipée (rupture d'embargo) — coordination linux-distros |
| 10 mai 2026 | Patch RxRPC mergé dans le mainline (`aa54b1d27fe0`) |

---

## 🔗 Relation avec d'autres vulnérabilités

- **[Dirty Pipe (2022)](https://dirtypipe.cm4all.com/)** — Vulnérabilité ancêtre, même classe de bugs page-cache. Dirty Frag en est un descendant direct.
- **[Copy Fail (2024)](https://copy.fail/)** — Motivation initiale de la recherche. Dirty Frag partage le même sink que Copy Fail, mais contourne sa mitigation (blacklist algif_aead).

---

## 📚 Ressources

- 🔴 [Dépôt original — V4bel/dirtyfrag](https://github.com/V4bel/dirtyfrag) (PoC + write-up technique complet)
- 🔧 [Patch kernel CVE-2026-43284](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=f4c50a4034e62ab75f1d5cdd191dd5f9c77fdff4)
- 🔧 [Patch kernel CVE-2026-43500](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=aa54b1d27fe0c2b78e664a34fd0fdf7cd1960d71)
- 📖 [Dirty Pipe — dirtypipe.cm4all.com](https://dirtypipe.cm4all.com/)
- 📖 [Copy Fail — copy.fail](https://copy.fail/)

---

## 👤 Crédits

- **Découverte & recherche** : Hyunwoo Kim ([@v4bel](https://x.com/v4bel))
- **Dépôt de sensibilisation** : [@Maxime288](https://github.com/Maxime288)

---

*Ce dépôt ne contient pas de code d'exploitation. Pour le PoC technique, référez-vous au dépôt original de l'auteur.*
