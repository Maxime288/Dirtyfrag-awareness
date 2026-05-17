DirtyFrag — Sensibilisation à la Vulnérabilité
Analyse d'Élévation Locale de Privilèges (LPE) du Noyau Linux
TARGET: LINUX KERNEL TYPE: LOCAL PRIVILEGE ESCALATION CVE-2026-43284 / 43500
ÉDUCATIF & RECHERCHE

AVERTISSEMENT : Ce document et son dépôt associé sont fournis exclusivement à des fins de sensibilisation,
de recherche en sécurité et de défense préventive. N'utilisez jamais ces informations sur des architectures de
production sans autorisation explicite. La divulgation a fait l'objet d'une coordination globale avec les mainteneurs
du noyau Linux.

Objectif du Projet
Ce projet initié par la communauté a pour vocation de vulgariser et documenter le fonctionnement de la
vulnérabilité critique appelée Dirty Frag. Contrairement aux outils offensifs clés en main, cette documentation
analytique est dépourvue d'arme cybernétique (Weaponized Exploit). Elle met l'accent sur la mécanique interne du
sous-système réseau de Linux, sur l'identification des cibles et sur le déploiement de stratégies de mitigation
immédiates.

Qu'est-ce que Dirty Frag ?
Dirty Frag est une faille de type Local Privilege Escalation (LPE) impactant profondément la gestion de la
mémoire du noyau Linux. Elle offre à un utilisateur local authentifié, même doté des privilèges les plus restreints
(ex: utilisateur nobody ou un conteneur mal isolé), la capacité d'exécuter du code arbitraire avec les droits du
super-utilisateur root .
Cette faille est un descendant direct de vulnérabilités mémorables telles que Dirty Pipe (2022) ou Copy Fail (2024).
Toutes exploitent une corruption du page-cache (le mécanisme de mise en cache des pages de stockage en
RAM). La spécificité technique de Dirty Frag réside dans le détournement du membre frag au sein de la structure
fondamentale des paquets réseau Linux : la structure sk_buff .
Caractéristiques majeures de la faille
Déterminisme Logique : Contrairement à d'autres failles de noyau complexes, elle ne repose sur aucune race
condition (condition de concurrence). L'exécution réussit à chaque tentative sans aléa.
Stabilité Absolue : L'échec potentiel de l'alignement mémoire ne déclenche aucun kernel panic ou plantage de
la machine hôte. L'attaquant peut retenter sa primitive de corruption en toute discrétion.
Contournement de Défense : Elle neutralise de manière transparente les mitigations implémentées pour
bloquer Copy Fail, notamment la désactivation ou la mise sur liste noire du module algif_aead .
Persistance Historique : Le code vulnérable sous-jacent était silencieusement installé dans la branche
principale (mainline) du noyau Linux depuis environ 9 ans.
•

•

•

•

DirtyFrag-Awareness | Rapport de Sécurité Page 1 / 4

Anatomie de la Chaîne d'Exploitation
Pour garantir un taux de succès universel, l'attaque chaîne intelligemment deux vulnérabilités distinctes de manière
à contourner les mécanismes de durcissement spécifiques à chaque distribution Linux.
1. CVE-2026-43284 — xfrm-ESP Page-Cache Write
Propriété Détails Techniques
Composant affecté Sous-système IPsec / infrastructure de transformation xfrm
Primitive d'attaque Écriture arbitraire ciblée de 4 octets dans le page-cache du noyau
Introduction du bug Commit cac2661c53f3 — daté du 17 janvier 2017
Correctif officiel Commit f4c50a4034e6 — intégré le 5 mai 2026
Contrainte requise Nécessite la capacité à créer un namespace utilisateur ( user namespace )
Particularité de sécurité : Bien que certaines distributions comme Ubuntu autorisent la création d'espaces de noms
utilisateurs non privilégiés, des règles de profil strictes AppArmor bloquent nativement l'appel au sous-système
réseau XFRM. C'est ici qu'intervient la seconde faille.
2. CVE-2026-43500 — RxRPC Page-Cache Write
Propriété Détails Techniques
Composant affecté Module réseau RxRPC ( rxrpc.ko )
Primitive d'attaque Écriture de blocs arbitraires directement dans le page-cache
Introduction du bug Commit 2dc334f1a63a — daté du 8 juin 2023
Correctif officiel Commit aa54b1d27fe0 — intégré le 10 mai 2026
Contrainte requise Aucun privilège d'espace de noms requis. Nécessite simplement la présence

