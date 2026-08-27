# Exemple de sortie conforme

Exemple complet de fichier généré par cette skill, à consulter en cas de doute sur la granularité attendue. Feature fictive mais réaliste : un wallet distinct par projet sur une plateforme de financement participatif (BO admin + espace investisseur, prestataire de paiement externe).

Ce que l'exemple illustre :

- **Noms lisibles** ("Porteur A", "Projet en collecte") — aucun identifiant technique, aucun accent grave dans les étapes. Un seul admin dans le fichier, donc "l'admin" sans suffixe.
- **8 entrées de données de test pour 5 scénarios.** Une seule entrée dédiée, pour la seule mutation destructive du fichier (un archivage définitif).
- **Section monitoring en propre**, avec effet observable, date, occurrences et traitement.
- **Un vrai scénario de régression** : l'investisseur averti souscrit *sans passer par la nouvelle feature*, et son parcours est inchangé. Pas l'intégration entre deux parties nouvelles.
- **Les contraintes non jouables en recette** (panne du prestataire, feature désactivée) sortent des scénarios et vont dans leur table, avec le verdict de couverture automatisée.
- **Traçabilité** : chaque ligne ✓ cite le résultat attendu numéroté qui l'assère ; les autres sont ✗ avec leur raison.

---

# Scénarios de test — Un wallet par projet

