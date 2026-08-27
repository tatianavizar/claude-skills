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

Toutes les entités citées en pré-conditions figurent ici : comptes, mais aussi projets, enveloppes, réseaux, comptes externes. Réutiliser par défaut — une entrée dédiée uniquement si un scénario la mute de façon destructive.

- `Muté par` : le scénario unique qui modifie l'état de façon destructive, ou `—`.
- `Préparation` : `existant en staging` | `BO` | `seed dev`.

| Identifiant | Rôle / état | Muté par | Préparation | Détail |
|---|---|---|---|---|
| | | | | |

## Scénarios

*Tous les scénarios de cette section sont jouables sans intervention d'un développeur. Ceux qui en demandent une sont regroupés plus bas.*

### Scénario 1 — {Titre} ({Critique | Majeure | Mineure})
- **Rôle / device** : {ex. investisseur KYC validé, mobile}
- **Exécutable par** : {PM seul | PM + accès BO}
- **Pré-conditions** : {état des données, feature flags, conditions du rôle — aucun conditionnel}
- **Étapes** :
  1. {Action}
  2. {Action}
- **Résultats attendus** :
  1. {État observable après l'étape 1 — pas une reformulation de l'action}
  2. {État observable après l'étape 2}

### Scénario 2 — {Titre} ({Critique | Majeure | Mineure})
{même structure}

### Scénario de régression — {Titre} ({criticité})
{même structure — ou, si feature entièrement nouvelle : "Pas de régression pertinente — feature entièrement nouvelle." Jamais les deux.}

## À faire avec un dev

*Scénarios qui demandent une manipulation hors interface : activer ou couper un flag, simuler une panne de service externe, interrompre un traitement de fond, soumettre une action depuis un rôle non autorisé. Les écrire ici plutôt que de les omettre : la section devient une demande d'outillage, et ce qui n'est pas joué reste visible.*

### {Titre} ({criticité})
- **Exécutable par** : avec un dev
- **Ce qu'il faut obtenir de l'équipe** : {toggle admin pour le flag, moyen de couper le service, compte de test, accès console...}
- **Pré-conditions** :
- **Étapes** :
  1.
- **Résultats attendus** :
  1.

{Scénario feature flag désactivé — obligatoire si la feature est derrière un flag : accès à chaque URL concernée, une entrée de résultat par URL, 404 ou redirection, jamais une 500. Si aucun flag : "Pas de feature flag sur cette feature."}

## Points de vigilance
- {Point — formulé en effet observable, sans nom de classe ni d'exception}

## Tableau de couverture

| Contrainte | Scénario(s) associé(s) | Case traitée (✓/✗) | Critère d'acceptation |
|---|---|---|---|
| | | | |

## Annexe technique (équipe back)

*Références de code, classes d'exception, noms de jobs justifiant les points de vigilance. Aucun de ces éléments n'apparaît dans le corps du fichier.*

- {Point de vigilance associé} → {référence technique}
