---
name: psp-migration-impact-study
description: >-
  Méthodologie complète pour étudier l'impact d'un changement de prestataire de
  services de paiement (PSP) sur une plateforme fintech : cartographie des points
  de contact dans le code, flux par scénario utilisateur, confrontation à la
  documentation du PSP cible et au cadre réglementaire (LCB-FT, DSP2, SEPA, SCA),
  segmentation de la base client par requêtes SQL, et production de livrables de
  décision pour le client. Déclencher cette skill dès qu'un PM doit évaluer, cadrer
  ou chiffrer une migration de PSP (ex. Lemonway, Treezor, Mangopay, Swan, Stripe,
  Adyen), répondre à « quel est l'impact de changer de PSP », « migration
  prestataire de paiement », « étude d'impact PSP », « on change de PSP », «
  cartographie des points de contact du PSP », « qui doit refaire son KYC », «
  combien de comptes à migrer », même si le mot « skill » n'est pas prononcé.
---

# Étude d'impact d'une migration de PSP

## À quoi sert cette skill

Changer de prestataire de paiement (PSP) touche le cœur financier d'une plateforme :
ouverture de comptes, vérification d'identité (KYC), encaissements, prélèvements,
versements, remboursements. Cette skill guide un PM pour produire une **étude
d'impact rigoureuse et chiffrable**, du code jusqu'au livrable de décision client.

Elle a été distillée d'une mission réelle (migration Lemonway → Treezor). Les
exemples citent ces deux PSP, mais la démarche est **générique** : remplace les noms
par ton PSP source et ton PSP cible.

## Principe directeur (à garder en tête tout du long)

Une migration PSP n'est pas « remplacer un code par un autre ». Ce sont **trois
chantiers distincts, de risques très différents** :

1. **Réimplémentation** des parcours (nouveau code pour le PSP cible).
2. **Migration des données** existantes (comptes, KYC, mandats, soldes, IDs).
3. **Bascule / cutover** sans coupure ni perte de fonds.

Le code est le chantier le moins risqué. **La donnée et le cutover tuent les
migrations PSP.** Oriente l'analyse en conséquence : ne te laisse pas absorber par
la liste des endpoints, le vrai enjeu est « qui/quoi migrer, dans quel état, sans
rien casser ».

## Déroulé en 5 phases

Travaille dans l'ordre. Chaque phase produit un livrable intermédiaire qui nourrit
la suivante. Annonce au PM le livrable de chaque phase et fais-le valider avant
d'enchaîner.

### Phase 0 — Cadrage

Avant de toucher au code, poser les arbitrages bloquants. Ils déterminent 50 % de la
charge et ne se devinent pas dans le code :

- **Statut réglementaire** : le PSP cible opère sous son propre agrément. Qui porte
  la relation client réglementaire après migration ?
- **Reprise KYC** : le PSP cible accepte-t-il les KYC déjà validés, ou faut-il tout
  re-vérifier ? (voir `references/regulatory-checklist.md` : c'est souvent légalement
  possible mais techniquement contraint.)
- **Sort des fonds** : comment les soldes clients passent du cantonnement source au
  cantonnement cible.
- **Deadline** : fin de contrat du PSP source = deadline dure du cutover.

Livrable : note de cadrage listant ces arbitrages et à qui poser chaque question
(PSP cible, PSP source, client).

### Phase 1 — Cartographie des points de contact

Objectif : inventaire exhaustif de chaque endroit où la plateforme parle au PSP
source.

1. **Trouver le PSP dans le code.** Cherche le gem/package d'intégration et son nom
   dans les dépendances, puis l'ampleur du couplage :
   - `Gemfile`/`package.json`/`requirements.txt` pour la lib PSP.
   - `grep -riE "<psp>|wallet|kyc|payin|payout|mandate|sdd"` sur `app/ lib/ config/`.
2. **Classer chaque point de contact** par type : appel API sortant, webhook
   entrant, colonne DB stockant un ID/statut PSP, service, worker/cron, transaction,
   UI (iframe/widget/redirect), admin/back-office, config/ENV, dictionnaire de
   mapping de codes, mailer.
3. **Produire un tableau** : `fichier:ligne | type | objet métier | description`.

Pour un repo volumineux, délègue le balayage à un sous-agent d'exploration (si
disponible) avec une consigne précise par type et par zone. Demande-lui un tableau
`fichier:ligne` et une synthèse (colonnes DB, ENV, webhooks, objets métier).

