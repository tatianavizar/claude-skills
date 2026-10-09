---
name: etude-besoin-produit
description: >-
  Construit une étude de besoin + un business case chiffré pour justifier (ou
  non) un produit — app mobile, feature, nouveau parcours — à partir de la data
  interne (Google Analytics + base SQL/Metabase) croisée avec des benchmarks
  marché sourcés, et produit un livrable C-suite. Déclencher quand un PM veut
  « prouver le besoin », « faire une étude de besoin », « monter un business
  case », « chiffrer le ROI / le payback », « justifier une app ou une feature »,
  « convaincre le comex / la direction », « est-ce que ça vaut le coup de
  construire X », ou partage de la data (GA, export SQL) en demandant quoi en
  conclure. FR & EN.
---

# Étude de besoin produit + business case

Objectif : transformer de la **data** (interne + marché) en une **décision** défendable
(go / no-go), et la présenter à des décideurs. La méthode marche pour une app mobile,
une feature, un nouveau parcours — toute décision de build à justifier.

## Règle d'or : advocate, pas commercial
Le PM d'agence défend la **valeur pour le client**, pas la vente du projet. Un dossier
crédible dit la vérité, **y compris les conditions et les cas où il ne faut PAS le
faire**. C'est ça qui emporte la décision, pas l'enthousiasme.

## Les 6 étapes

### 1. Cadrer la décision
- Une phrase : **quelle décision** cette étude éclaire (ex. « construire une app mobile ? »).
- **Ce que le produit sert vraiment** (souvent ≠ l'intuition de départ). Ex. une app = canal
  de rétention/activation, pas forcément d'acquisition. Pose-le explicitement, ça recadre tout.
- Ce qu'il ne sert PAS (évite de sur-promettre).

### 2. Prouver le besoin par la data INTERNE
Deux sources, complémentaires — voir `references/data-interne.md` (guide GA + requêtes SQL).
- **Google Analytics** : part mobile/desktop, split OS, **entonnoirs de conversion par device**
  (c'est souvent là que se cache l'argument : la majorité de l'audience sur le support qui
  convertit le moins).
- **Base SQL / Metabase** : la vérité métier que GA n'a pas — volumes réels, fidélité (cohortes),
  gisement d'activation, features sous-exploitées, ticket moyen/médian, revenus.
- ⚠️ GA et la DB ne mesurent pas la même chose. GA sous-compte (consentement cookies) → sers-t'en
  pour les **ratios** (device), et de la DB pour les **volumes absolus**.

### 3. Confronter au MARCHÉ (benchmarks externes)
Voir `references/benchmarks.md`. Tes chiffres internes n'ont de sens que **situés**. Cherche des
repères publics sourcés (avec niveau de fiabilité), et sers-t'en pour dire si tes hypothèses sont
hautes ou basses. **Le meilleur benchmark reste ta propre data historique** (cohortes).

### 4. Construire le business case bottom-up
Voir `references/business-case.md`. Jamais un uplift « sorti du chapeau ». Chaque levier =
**donnée réelle × un seul taux hypothétique (justifié) × valeur unitaire (ticket médian)**. On
additionne → uplift total → on projette sur la croissance → payback vs coût de build.
⚠️ Raisonne sur le **revenu réel du client** (sa marge/commission), pas sur le volume brut.

### 5. Lecture stratégique (registre conseil)
Quatre temps, dans cet ordre : **ce qui est solide · ce qui conditionne le résultat · quand ce
ne serait PAS le bon investissement · notre lecture**. Pas de « GO » triomphal. Recommande
souvent de **piloter/tester avant de committer** le gros du budget.

### 6. Livrable C-suite
Voir `references/livrable.md`. Un document court, hiérarchisé, à la DA du client, avec les
**données brutes documentées à part** (sans interprétation) pour résister au challenge.

## Principes d'honnêteté intellectuelle (ce qui distingue un bon dossier)
- **Raffine avant de titrer.** Un gros chiffre brut (« 2,1 M€ dormants ») est souvent trompeur :
  applique un seuil, croise, et regarde le chiffre NET avant de l'afficher.
- **Médiane > moyenne** dès qu'il y a des outliers (un gros ticket déforme la moyenne).
- **Exclure les pics exceptionnels** (op de com, saisonnalité) du « trend moyen ».
- **Dénominateur juste** : « 5,5 % des investisseurs » ≠ « 1,3 % de la base ». Dis lequel.
- **Descriptif ≠ causal** : « les users app convertissent 2× mieux » = biais de sélection, pas un
  uplift causal. Ne les mélange pas.
- **Vérifie avant d'écrire** dans le livrable (ne pas affirmer puis corriger).
- **Flag toute hypothèse** comme telle, et nomme ses sources.

## Garde-fous
- Ne jamais présenter une hypothèse comme un fait (surtout la marge/économie du client si elle
  est « connue en off »).
- Toujours séparer **data réelle** (gras, sourcée) et **taux d'amélioration** (hypothèse à valider).
- Si un chiffre clé manque (coût de build, marge, collecte), **le demander** — ne pas inventer ;
  à défaut, poser une hypothèse explicitement marquée *ILLUSTRATIF* et faire varier en scénarios.
