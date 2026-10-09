# Plans détaillés des livrables

Trois livrables. Le **socle** est la source unique de vérité (agrégats, RGPD-safe) ;
les deux autres le citent. Mise en page HTML A4 imprimable. Applique la DA du projet
si disponible, sinon pars des gabarits `assets/*.html`.

## 1. Socle de données (annexe commune, partageable)

Uniquement des agrégats, aucune donnée personnelle.

1. **Volumes & statuts** : total profils, validés / non validés, physique / morale.
2. **Segmentation des validés** : wallets à maintenir vs dormants ; trésorerie wallet
   et **capital en cours de remboursement** présentés comme deux montants distincts.
3. **Volume de re-vérification** (preuve de vivant) sur toute la base validée :
   réutilisables (liveness) vs à re-vérifier (legacy), avec la priorité aux actifs.
4. **Résidence fiscale** (national / UE hors national / hors UE / US).
5. **Prélèvements SEPA** : signés vs actifs.
6. **Disponibilité des données par exigence du PSP cible** : présent / à retraiter /
   manquant, pour physique, morale, documents. Puis une synthèse « données
   manquantes : à qui les redemander » (investisseurs / à demander au PSP source, en
   distinguant ce qui est récupérable par API de ce qui est une vraie question).
7. **Multi-wallets par compte** : créés vs actifs.

Dans le corps, expliciter les définitions (validé, actif, receveur futur, wallet à
maintenir, proxy liveness) et les réserves (colonnes cache, extraction non atomique)
ou les retirer si le client préfère un document épuré, en les gardant en mémoire.

## 2. Livrable client (décision)

Audience : direction du client. Objectif : décider, comprendre la friction, arbitrer.
Format : **résumé exécutif en tête, puis détail par parcours**.

1. **Résumé exécutif** (encadré) : population réelle à migrer, enjeu financier,
   frictions certaines, friction nouvelle (SCA), faible impact ailleurs,
   recommandation (migrer par batch avant mise en production, soigner la
   communication RIB, lever les prérequis).
2. **Comment lire** : légende ✅ ne change pas / 🔄 change + friction 🟢🟡🔴 ; préciser
   que la friction UX est indépendante de la complexité technique.
3. **Chiffres clés** : deux axes distincts (qui migrer selon l'activité ; effort de
   vérification selon la preuve de vivant) + un tableau croisé des deux axes.
4. **Impact par parcours** (un par scénario de la Phase 2) : tableau ne change pas /
   change + niveau de friction. Inclure une section transverse pour l'authentification
   forte.
5. **Synthèse de la friction** par parcours.
6. **Impact équipes internes** (admin) : nouvel outil de supervision, réconciliation,
   vague de support, campagne de re-vérification, récupération des données manquantes
   avant résiliation.
7. **Démarche de migration** : processus proposé par le PSP cible (vulgarisé) +
   prérequis bloquant RGPD.
8. **Étapes de la migration** (macro, séquencées).
9. **Points à trancher** (avec le PSP cible / le PSP source / décisions internes).
10. **Risques et coûts** (le « bénéfice » est au client de le définir ; ne pas le
    remplir à sa place).

## 3. Livrable PSP cible (profil de données)

Audience : intégration / conformité du PSP cible. Objectif : leur transmettre l'état
de la base pour cadrer/chiffrer.

1. Contexte & périmètre.
2. **Profil de la base** (reprend le socle) : volumes, cohortes, preuve de vivant,
   wallets à maintenir vs dormants, résidence, mandats, multi-wallets.
3. **Disponibilité par champ requis** par le PSP cible (mapping + trous).
4. **Architecture cible** (schéma de flux, pour confirmer la compréhension).
5. **Points à confirmer** (questions ouvertes).

Rappel RGPD : ce document reste au niveau agrégat. L'envoi de données nominatives
(listes de screening, documents) est **gelé** tant que le client n'a pas donné son
feu vert et mis en place le cadre RGPD.
