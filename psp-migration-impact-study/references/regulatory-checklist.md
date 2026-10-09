# Cadre réglementaire d'une migration PSP (UE / France)

But : transformer les « zones grises » en « cadre légal connu, à confirmer avec le
PSP ». Lance une recherche documentaire (délègue à un sous-agent avec accès web si
disponible) sur les 4 questions ci-dessous, puis confronte à la doc du PSP cible.

Rappel : la loi dit ce qui est **autorisé** ; la faisabilité technique/commerciale
propre au PSP cible est un autre sujet, à confirmer avec lui. Sépare toujours les
deux dans le livrable.

## Q1 — Portabilité du KYC / vigilance LCB-FT entre PSP

Question : le PSP cible peut-il s'appuyer sur les vérifications d'identité déjà
faites par le PSP source, sans tout revérifier ?

Cadre (FR + UE) :
- **Tierce introduction** (« reliance on third parties ») : art. **L561-7 CMF** ;
  Directive (UE) 2015/849 (AMLD) art. **25-29** ; règlement AML UE **2024/1624**
  (AMLR) art. **48-50**, applicable à partir du 10/07/2027.
- Principe : reprise **autorisée**, mais le PSP cible **demeure responsable** ; cela
  ne couvre que l'identification initiale, pas la vigilance continue. **Aucune
  obligation légale de re-KYC** du seul fait d'un changement de PSP ; mais le PSP
  cible peut, par approche des risques, rafraîchir les dossiers anciens/incomplets.
- Conditions : contrat écrit, transmission sans délai des pièces, tiers assujetti
  supervisé dans l'UE.

Nuance pratique décisive : même si la reprise est légale, **il faut qu'il existe
quelque chose à reprendre**. Si l'ancien PSP n'a pas de preuve de vivant (liveness)
pour une partie du stock (comptes anciens), le re-parcours d'identité devient le
scénario de base pour ces comptes, indépendamment du droit. Le re-parcours peut
rester **court** (vérification d'identité + quelques questions) si le reste est
pré-remplissable.

À confirmer avec le PSP cible : accepte-t-il la reprise ? format de dossier exigé ?
périmètre de re-vérification par les risques ? récupération des certificats de
liveness auprès du PSP source ?

## Q2 — Transfert des fonds cantonnés (safeguarding) entre PSP

Question : comment les fonds clients passent du cantonnement source au cantonnement
cible, et combien de temps les comptes sont gelés ?

Cadre :
- Cantonnement EP : art. **L522-17 CMF** (DSP2, dir. (UE) 2015/2366 art. 10).
- Cantonnement EME : art. **L526-32 CMF** (dir. 2009/110/CE art. 7).
- **Pas de régime de « mobilité »** automatique pour les comptes de paiement
  (contrairement au mandat de mobilité bancaire L312-1-7 CMF, réservé aux comptes
  bancaires de consommateurs). La migration passe par un **transfert de fonds réel**
  (pour le compte des clients) + une **ré-affectation compte par compte**, encadrés
  contractuellement. Point d'attention ACPR : traiter les clients ne souhaitant pas
  migrer.

À confirmer avec le PSP cible : mécanique du virement de masse, format du fichier de
réconciliation, timing de bascule, outil de recréditation en masse.

## Q3 — Migration des mandats SEPA (SDD) avec changement d'ICS

Question : les mandats existants peuvent-ils être migrés sans re-signature quand
l'identifiant créancier (ICS) change ?

Cadre :
- **EPC SEPA Core Direct Debit Rulebook** : mécanisme d'**amendment** (champs
  **AT-24** motif, **AT-18** ancien créancier, **AT-19** ancienne référence de
  mandat). L'autorisation d'origine **reste valable** ; seule une **notification du
  débiteur** est requise, **pas de re-signature**.
- Point déterminant : **qui porte l'ICS** aujourd'hui. Si le client conserve son
  propre ICS et que seul le PSP collecteur change, il n'y a même pas de changement
  d'émetteur. Si c'est l'ICS du PSP source, il faut un amendement (toujours sans
  re-signature).

À confirmer avec le PSP cible : sait-il porter l'amendement (ancien ICS + ancienne
UMR dans les collectes) et reprendre les UMR existantes ? modalités de notification.

## Q4 — Authentification forte (SCA / DSP2)

Question : pour l'activité concernée (ex. collecte crowdfunding), faut-il une SCA, et
qui l'implémente ?

Cadre :
- DSP2 art. 97 + RTS règlement délégué (UE) **2018/389**. SCA obligatoire sur
  l'accès au compte et l'initiation d'un paiement par le payeur.
- Répartition typique :
  - Payin virement entrant : SCA gérée par la **banque du payeur** (rien à faire
    côté plateforme).
  - Payin carte : **3DS** géré par le prestataire carte ; la plateforme redirige.
  - **Investir (transfert interne initié par l'utilisateur), retirer, ajouter un
    bénéficiaire** : SCA requise ; **la plateforme intègre le mécanisme fourni par le
    PSP cible** (SDK / passkey). C'est le vrai chantier SCA.
  - Prélèvement SEPA : hors périmètre SCA côté payeur (initié par le créancier).
- **Exemptions** (art. 13 bénéficiaire de confiance, 14 récurrent, 15 self, 18 TRA) :
  décidées par le PSP, à activer pour réduire la friction sur « investir ».

À confirmer avec le PSP cible : périmètre SCA exact sur les flux, exemptions
activables, mécanisme fourni.

## Posologie dans le livrable

- Reformuler chaque zone grise en « la loi autorise X ; reste à confirmer avec le
  PSP : Y ».
- Ne jamais présenter un paramètre commercial du PSP (ex. seuil « KYC de moins d'1
  an ») comme une fatalité : c'est **négociable** (seuil, date de référence figée au
  lancement de la migration).