## Contexte
- Feature : un wallet distinct par projet publié (au lieu d'un wallet unique par porteur), + wallet séquestre par souscription d'investisseur non averti
- Brief utilisé : brief "Multi-wallets" — version éditée du 12/06, section "Réutilisation du wallet par défaut" ajoutée après le round 1
- Ticket(s) : CAPS-1428, CAPS-1431
- Repo / commit : `app-plateforme` @ `a3f19c2`
- Mode : Standard (brief + code)

## Erreurs de monitoring liées à la feature

| Effet observé | Date | Occurrences | Traitement |
|---|---|---|---|
| La publication d'un projet a échoué parce que le prestataire de paiement n'a pas répondu à la création du wallet | 04/06 et 19/06 | 2 en 30 jours | Non reproductible sans couper le prestataire → contrainte non jouable en recette + point de vigilance 1 |

## Données de test à préparer

- `Muté par` : le scénario qui modifie l'état de façon **destructive**. `—` sinon ; les mutations additives sont décrites dans le détail.
- `Préparation` : `existant en staging` | `à créer au BO` | `à créer au BO à chaque run` (non réutilisable). Toutes réalisables par le CDP — un état non créable au BO bloque le scénario et se signale, il ne se maquille pas en instruction de préparation.

| Nom | Rôle / état | Muté par | Préparation | Détail |
|---|---|---|---|---|
| L'admin | Admin BO | — | existant en staging | Droits complets, aucun état propre modifié |
| Porteur A | Porteur de projet | — | à créer au BO | Onboarding paiement complété, **aucun** projet publié, pas d'IBAN virtuel. Gagne deux projets et un IBAN au scénario 1 — additif, aucun scénario n'assère un décompte de projets |
| Porteur avec IBAN | Porteur de projet | — | existant en staging | IBAN virtuel **déjà généré** — consulté en lecture seule au scénario 1 |
| Porteur à archiver | Porteur de projet | sc3 | à créer au BO | Un projet publié, archivé définitivement au scénario 3 |
| Investisseur non averti | Investisseur | — | existant en staging | KYC validé, statut **non averti**, solde suffisant. Souscrit puis annule au scénario 2 — l'état revient à son point de départ |
| Investisseur averti | Investisseur | — | existant en staging | KYC validé, statut **averti**, solde suffisant. Souscrit au scénario 5 — additif |
| Investisseur sans KYC | Investisseur | — | à créer au BO | KYC **non validé**. Sa tentative de souscription au scénario 4 est refusée : aucun état créé, réutilisable indéfiniment |
| Projet en collecte | Projet publié | — | à créer au BO | Collecte ouverte. Reçoit les souscriptions des scénarios 2 et 5, et la tentative refusée du scénario 4 — additif |

## Scénarios

*Tous les scénarios de ce fichier sont jouables sans intervention d'un développeur.*

### Scénario 1 — Un wallet par projet et IBAN virtuel du porteur (Majeure)
- **Rôle / device** : l'admin, desktop puis **mobile** à l'étape 7
- **Pré-conditions** : Porteur A sans projet publié et sans IBAN virtuel ; Porteur avec IBAN disponible en lecture
- **Étapes** :
  1. Ouvrir la fiche de Porteur A et noter l'identifiant de son wallet par défaut
  2. Créer et publier un premier projet pour ce porteur, puis ouvrir la fiche du projet
  3. Créer et publier un second projet pour le même porteur, puis noter l'identifiant de son wallet de collecte
  4. Comparer les identifiants de collecte des deux projets
  5. Depuis la fiche de Porteur A, cliquer sur "Générer l'IBAN virtuel"
  6. Ouvrir la fiche de Porteur avec IBAN
  7. Rouvrir la fiche de Porteur A sur mobile (viewport 375 px)
- **Résultats attendus** :
  1. Un identifiant de wallet par défaut est affiché, avec son solde en euros
  2. Deux blocs sont présents — "Wallet projet (collecte)" et "Wallet porteur (remboursements)" — et l'identifiant du bloc collecte est **identique** à celui noté à l'étape 1 : le wallet existant a été réutilisé
  3. La publication aboutit sans message d'erreur et un identifiant de wallet est affiché
  4. Les deux identifiants sont **différents** : un wallet dédié a été créé pour le second projet
  5. Un message de confirmation s'affiche, l'IBAN virtuel et le BIC apparaissent sur la fiche, le bouton disparaît
  6. Le bouton "Générer l'IBAN virtuel" est **absent** (un IBAN existe déjà), l'IBAN et le BIC sont affichés
  7. La section paiement reste lisible : blocs empilés, identifiants et IBAN non tronqués, aucun débordement horizontal

### Scénario 2 — Souscription non avertie : séquestre puis restitution en rétractation (Critique)
- **Rôle / device** : Investisseur non averti (**mobile** à l'étape 1) puis l'admin, desktop
- **Pré-conditions** : Projet en collecte ouverte ; Investisseur non averti avec KYC validé et solde suffisant ; enchaîner les étapes 1 à 4 **sans attendre l'expiration du délai de rétractation**
- **Étapes** :
  1. Depuis le front mobile (375 px), souscrire et payer sur Projet en collecte
  2. En BO, ouvrir la fiche de cette souscription, dérouler jusqu'au bloc "Wallet séquestre", puis cliquer sur "Rafraîchir"
  3. Depuis l'espace investisseur, annuler la souscription
  4. En BO, rouvrir la fiche de la souscription, bloc "Wallet séquestre", puis "Rafraîchir"
- **Résultats attendus** :
  1. Le tunnel de souscription est utilisable sur mobile (champs et bouton accessibles sans zoom), le paiement est confirmé
  2. Le bloc "Wallet séquestre" affiche un identifiant de compte et un solde égal au montant investi ; le rafraîchissement recharge la même fiche sans redirection ni changement de montant
  3. L'annulation aboutit sans erreur, la souscription passe au statut "Annulée"
  4. Le solde du wallet séquestre affiche **0 €**

### Scénario 3 — Archivage d'un projet : wallet conservé, collecte fermée (Critique)
- **Rôle / device** : l'admin, desktop
- **Pré-conditions** : Porteur à archiver avec un projet publié — entrée dédiée, l'archivage est irréversible
- **Étapes** :
  1. Ouvrir la fiche du projet de Porteur à archiver et noter l'identifiant du wallet de collecte
  2. Archiver le projet
  3. Rouvrir la fiche du projet archivé
  4. Depuis le front, accéder à l'adresse publique de ce projet
- **Résultats attendus** :
  1. L'identifiant du wallet de collecte est affiché
  2. L'archivage aboutit sans erreur, le statut passe à "Archivé"
  3. Le bloc wallet est toujours présent avec le **même identifiant** — le wallet n'est pas supprimé — et la collecte est marquée fermée
  4. La page affiche "projet introuvable" ou redirige vers la liste des projets, jamais une erreur serveur

### Scénario 4 — KYC non validé : souscription refusée (Majeure)
- **Rôle / device** : Investisseur sans KYC, desktop
- **Pré-conditions** : Investisseur sans KYC validé ; Projet en collecte ouverte
- **Étapes** :
  1. Depuis le front, ouvrir Projet en collecte et lancer une souscription
  2. En BO, ouvrir la fiche de Investisseur sans KYC
- **Résultats attendus** :
  1. La souscription est refusée avec un message renvoyant vers la finalisation du KYC ; aucun formulaire de paiement n'est proposé
  2. Aucune souscription et aucun wallet séquestre ne sont rattachés au compte

### Scénario 5 — Régression : l'investisseur averti souscrit seul, parcours inchangé (Critique)
- **Rôle / device** : Investisseur averti puis l'admin, desktop
- **Pré-conditions** : Projet en collecte ouverte ; Investisseur averti avec KYC validé et solde suffisant. Ce scénario vérifie le parcours de souscription **tel qu'il existait avant la feature** — l'investisseur agit de lui-même, sans intervention d'un tiers
- **Étapes** :
  1. Se connecter à l'espace investisseur et observer le tableau de bord
  2. Souscrire et payer sur Projet en collecte
  3. Consulter le récapitulatif de la souscription depuis l'espace investisseur
  4. En BO, ouvrir la fiche de la souscription
- **Résultats attendus** :
  1. Le tableau de bord affiche les blocs habituels, sans card ni encart nouveau lié aux wallets ou à une réservation
  2. Le paiement aboutit selon le parcours averti existant, aucun délai de rétractation n'est appliqué, l'email de confirmation habituel est reçu
  3. Le récapitulatif affiche les mêmes informations qu'avant la feature : montant, projet, statut — aucune mention de wallet
  4. Les fonds apparaissent sur le wallet de collecte du projet et **aucun** bloc "Wallet séquestre" n'est affiché

## Points de vigilance
- La création du wallet d'un projet n'est pas relancée automatiquement quand le prestataire de paiement ne répond pas : la publication échoue et doit être relancée à la main. Deux occurrences en 30 jours (voir section monitoring).
- L'identifiant de compte du prestataire devrait être un lien vers son tableau de bord (prévu au brief) — non implémenté sur la version auditée, non testable.
- Les soldes affichés viennent d'un cache : un écart transitoire après une opération de paiement n'est pas un bug tant que le bouton "Rafraîchir" le résorbe.
- Écart de vocabulaire brief ↔ interface, sans impact fonctionnel : le brief parle de "wallet de remboursement", le BO affiche "Wallet porteur (remboursements)".

## Tableau de couverture

| Contrainte | Scénario(s) | ✓/✗ | Critère d'acceptation |
|---|---|---|---|
| Le premier projet d'un porteur réutilise son wallet par défaut | sc1 | ✓ | Identifiant de collecte identique à celui de la fiche porteur (sc1 rés. 2) |
| Chaque projet suivant obtient un wallet distinct | sc1 | ✓ | Identifiants de collecte différents entre les deux projets (sc1 rés. 4) |
| L'IBAN virtuel n'est générable qu'une fois | sc1 | ✓ | IBAN affiché et bouton disparu (sc1 rés. 5), bouton absent si un IBAN existe (sc1 rés. 6) |
| Les blocs de paiement restent lisibles sur mobile | sc1, sc2 | ✓ | Aucun débordement horizontal à 375 px (sc1 rés. 7), tunnel utilisable sur mobile (sc2 rés. 1) |
| Un paiement d'investisseur non averti crée un wallet séquestre | sc2 | ✓ | Solde du séquestre égal au montant investi (sc2 rés. 2) |
| L'annulation pendant la rétractation restitue les fonds | sc2 | ✓ | Solde du séquestre à 0 € (sc2 rés. 4) |
| L'archivage conserve le wallet et ferme la collecte | sc3 | ✓ | Même identifiant après archivage, collecte fermée (sc3 rés. 3), adresse publique inaccessible (sc3 rés. 4) |
| Un KYC non validé bloque la souscription | sc4 | ✓ | Refus avec message KYC (sc4 rés. 1), aucun wallet séquestre créé (sc4 rés. 2) |
| Le parcours de souscription averti est inchangé | sc5 | ✓ | Aucun encart nouveau au tableau de bord (sc5 rés. 1), pas de rétractation appliquée et email habituel reçu (sc5 rés. 2), aucun bloc séquestre (sc5 rés. 4) |
| Une indisponibilité du prestataire ne laisse pas un projet publié sans wallet | — | ✗ | Non jouable en recette et non couvert par un test automatisé — voir contraintes non jouables |
| Feature désactivée : aucune régression d'affichage | — | ✗ | Couvert par un test automatisé — voir contraintes non jouables |
| Reprise automatique après indisponibilité du prestataire | — | ✗ | Non implémenté sur la version auditée — voir point de vigilance 1 |

## Contraintes non jouables en recette manuelle

| Contrainte | Couverte par un test automatisé | Suite à donner |
|---|---|---|
| Une indisponibilité du prestataire à la création du wallet laisse un message d'erreur explicite et aucun projet publié sans wallet | Non | **Demander un test à l'équipe** — c'est l'erreur la plus fréquente du monitoring et rien ne la couvre |
| Multi-wallets désactivé : les fiches projet et souscription restent fonctionnelles, sans bloc wallet | Oui — référence en annexe | Rien à faire |

*Scénarios détaillés pour ces deux contraintes disponibles sur demande, à jouer avec un développeur.*

## Annexe technique (équipe back)

*Aucun de ces éléments n'apparaît dans le corps du fichier.*

- Indisponibilité du prestataire (section monitoring, point de vigilance 1) : `Psp::TimeoutError` levée dans la création du wallet projet, 2 occurrences sur 30 jours, aucune relance configurée. Aucun test ne couvre ce chemin d'erreur.
- Feature désactivée : couvert par `spec/features/projects/multi_wallets_disabled_spec.rb`.
