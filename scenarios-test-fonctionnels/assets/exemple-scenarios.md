# Exemple de sortie conforme

Exemple complet de fichier généré par cette skill, à consulter en cas de doute sur la granularité attendue. Feature fictive mais réaliste : gestion multi-wallets sur une plateforme de financement participatif (BO admin + FO investisseur, prestataire de paiement externe).

Ce que l'exemple illustre :

- **8 entrées de données de test pour 7 scénarios**, pas une par scénario. Un seul porteur sert deux scénarios parce que ses mutations sont additives ; un seul projet reçoit deux souscriptions et une tentative refusée.
- **Une seule entrée dédiée** (`porteur-archive`), pour la seule mutation destructive du fichier : un archivage définitif.
- **Champ `Exécutable par`** sur chaque scénario, et les deux scénarios qui demandent un développeur isolés dans leur section.
- **Aucun nom de classe, d'exception ou de job dans le corps** — tout est en annexe technique.
- Criticité dans les titres, listes parallèles où chaque résultat énonce un état observable, ordre métier, mutualisation (le scénario 1 enchaîne trois actions du même rôle), responsive intégré aux scénarios existants.
- **Traçabilité du tableau de couverture** : chaque ligne ✓ renvoie à un résultat attendu numéroté qui assère la contrainte ; la seule contrainte non couverte est ✗ avec renvoi en point de vigilance.

---

# Scénarios de test — Multi-wallets par projet