Livrable : document « cartographie » (voir structure interne, pas destiné au client).

### Phase 2 — Flux par scénario utilisateur

Objectif : décrire les parcours réels, pas les endpoints. Un flux par parcours, en
séquence acteur → plateforme → PSP → retour, chaque appel PSP marqué comme point à
re-câbler.

Scénarios typiques d'une plateforme d'investissement (adapte au métier) :
onboarding particulier, onboarding personne morale, alimentation (carte / virement /
prélèvement), investir, être remboursé (interne), retirer (externe), cas admin.

Point d'attention récurrent : **distingue « remboursement » (transfert interne
wallet→wallet, reste dans le PSP) de « retrait/payout » (sortie vers compte bancaire
externe)**. Ce sont deux scénarios, deux impacts.

Livrable : document « flux par scénario » (interne).

### Phase 3 — Impact vs PSP cible + cadre réglementaire

Objectif : pour chaque scénario/point de contact, confronter à ce que le PSP cible
sait faire et à ce que la loi autorise.

1. **Documentation du PSP cible** : recherche en ligne (délègue à un sous-agent si
   disponible). Mappe objet par objet (compte/wallet, onboarding/KYC, documents,
   payin carte, payin virement/IBAN, payout/bénéficiaire, transfert interne, mandat
   SDD, webhooks, authentification). Note pour chacun : équivalent cible, écart, qui
   l'implémente, ce qui reste « à confirmer ».
2. **Cadre réglementaire** : lance la recherche décrite dans
   `references/regulatory-checklist.md` (reprise KYC, transfert de fonds cantonnés,
   migration des mandats SEPA, SCA). Transforme les « zones grises » en « cadre légal
   connu, à confirmer avec le PSP ».

Structure ce livrable **par scénario** (calqué sur la Phase 2), format « flux actuel
→ ce qui change avec le PSP cible ». C'est plus lisible qu'un classement par objet.

Livrable : document « impact PSP cible » (interne, base du chiffrage).

### Phase 4 — Segmentation de la base client (données réelles)

C'est la phase qui transforme une étude théorique en décision chiffrée. Objectif :
savoir **qui/quoi migrer, dans quel état, pondéré par l'enjeu financier**.

Lis `references/sql-segmentation.md` pour les requêtes (à adapter aux noms de tables
du projet). Les axes à produire :

- **Volumes & statuts** : total comptes, validés vs non validés, physique vs morale.
- **Segmentation des validés** selon l'activité : « à maintenir » (actif ∪ receveur
  futur ∪ solde > 0) vs « dormants ». Ne migre/re-vérifie en priorité que les actifs.
- **Preuve de vivant / re-vérification** : combien de comptes réutilisables vs à
  re-vérifier (souvent déductible d'un statut d'onboarding : un tunnel récent inclut
  un liveness, l'ancien non).
- **Résidence fiscale** regroupée (national / UE hors national / hors UE / US).
- **Mandats de prélèvement actifs** (pas seulement signés).
- **Multi-wallets par compte** (entiercements jamais fermés ; actifs vs créés).
- **Capital en cours** : l'enjeu financier réel. Attention au piège ci-dessous.

**Pièges de données à toujours vérifier** (ils ont tous mordu sur une vraie mission) :

- **Ne jamais confondre deux axes de segmentation.** « Actif » (activité récente) et
  « liveness » (preuve de vivant) sont orthogonaux ; leurs totaux peuvent être
  proches par coïncidence. Croise-les dans un tableau, ne les additionne pas.
- **Trésorerie wallet ≠ capital investi.** Le solde des wallets investisseurs est de
  la trésorerie dormante, pas l'argent en jeu. Le capital en cours de remboursement
  vit ailleurs (wallets projets) ou a déjà quitté la plateforme et transitera au fil
  des remboursements. Mesure-le à part (somme du capital restant dû).
- **Colonnes « cache »** (ex. un solde recopié en base) ne sont pas la source de
  vérité : signale-le, recoupe avec le PSP si possible.
- **Vérifie l'unité** (euros vs centimes) par un ordre de grandeur (montant / nombre
  de lignes) avant de publier un chiffre.
- **Un attribut métier peut être une méthode, pas une colonne** : vérifie le schéma
  avant d'écrire une requête (ex. `amount_in_cents` calculé depuis `amount`).
