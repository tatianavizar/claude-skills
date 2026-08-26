# Scénarios de test — {Nom de la feature}

> ⚠️ **Mode dégradé** — {bloc à supprimer si le repo était accessible} Scénarios générés sans accès au code : comportement réel du repo non vérifié, à confirmer manuellement avant recette.

## Contexte
- Feature :
- Brief utilisé (version initiale / éditée) :
- Ticket(s) :
- Repo / commit :
- Mode : Standard (brief + code) | Dégradé (brief + tickets uniquement)
- Monitoring : vérifié, aucune erreur récente | non accessible | erreurs trouvées (voir {scénario | point de vigilance})

## Données de test à préparer

Toutes les entités citées en pré-conditions figurent ici : rôles, mais aussi projets, enveloppes, réseaux, comptes externes. `Muté par` = le scénario unique qui modifie l'état de l'entité, ou `—` si l'entité est en lecture seule sur tout le fichier.

| Identifiant | Rôle / état | Muté par | Détail |
|---|---|---|---|
| | | | |

## Scénarios

### Scénario 1 — {Titre} ({Critique | Majeure | Mineure})
- **Rôle / device** : {ex. investisseur KYC validé, mobile}
- **Pré-conditions** : {état des données, feature flags, conditions du rôle}
- **Étapes** :
  1. {Action}
  2. {Action}
- **Résultats attendus** :
  1. {État observable après l'étape 1 — pas une reformulation de l'action}
  2. {État observable après l'étape 2}

### Scénario 2 — {Titre} ({Critique | Majeure | Mineure})
{même structure}

### Scénario de régression — {Titre} ({criticité})
{même structure — ou, si feature entièrement nouvelle : "Pas de régression pertinente — feature entièrement nouvelle."}

### Scénario feature flag désactivé — {Titre} ({criticité})
- **Rôle / device** :
- **Pré-conditions** : feature flag désactivé
- **Étapes** :
  1. {Accès à chaque URL concernée}
- **Résultats attendus** :
  1. {404 ou redirection, jamais une 500}

{Si la feature n'est pas derrière un flag : "Pas de feature flag sur cette feature."}

## Points de vigilance
- {Point}

## Tableau de couverture

| Contrainte | Scénario(s) associé(s) | Case traitée (✓/✗) | Critère d'acceptation |
|---|---|---|---|
| | | | |