ou le chargement automatique du module réseau.

Particularité de sécurité : Sur de nombreuses configurations minimales de type serveurs d'entreprise (comme
RHEL standard), le module rxrpc.ko n'est pas installé ou est désactivé par défaut. Néanmoins, sur des
distributions orientées Desktop ou polyvalentes comme Ubuntu, ce module est disponible de base.

DirtyFrag-Awareness | Rapport de Sécurité Page 2 / 4

Logique de ciblage croisé

Ubuntu (AppArmor bloque xfrm-ESP) ──► Exploitation via le module RxRPC (Validé)
RHEL / Distros Pro (RxRPC absent) ──► Exploitation via user namespace + xfrm-ESP
(Validé)
Résultat global : Accès ROOT garanti sur l'ensemble du spectre des distributions Linux.
Note additionnelle sur Fragnesia : Peu de temps après la publication de ces deux failles, la variante **Fragnesia**
(référencée sous la vulnérabilité **CVE-2026-46300**) a été découverte, démontrant que les premiers correctifs
partiels laissaient subsister des vecteurs d'attaque résiduels sous certaines conditions d'assemblage de fragments
IP.

Distributions Validées Vulnérables (Versions d'Origine)
Les versions d'origine et non corrigées des distributions majeures listées ci-dessous ont été confirmées comme
pleinement exposées :
Ubuntu 24.04.4 LTS — Noyau de la branche stable standard 6.8.0-xx-generic
Red Hat Enterprise Linux (RHEL) 10.1 — Noyau d'origine 6.12.0-124.49.1.el10_1.x86_64
Fedora 44 — Noyau d'origine 6.19.14-300.fc44.x86_64
openSUSE Tumbleweed — Environnement basé sur le noyau 7.0.2-1-default
CentOS Stream 10 & AlmaLinux 10 — Noyaux d'origine de la branche 6.12
Guide de Mitigation et Remédiation Proactive
1. Action Immédiate : Neutralisation des Vecteurs Réseau
Si la mise à jour immédiate de vos serveurs en production s'avère impossible pour des raisons opérationnelles,
vous devez neutraliser préventivement le chargement des modules noyau incriminés. La commande ci-dessous
automatise ce blocage et vide le cache système :
sudo sh -c "printf 'install esp4 /bin/false\ninstall esp6 /bin/false\ninstall rxrpc /bin/false\n' \
> /etc/modprobe.d/dirtyfrag.conf; \
rmmod esp4 esp6 rxrpc 2>/dev/null; \
sync && echo 3 > /proc/sys/vm/drop_caches; true"
Attention : L'application de cette restriction rendra temporairement inutilisables les tunnels VPN s'appuyant sur les
protocoles IPsec/ESP, ainsi que les montages réseau s'appuyant sur le système de fichiers distribué AFS (RxRPC).
2. Action Post-Incident ou Post-Test : Purge du Page-Cache
Dans l'éventualité où une tentative d'audit ou un test d'intrusion a été mené, le page-cache de la machine peut se
retrouver dans un état instable ou pollué. Afin de restaurer l'intégrité de la mémoire vive sans procéder à un
redémarrage lourd de l'infrastructure, appliquez la commande suivante :
•
•
•
•
•

DirtyFrag-Awareness | Rapport de Sécurité Page 3 / 4

sudo sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
3. Résolution Définitive
La seule protection pérenne réside dans l'application des paquets de mise à jour fournis par vos éditeurs respectifs.
Les équipes de sécurité des distributions majeures ont intégré les rétro-corrections (backports) nécessaires.
Chronologie Technique Finale (Année 2026)
Date de l'Événement Description de l'Étape de Cycle de Vie
17 Janvier 2017 Introduction initiale du bug logique xfrm-ESP (commit historique cac2661c53f3 ).
8 Juin 2023

Introduction du second bug au sein du code RxRPC (commit historique
2dc334f1a63a ).

5 Mai 2026

Le correctif officiel de sécurité xfrm-ESP est fusionné dans la branche stable
(Mainline).

7 Mai 2026

Rupture anticipée de l'embargo de sécurité et coordination avec l'écosystème linux-
distros .

10 Mai 2026 Le correctif du module RxRPC est validé et injecté globalement.

Auteur Original & Découverte : Hyunwoo Kim (@v4bel) — Dépôt de référence technique : V4bel/dirtyfrag .
Dépôt de Sensibilisation Francophone : Développé et maintenu par @Maxime288 sur Dirtyfrag-awareness .