- **Extraction atomique** : idéalement une seule photo datée ; sinon signale les
  écarts ±1 entre requêtes lancées à des instants différents.

Donne les requêtes SQL au PM pour qu'il les lance lui-même sur la base ; tu ne te
connectes pas à la base de production.

Livrable : **socle de données** (voir ci-dessous).

## Livrables

Produis **deux documents distincts + un socle de données commun** (voir
`references/deliverable-structure.md` pour les plans détaillés) :

1. **Socle de données** (annexe commune, source unique de vérité) : uniquement des
   **agrégats, aucune donnée personnelle** → partageable sans risque RGPD. Les deux
   autres livrables le citent sans recalculer.
2. **Livrable client (décision)** : audience = direction du client. Objectif =
   décider go/no-go, comprendre la friction utilisateur, arbitrer risque/coût.
   Résumé exécutif en tête + détail par parcours.
3. **Livrable PSP cible (profil de données)** : audience = intégration/conformité du
   PSP cible. Objectif = leur transmettre l'état de la base pour qu'ils cadrent et
   chiffrent, + les questions à confirmer.

Mets en page en HTML imprimable (A4). Si une direction artistique (DA) est
disponible dans le projet (skill ou tokens de marque), applique-la ; sinon pars des
templates `assets/socle-template.html` et `assets/deliverable-template.html`.

## Règles de rédaction (livrables client)

- **Vocabulaire métier, zéro jargon technique.** Le lecteur est un décideur, pas un
  dev. « compte de paiement », « vérification d'identité », « mandat de prélèvement »,
  pas « wallet », « endpoint », « webhook ».
- **Distinguer, pour chaque parcours, ce qui ne change pas de ce qui change**, du
  point de vue utilisateur, avec un **niveau de friction** (imperceptible / sensible
  / fort). Précise que la friction UX n'a aucun rapport avec la complexité technique.
- **Mettre en avant les points de friction investisseur** : re-KYC, changement de
  RIB (échecs silencieux des virements récurrents, risque de perception « phishing »
  sur un email « votre RIB a changé »), re-signature ou notification de mandats,
  authentification forte sur les gestes clés.
- **Pas de tiret cadratin « — »** : remplace par deux-points, virgule, point ou
  parenthèses.
- **Gras minimal** : réserve-le aux libellés de lignes, titres, termes de
  définition. Jamais en plein milieu d'une phrase pour « insister ».
- **Citer le socle** pour tout chiffre ; ne recalcule pas dans plusieurs documents.
- **Honnêteté intellectuelle** : signale les hypothèses, les colonnes cache, les
  chiffres « à confirmer », les échantillons. Un auditeur strict ne doit rien
  trouver de caché. Quand un chiffre repose sur un proxy, dis-le.

## Gouvernance et RGPD (non négociable)

- **Ne transmettre aucune donnée personnelle à un PSP sans le feu vert explicite du
  client** (c'est sa base, sa responsabilité de traitement) et sans cadre RGPD
  (accord de sous-traitance, base légale, information des utilisateurs).
- Les étapes de screening / envoi de données/documents au PSP cible sont **gelées**
  tant que ce feu vert n'est pas donné : décris-les dans le livrable comme prérequis
  bloquant, pas comme action à lancer.
- Toi (l'assistant) **ne transmets rien à l'extérieur** : les requêtes servent au PM
  en interne. Les livrables partagés ne contiennent que des agrégats.

## Bonnes pratiques de collaboration avec le PM

- Avance par phase, fais valider chaque livrable intermédiaire.
- Quand le PM donne un chiffre ou une correction métier, il a raison sur le métier :
  intègre et propage la cohérence dans tous les documents (un chiffre changé dans le
  socle doit être répercuté dans les livrables).
- Propose une critique stricte de tes propres livrables quand on te le demande :
  liste les trous, les réserves non résolues, ce qui n'est pas « bulletproof ».
- Garde en mémoire (fichiers mémoire projet) les décisions et chiffres clés pour
  reprendre la mission sans tout réexpliquer.

## Références

- `references/sql-segmentation.md` : requêtes SQL de segmentation (à adapter).
- `references/regulatory-checklist.md` : questions réglementaires (LCB-FT, DSP2,
  SEPA, SCA) et où chercher.
- `references/deliverable-structure.md` : plans détaillés des trois livrables.
- `assets/socle-template.html` et `assets/deliverable-template.html` : gabarits HTML
  A4 imprimables (DA à adapter).