## Contexte
- Feature : un wallet distinct par projet publié (au lieu d'un wallet unique par porteur), + wallet séquestre par souscription d'investisseur non averti
- Brief utilisé (version initiale / éditée) : brief "Multi-wallets" — version éditée du 12/06, section "Réutilisation du wallet par défaut" ajoutée après le round 1
- Ticket(s) : CAPS-1428, CAPS-1431
- Repo / commit : `app-plateforme` @ `a3f19c2`
- Mode : Standard (brief + code)
- Monitoring : vérifié — la création du wallet d'un projet a échoué 2 fois en 30 jours par absence de réponse du prestataire de paiement (voir scénario D1 et annexe technique)

## Données de test à préparer

- `Muté par` : le scénario qui modifie l'état de façon **destructive**. `—` sinon ; les mutations additives sont indiquées dans le détail.
- `Préparation` : `existant en staging` | `BO` | `seed dev`.

| Identifiant | Rôle / état | Muté par | Préparation | Détail |
|---|---|---|---|---|
| `admin-recette` | Admin BO | — | existant | droits complets, aucun état propre modifié |
| `porteur-actif` | Porteur de projet | — | BO | onboarding paiement complété, **aucun** projet publié, pas d'IBAN virtuel. Gagne des projets et un IBAN en sc1, une tentative de publication en D1 — additif, aucun scénario n'assère un décompte de projets |
| `porteur-iban` | Porteur de projet | — | existant | IBAN virtuel **déjà généré** — consulté en lecture seule en sc1 |
| `porteur-archive` | Porteur de projet | sc3 | BO | 1 projet publié, archivé définitivement en sc3 |
| `investisseur-nonaverti` | Investisseur FO | — | existant | KYC validé, statut **non averti**, solde suffisant. Souscrit puis annule en sc2 |
| `investisseur-averti` | Investisseur FO | — | existant | KYC validé, statut **averti**, solde suffisant. Souscrit en sc5 |
| `investisseur-sanskyc` | Investisseur FO | — | BO | KYC **non validé**. Sa tentative de souscription en sc4 est refusée : aucun état créé, réutilisable indéfiniment |
| `projet-collecte` | Projet publié | — | BO | collecte ouverte. Reçoit la souscription de sc2 puis celle de sc5, et la tentative refusée de sc4 — additif |

*`porteur-actif` et `projet-collecte` remplacent chacun trois entrées d'une première version de ce fichier : gagner un projet ou une souscription est additif, donc aucune duplication n'était nécessaire.*

## Scénarios

*Tous les scénarios de cette section sont jouables sans intervention d'un développeur.*

### Scénario 1 — Wallets par projet et IBAN virtuel du porteur (Majeure)
- **Rôle / device** : `admin-recette`, desktop puis **mobile** à l'étape 7
- **Exécutable par** : CDP + accès BO
- **Pré-conditions** : `porteur-actif` sans projet publié et sans IBAN virtuel ; `porteur-iban` disponible en lecture ; multi-wallets actif sur l'environnement
- **Étapes** :
  1. Ouvrir la fiche de `porteur-actif` et noter l'identifiant de son wallet par défaut
  2. Créer et publier un premier projet pour ce porteur, puis ouvrir la fiche du projet
  3. Créer et publier un second projet pour le même porteur, puis noter l'identifiant de son wallet de collecte
  4. Comparer les identifiants de collecte des deux projets
  5. Depuis la fiche de `porteur-actif`, cliquer sur "Générer l'IBAN virtuel"
  6. Ouvrir la fiche de `porteur-iban`
  7. Rouvrir la fiche de `porteur-actif` sur mobile (viewport 375 px)
- **Résultats attendus** :
  1. Un identifiant de wallet par défaut est affiché, avec son solde en euros
  2. Deux blocs sont présents — "Wallet projet (collecte)" et "Wallet porteur (remboursements)" — et l'identifiant du bloc collecte est **identique** à celui noté à l'étape 1 : le wallet existant a été réutilisé
  3. La publication aboutit sans message d'erreur et un identifiant de wallet est affiché
  4. Les deux identifiants sont **différents** : un wallet dédié a été créé pour le second projet
  5. Un message de confirmation s'affiche, l'IBAN virtuel et le BIC apparaissent sur la fiche, le bouton disparaît
  6. Le bouton "Générer l'IBAN virtuel" est **absent** (un IBAN existe déjà), l'IBAN et le BIC sont affichés
  7. La section paiement reste lisible : blocs empilés, identifiants et IBAN non tronqués, aucun débordement horizontal

### Scénario 2 — Souscription non avertie : séquestre puis restitution en rétractation (Critique)
- **Rôle / device** : `investisseur-nonaverti` (**mobile** à l'étape 1) puis `admin-recette`, desktop
- **Exécutable par** : CDP + accès BO
- **Pré-conditions** : `projet-collecte` en collecte ouverte ; `investisseur-nonaverti` KYC validé, statut non averti, solde suffisant ; enchaîner les étapes 1 à 4 **sans attendre l'expiration du délai de rétractation**
- **Étapes** :
  1. Depuis le front mobile (375 px), souscrire et payer sur `projet-collecte`
  2. En BO, ouvrir la fiche de cette souscription, dérouler jusqu'au bloc "Wallet séquestre", puis cliquer sur "Rafraîchir"
  3. Depuis l'espace investisseur, annuler la souscription
  4. En BO, rouvrir la fiche de la souscription, bloc "Wallet séquestre", puis "Rafraîchir"
- **Résultats attendus** :
  1. Le tunnel de souscription est utilisable sur mobile (champs et bouton accessibles sans zoom), le paiement est confirmé
  2. Le bloc "Wallet séquestre" affiche un identifiant de compte et un solde égal au montant investi ; le rafraîchissement recharge la même fiche sans redirection ni changement de montant
  3. L'annulation aboutit sans erreur, la souscription passe au statut "Annulée"
  4. Le solde du wallet séquestre affiche **0 €**

### Scénario 3 — Archivage d'un projet : wallet conservé, collecte fermée (Critique)
- **Rôle / device** : `admin-recette`, desktop
- **Exécutable par** : CDP + accès BO
- **Pré-conditions** : `porteur-archive` avec un projet publié — entrée dédiée, l'archivage est irréversible ; multi-wallets actif
- **Étapes** :
  1. Ouvrir la fiche du projet de `porteur-archive` et noter l'identifiant du wallet de collecte
  2. Archiver le projet
  3. Rouvrir la fiche du projet archivé
  4. Depuis le front, accéder à l'adresse publique de ce projet
- **Résultats attendus** :
  1. L'identifiant du wallet de collecte est affiché
  2. L'archivage aboutit sans erreur, le statut passe à "Archivé"
  3. Le bloc wallet est toujours présent avec le **même identifiant** — le wallet n'est pas supprimé — et la collecte est marquée fermée
  4. La page affiche "projet introuvable" ou redirige vers la liste des projets, jamais une erreur serveur

### Scénario 4 — KYC non validé : souscription refusée (Majeure)
- **Rôle / device** : `investisseur-sanskyc`, desktop
- **Exécutable par** : CDP seul
- **Pré-conditions** : `investisseur-sanskyc` avec KYC non validé ; `projet-collecte` en collecte ouverte
- **Étapes** :
  1. Depuis le front, ouvrir `projet-collecte` et lancer une souscription
  2. En BO, ouvrir la fiche de `investisseur-sanskyc`
- **Résultats attendus** :
  1. La souscription est refusée avec un message renvoyant vers la finalisation du KYC ; aucun formulaire de paiement n'est proposé
  2. Aucune souscription et aucun wallet séquestre ne sont rattachés au compte

### Scénario 5 — Régression : souscription d'un investisseur averti (Critique)
- **Rôle / device** : `investisseur-averti` puis `admin-recette`, desktop
- **Exécutable par** : CDP + accès BO
- **Pré-conditions** : `projet-collecte` en collecte ouverte ; `investisseur-averti` KYC validé, statut averti, solde suffisant ; multi-wallets actif
- **Étapes** :
  1. Souscrire et payer sur `projet-collecte`
  2. En BO, ouvrir la fiche de la souscription
- **Résultats attendus** :
  1. Le paiement aboutit selon le parcours averti existant, aucun délai de rétractation n'est appliqué
  2. Les fonds apparaissent sur le wallet de collecte du projet et **aucun** bloc "Wallet séquestre" n'est affiché — comportement inchangé par la feature

## À faire avec un dev

*Deux vérifications ne sont pas déclenchables depuis l'interface. Elles sont écrites ici pour rester visibles et devenir une demande d'outillage, pas pour être omises.*

### D1 — Résilience : le prestataire de paiement ne répond pas à la création du wallet (Majeure)
- **Exécutable par** : avec un dev
- **Ce qu'il faut obtenir de l'équipe** : un moyen de rendre le prestataire injoignable sur l'environnement de recette (bac à sable coupé ou blocage réseau), et de le rétablir
- **Pré-conditions** : `porteur-actif` avec un projet prêt à publier ; prestataire rendu injoignable
- **Étapes** :
  1. Publier le projet pendant que le prestataire est injoignable
  2. Rétablir le prestataire, rouvrir la fiche du projet, puis cliquer sur "Rafraîchir"
- **Résultats attendus** :
  1. Un message d'erreur explicite est affiché, aucune page d'erreur serveur, et le projet n'est pas laissé publié sans wallet
  2. La fiche affiche soit le wallet correctement créé, soit une invitation claire à relancer l'opération — jamais un bloc vide sans explication

### D2 — Multi-wallets désactivé (Majeure)
- **Exécutable par** : avec un dev
- **Ce qu'il faut obtenir de l'équipe** : un interrupteur d'activation dans le BO, ou la désactivation à la demande sur l'environnement de recette
- **Pré-conditions** : multi-wallets **désactivé** ; un projet publié portant une souscription payée (`projet-collecte` après le scénario 5, ou tout projet équivalent existant en staging)
- **Étapes** :
  1. Ouvrir la fiche du projet
  2. Ouvrir la fiche de sa souscription payée
- **Résultats attendus** :
  1. Aucun bloc multi-wallets n'est affiché, la fiche s'affiche normalement, aucune erreur serveur
  2. Aucun bloc "Wallet séquestre", la fiche reste fonctionnelle

## Points de vigilance
- La création du wallet d'un projet n'est pas relancée automatiquement quand le prestataire ne répond pas : le scénario D1 vérifie le message affiché, pas une reprise, qui n'existe pas.
- L'identifiant de compte du prestataire devrait être un lien vers son tableau de bord (prévu au brief) — non implémenté sur la version auditée, non testable.
- Les soldes affichés viennent d'un cache : un écart transitoire après une opération de paiement n'est pas un bug tant que le bouton "Rafraîchir" le résorbe.
- Écart de vocabulaire brief ↔ interface, sans impact fonctionnel : le brief parle de "wallet de remboursement", le BO affiche "Wallet porteur (remboursements)".

## Tableau de couverture

| Contrainte | Scénario(s) | ✓/✗ | Critère d'acceptation |
|---|---|---|---|
| Le premier projet d'un porteur réutilise son wallet par défaut | sc1 | ✓ | Identifiant du wallet de collecte identique à celui de la fiche porteur (résultat 2) |
| Chaque projet suivant obtient un wallet distinct | sc1 | ✓ | Identifiants de collecte différents entre les deux projets (résultat 4) |
| L'IBAN virtuel n'est générable qu'une fois | sc1 | ✓ | IBAN affiché et bouton disparu après génération (résultat 5), bouton absent quand un IBAN existe (résultat 6) |
| Un paiement d'investisseur non averti crée un wallet séquestre | sc2 | ✓ | Bloc "Wallet séquestre" avec solde égal au montant investi (résultat 2) |
| L'annulation pendant la rétractation restitue les fonds | sc2 | ✓ | Solde du wallet séquestre à 0 € (résultat 4) |
| L'archivage conserve le wallet et ferme la collecte | sc3 | ✓ | Même identifiant de wallet après archivage, collecte fermée (résultat 3), adresse publique inaccessible (résultat 4) |
| Un KYC non validé bloque la souscription | sc4 | ✓ | Refus avec message KYC (résultat 1), aucun wallet séquestre créé (résultat 2) |
| Le parcours averti est inchangé | sc5 | ✓ | Aucun bloc séquestre, fonds sur le wallet de collecte (résultat 2) |
| Une indisponibilité du prestataire ne laisse pas un projet publié sans wallet | D1 | ✓ | Message d'erreur explicite, aucun état incohérent en BO (résultat 1) |
| Multi-wallets désactivé : aucune régression d'affichage | D2 | ✓ | Fiches projet et souscription fonctionnelles, aucun bloc, aucune erreur serveur (résultats 1 et 2) |
| Sections paiement lisibles sur mobile | sc1, sc2 | ✓ | Aucun débordement horizontal à 375 px (sc1 résultat 7), tunnel utilisable sur mobile (sc2 résultat 1) |
| Reprise automatique après indisponibilité du prestataire | — | ✗ | Non implémenté sur la version auditée — voir points de vigilance |

## Annexe technique (équipe back)

*Aucun de ces éléments n'apparaît dans le corps du fichier.*

- Indisponibilité du prestataire (contexte, scénario D1, point de vigilance 1) : `Psp::TimeoutError` levée dans la création du wallet projet, 2 occurrences sur 30 jours, aucune relance configurée.
