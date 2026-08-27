# Scénarios de test — {Nom de la feature}

> ⚠️ **Mode dégradé sans code** — {bloc à supprimer si le repo était accessible} Scénarios générés sans accès au code : comportement réel non vérifié, à confirmer manuellement avant recette.

> 🚨 **Mode dégradé sans attendu** — {bloc à supprimer si le brief était accessible} Aucun brief ni critère d'acceptation disponible : les scénarios décrivent ce que le code **fait**, pas ce qu'il **devrait** faire. Un bug présent dans le code apparaît donc ici comme un résultat attendu. Les règles métier listées en fin de fichier doivent être confirmées par le CDP avant toute recette.

## Contexte
- Feature :
- Brief utilisé (version initiale / éditée, ou source si non versionné) :
- Ticket(s) :
- Repo / commit :
- Mode : Standard (brief + code) | Dégradé sans code (brief + tickets seuls) | Dégradé sans attendu (code seul)
- Divergences code ↔ brief : {N détectées, tranchées au round 1 | aucune divergence non documentée — comparaison faite}

## Erreurs de monitoring liées à la feature

*Section toujours présente. Si rien : "Aucune erreur récente liée à la feature" ou "Monitoring non accessible". Formuler l'effet observable, pas la trace technique — celle-ci va en annexe.*

| Effet observé | Date | Occurrences | Traitement |
|---|---|---|---|
| {ce qui a échoué, en langage d'utilisateur} | | | {scénario N | point de vigilance N | hors périmètre : préciser} |

## Données de test à préparer

*Noms lisibles ("Investisseur A", "Projet A"), pas d'identifiant technique. Toutes les entités citées en pré-conditions figurent ici : comptes, projets, enveloppes, réseaux. Réutiliser par défaut — une entrée dédiée uniquement si un scénario la mute de façon destructive.*

- `Muté par` : le scénario unique qui modifie l'état de façon destructive, ou `—` (les mutations additives sont décrites dans le détail).
- `Préparation` : `existant en staging` | `à créer au BO` | `à créer au BO à chaque run` (non réutilisable). Toutes réalisables par le CDP — un état non créable au BO bloque le scénario et se signale, il ne se maquille pas en instruction de préparation.

| Nom | Rôle / état | Muté par | Préparation | Détail |
|---|---|---|---|---|
| | | | | |

## Scénarios

*Tous les scénarios de ce fichier sont jouables sans intervention d'un développeur.*

### Scénario 1 — {Titre} ({Critique | Majeure | Mineure})
- **Rôle / device** : {ex. investisseur KYC validé, mobile}
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
{Vérifie qu'un parcours pré-existant, déclenché SANS passer par la nouvelle feature, est inchangé. Même structure.}

{S'il n'y a réellement aucune régression, remplacer ce scénario par le constat de la revue des surfaces partagées : "Aucune régression : la feature n'ajoute ni redirection après connexion, ni bloc sur un écran existant, ni email sur un déclencheur existant." Jamais les deux formes.}

## Points de vigilance
- {Point — formulé en effet observable, sans nom de classe ni d'exception}

## Tableau de couverture

*`✓` = un résultat attendu de ce fichier assère la contrainte. Sinon `✗`, avec la raison en critère d'acceptation.*

| Contrainte | Scénario(s) | ✓/✗ | Critère d'acceptation |
|---|---|---|---|
| | | | |

## Contraintes non jouables en recette manuelle

*Feature flag, panne de service, traitement de fond interrompu, soumission depuis un rôle non autorisé. Pour chacune : est-elle couverte par un test automatisé du repo ? Ces lignes figurent aussi dans le tableau de couverture, en `✗`.*

| Contrainte | Couverte par un test automatisé | Suite à donner |
|---|---|---|
| | {oui — référence en annexe | non} | {rien à faire | demander un test à l'équipe | scénario dev sur demande} |

*Proposer au CDP, après remise du fichier, des scénarios détaillés à jouer avec un développeur pour ces contraintes. Si accepté, les livrer dans un fichier séparé `scenarios-{slug-feature}-dev.md`.*

## Annexe technique (équipe back)

*Références de code, classes d'exception, noms de jobs, chemins de tests. Aucun de ces éléments n'apparaît dans le corps du fichier.*

- {Point de vigilance ou contrainte associée} → {référence technique}
