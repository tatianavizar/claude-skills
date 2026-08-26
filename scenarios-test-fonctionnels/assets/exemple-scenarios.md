# Exemple de sortie conforme

Exemple complet de fichier généré par cette skill, à consulter en cas de doute sur la granularité attendue. Feature fictive mais réaliste : gestion multi-wallets sur une plateforme de financement participatif (BO admin + FO investisseur, PSP externe).

Ce que l'exemple illustre : une fixture dédiée par entité mutée avec la colonne `Muté par` renseignée (aucun scénario ne dépend de l'état laissé par un autre), criticité dans les titres, listes parallèles étapes/résultats où chaque résultat énonce un état observable et non l'action, ordre métier, mutualisation d'actions enchaînées par un même rôle (scénario 3 : paiement puis annulation), responsive intégré dans des scénarios existants, résilience infra (scénario 5 : timeout PSP), régression et feature flag traités.

Vérifier surtout la **traçabilité du tableau de couverture** : chaque ligne ✓ renvoie à un scénario dont un résultat attendu numéroté assère littéralement la contrainte, et la seule contrainte non couverte est marquée ✗ avec renvoi en point de vigilance.

---

# Scénarios de test — Multi-wallets par projet

## Contexte
- Feature : un wallet PSP distinct par projet publié (au lieu d'un wallet unique par porteur), + wallet séquestre par souscription d'investisseur non averti
- Brief utilisé (version initiale / éditée) : brief "Multi-wallets" — version éditée du 12/06, section "Réutilisation du wallet par défaut" ajoutée après le round 1
- Ticket(s) : CAPS-1428, CAPS-1431
- Repo / commit : `app-plateforme` @ `a3f19c2`
- Mode : Standard (brief + code)
- Monitoring : vérifié, 2 erreurs `Psp::TimeoutError` sur la création de wallet dans les 30 derniers jours → couvertes par le scénario 5

## Données de test à préparer

Toutes les entités citées en pré-conditions figurent ici : rôles, mais aussi projets, enveloppes, réseaux, comptes externes. `Muté par` = le scénario unique qui modifie l'état de l'entité, ou `—` si l'entité est en lecture seule sur tout le fichier.

| Identifiant | Rôle / état | Muté par | Détail |
|---|---|---|---|
| `admin-recette` | Admin BO | — | droits complets, aucun état propre modifié par les scénarios |
| `porteur-sans-projet` | Porteur de projet | sc1 | onboarding PSP complété, **aucun** projet publié ; gagne un projet en sc1 |
| `porteur-un-projet` | Porteur de projet | sc2 | onboarding complété, **exactement un** projet publié, pas d'IBAN ; gagne un projet et un IBAN en sc2 |
| `porteur-iban` | Porteur de projet | — | IBAN virtuel **déjà généré**, consulté en lecture seule en sc2 |
| `porteur-archive` | Porteur de projet | sc4 | 1 projet publié ; le projet est archivé en sc4 (irréversible) |
| `porteur-timeout` | Porteur de projet | sc5 | onboarding complété, projet prêt à publier ; publication en échec puis rattrapée en sc5 |
| `investisseur-nonaverti` | Investisseur FO | sc3 | KYC validé, statut **non averti**, solde suffisant ; souscrit puis annule en sc3 |
| `investisseur-sansko` | Investisseur FO | — | KYC **non validé** ; sa tentative de souscription en sc5 est refusée, aucun état créé |
| `investisseur-averti` | Investisseur FO | sc6 | KYC validé, statut **averti**, solde suffisant ; souscrit en sc6 |
| `projet-souscription-A` | Projet publié | sc3 | collecte ouverte ; reçoit la souscription de sc3 (la tentative refusée de sc5 ne le modifie pas) |
| `projet-souscription-B` | Projet publié | sc6 | collecte ouverte ; reçoit la souscription de sc6 |
| `projet-flag-off` | Projet publié + 1 souscription payée | — | consulté en lecture seule en sc7 |

## Scénarios

### Scénario 1 — Premier projet d'un porteur : réutilisation du wallet par défaut (Majeure)
- **Rôle / device** : `admin-recette`, desktop
- **Pré-conditions** : `porteur-sans-projet`, onboarding PSP complété, aucun projet publié ; flag `multi_wallets` actif
- **Étapes** :
  1. Ouvrir la fiche de `porteur-sans-projet` et noter l'identifiant de son wallet par défaut (section PSP)
  2. Créer et publier un premier projet pour ce porteur
  3. Ouvrir la fiche du projet, section wallets PSP
- **Résultats attendus** :
  1. Un identifiant de wallet par défaut est affiché, avec son solde en euros
  2. La publication aboutit sans message d'erreur
  3. Deux blocs sont présents — "Wallet projet (collecte)" et "Wallet porteur (remboursements)" — et l'identifiant du bloc collecte est **identique** à celui noté à l'étape 1 : aucun nouveau wallet créé côté PSP

### Scénario 2 — Second projet et génération d'IBAN virtuel (Majeure)
- **Rôle / device** : `admin-recette`, desktop puis **mobile** à l'étape 5
- **Pré-conditions** : `porteur-un-projet` avec exactement un projet publié et sans IBAN virtuel ; `porteur-iban` disponible en lecture ; flag `multi_wallets` actif
- **Étapes** :
  1. Créer et publier un second projet pour `porteur-un-projet`, puis noter l'identifiant du bloc "Wallet projet (collecte)"
  2. Comparer avec l'identifiant de collecte du projet déjà publié
  3. Depuis la fiche de `porteur-un-projet`, cliquer sur "Générer l'IBAN virtuel"
  4. Ouvrir la fiche de `porteur-iban`
  5. Rouvrir la fiche de `porteur-un-projet` sur mobile (viewport 375 px)
- **Résultats attendus** :
  1. La publication aboutit et un identifiant de wallet est affiché
  2. Les deux identifiants sont **différents** : un wallet dédié a été créé pour le second projet
  3. Un message de confirmation s'affiche, l'IBAN virtuel et le BIC apparaissent sur la fiche, le bouton disparaît
  4. Le bouton "Générer l'IBAN virtuel" est **absent** (IBAN déjà existant), l'IBAN et le BIC sont affichés
  5. La section PSP reste lisible : blocs empilés, identifiants et IBAN non tronqués, aucun débordement horizontal

### Scénario 3 — Souscription non avertie : séquestre puis restitution en rétractation (Critique)
- **Rôle / device** : `investisseur-nonaverti` (**mobile** à l'étape 1) puis `admin-recette`, desktop
- **Pré-conditions** : `projet-souscription-A` publié, collecte ouverte ; `investisseur-nonaverti` KYC validé, statut non averti, solde suffisant ; enchaîner les étapes 1 à 4 **sans attendre l'expiration de la rétractation**
- **Étapes** :
  1. Depuis le FO mobile (375 px), souscrire et payer sur `projet-souscription-A`
  2. En BO, ouvrir la fiche de cette souscription et dérouler jusqu'au bloc "Wallet escrow", puis cliquer sur "Effacer le cache"
  3. Depuis l'espace investisseur, annuler la souscription
  4. En BO, rouvrir la fiche de la souscription, bloc "Wallet escrow", puis "Effacer le cache"
- **Résultats attendus** :
  1. Le tunnel de souscription est utilisable sur mobile (champs et CTA accessibles sans zoom), le paiement est confirmé
  2. Le bloc "Wallet escrow" est visible avec un identifiant de compte et un solde égal au montant investi ; le rafraîchissement recharge la même fiche sans redirection ni changement de montant
  3. L'annulation aboutit sans erreur, la souscription passe au statut "Annulée"
  4. Le solde du wallet escrow affiche **0 €**

### Scénario 4 — Archivage d'un projet : wallet conservé, collecte fermée (Critique)
- **Rôle / device** : `admin-recette`, desktop
- **Pré-conditions** : `porteur-archive` avec un projet publié — fixture exclusive, le projet est muté de façon irréversible ; flag `multi_wallets` actif
- **Étapes** :
  1. Ouvrir la fiche du projet de `porteur-archive` et noter l'identifiant du wallet de collecte
  2. Archiver le projet
  3. Rouvrir la fiche du projet archivé
  4. Depuis le FO, accéder à l'URL publique de ce projet
- **Résultats attendus** :
  1. L'identifiant du wallet de collecte est affiché
  2. L'archivage aboutit sans erreur, le statut passe à "Archivé"
  3. Le bloc wallet est toujours présent avec le **même identifiant** (le wallet n'est pas supprimé) et la collecte est marquée fermée
  4. La page renvoie un 404 ou une redirection vers la liste des projets, jamais une 500

### Scénario 5 — Résilience : timeout PSP et KYC manquant (Majeure)
- **Rôle / device** : `admin-recette` puis `investisseur-sansko`, desktop
- **Pré-conditions** : `porteur-timeout` avec un projet prêt à publier ; moyen de simuler un timeout PSP en recette (coupure réseau ou sandbox indisponible) ; `investisseur-sansko` KYC non validé ; `projet-souscription-A` publié
- **Étapes** :
  1. Publier le projet de `porteur-timeout` en simulant un timeout PSP au moment de la création du wallet
  2. Rétablir le PSP, rouvrir la fiche du projet, puis "Effacer le cache"
  3. Avec `investisseur-sansko`, tenter de souscrire sur `projet-souscription-A`
- **Résultats attendus** :
  1. Un message d'erreur explicite est affiché (pas de page 500, pas de succès silencieux) et le projet n'est pas laissé publié sans wallet
  2. La fiche affiche soit le wallet correctement créé, soit une invitation claire à relancer l'opération — jamais un bloc vide sans explication
  3. La souscription est refusée avec un message renvoyant vers la finalisation du KYC, aucun wallet séquestre n'est créé

### Scénario 6 — Régression : souscription d'un investisseur averti (Critique)
- **Rôle / device** : `investisseur-averti` puis `admin-recette`, desktop
- **Pré-conditions** : `projet-souscription-B` publié, collecte ouverte ; `investisseur-averti` KYC validé, statut averti, solde suffisant ; flag `multi_wallets` actif
- **Étapes** :
  1. Souscrire et payer sur `projet-souscription-B`
  2. En BO, ouvrir la fiche de la souscription
- **Résultats attendus** :
  1. Le paiement aboutit selon l'ancien parcours averti (aucune période de rétractation appliquée)
  2. Les fonds apparaissent sur le wallet de collecte du projet et **aucun** bloc "Wallet escrow" n'est affiché — comportement inchangé par la feature

### Scénario 7 — Feature flag `multi_wallets` désactivé (Majeure)
- **Rôle / device** : `admin-recette`, desktop
- **Pré-conditions** : flag `multi_wallets` **désactivé** ; `projet-flag-off` publié avec une souscription payée
- **Étapes** :
  1. Ouvrir la fiche de `projet-flag-off`
  2. Ouvrir la fiche de sa souscription payée
- **Résultats attendus** :
  1. Aucun bloc multi-wallets n'est affiché, la fiche s'affiche normalement (jamais de 500)
  2. Aucun bloc "Wallet escrow", la fiche reste fonctionnelle

## Points de vigilance
- Les 2 `Psp::TimeoutError` du monitoring proviennent d'une absence de retry sur la création de wallet — le scénario 5 vérifie le message d'erreur, pas la reprise automatique, qui n'existe pas côté code.
- L'identifiant de compte PSP devrait être un lien vers le dashboard du prestataire (mentionné au brief) — non implémenté au commit audité, non testable.
- La cohérence des soldes affichés dépend du cache : un écart transitoire après une opération PSP n'est pas un bug tant que "Effacer le cache" le résorbe.
- Écart mineur brief ↔ code non remonté en question : le brief parle de "wallet de remboursement", le BO affiche "Wallet porteur (remboursements d'échéances)". Vocabulaire à aligner, sans impact fonctionnel.

## Tableau de couverture

| Contrainte | Scénario(s) associé(s) | Case traitée (✓/✗) | Critère d'acceptation |
|---|---|---|---|
| Le premier projet d'un porteur réutilise son wallet par défaut | 1 | ✓ | Identifiant du wallet de collecte identique à celui de la fiche porteur |
| Chaque projet suivant obtient un wallet distinct | 2 | ✓ | Identifiants de collecte différents entre les deux projets du porteur |
| L'IBAN virtuel n'est générable qu'une fois | 2 | ✓ | Bouton absent dès qu'un IBAN existe |
| Un paiement d'investisseur non averti crée un wallet séquestre | 3 | ✓ | Bloc "Wallet escrow" présent, solde = montant investi |
| L'annulation en rétractation restitue les fonds | 3 | ✓ | Solde escrow à 0 € après annulation |
| L'archivage conserve le wallet et ferme la collecte | 4 | ✓ | Même identifiant après archivage, collecte fermée, URL publique en 404 |
| Un timeout PSP ne laisse pas un projet publié sans wallet | 5 | ✓ | Message d'erreur explicite, aucun état incohérent en BO |
| Un KYC non validé bloque la souscription | 5 | ✓ | Refus avec message KYC, aucun wallet séquestre créé |
| Le parcours averti est inchangé | 6 | ✓ | Aucun bloc escrow, paiement direct sur le wallet de collecte |
| Flag désactivé : aucune régression d'affichage | 7 | ✓ | Fiches projet et souscription fonctionnelles, aucun bloc multi-wallets, aucune 500 |
| Sections PSP lisibles sur mobile | 2, 3 | ✓ | Aucun débordement horizontal à 375 px, identifiants non tronqués |
| Retry automatique après timeout PSP | — | ✗ | Non implémenté au commit audité — voir points de vigilance |
