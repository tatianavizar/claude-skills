---
name: scenarios-test-fonctionnels
description: Génère des scénarios de test fonctionnels end-to-end pour une feature, en croisant le code du repo GitHub de l'app (comportement réel) avec le brief fonctionnel et les tickets (comportement attendu). Utiliser dès qu'un CDP demande de "créer des scénarios de test", "scénarios de recette", "tester cette feature", "couvrir fonctionnellement", "vérifier les cas limites", "tableau de couverture" — mais aussi pour des formulations indirectes comme "qu'est-ce qu'on doit vérifier avant de livrer", "comment on valide que ça marche", "on est prêts pour la recette ?", même sans le mot "scénario". Déclencher aussi en anglais ("test scenarios", "QA plan", "acceptance testing", "how do we test this", "coverage table"), ou si la personne partage un lien de feature/brief/ticket en demandant comment le tester.
license: Propriétaire — Tatiana Villamizar Mojica (Capsens). Usage interne.
allowed-tools: Read, Grep, Glob, Write, WebFetch, Bash(git log:*), Bash(git show:*), Bash(git diff:*), mcp__*, mcp__claude_ai_Slite__*, mcp__claude_ai_Trello__*, mcp__claude_ai_Linear__*, mcp__claude_ai_Sentry__*
---

# Génération de scénarios de test fonctionnels

## Principe

Le **code applicatif** fait foi pour "ce qui existe réellement". Le **brief + les tickets** font foi pour "ce qui est attendu". Les questions posées pendant le workflow servent uniquement à faire trancher des **divergences non documentées** entre l'attendu et le réel — jamais des questions d'opinion ou de conception sur la feature elle-même.

## Fichiers de cette skill

| Fichier | Quand le lire |
|---|---|
| `references/regles-redaction.md` | **Obligatoire** avant l'étape 7 — règles de rédaction détaillées et leurs pièges |
| `assets/template-scenarios.md` | **Obligatoire** avant l'étape 7 — structure du fichier de sortie |
| `assets/exemple-scenarios.md` | Avant l'étape 7 si le niveau de granularité attendu n'est pas clair |

## Inputs requis

- Nom / périmètre de la feature.
- Brief fonctionnel : version initiale **et** version éditée.
- Ticket(s) lié(s).
- Repo GitHub de l'app (accès déjà configuré en amont — cette skill n'a pas à le gérer ni à le demander).
- Outil de monitoring d'erreurs si disponible (facultatif mais recommandé).

## Étape 0 — Outillage

Cette skill est partagée entre plusieurs CDP qui n'utilisent pas les mêmes outils. **Détecter d'abord** dans les outils disponibles et dans le contexte de la conversation : gestion de tickets (Trello, Linear, Jira...), source du brief (Slite, Notion, Confluence, fichier...), monitoring (AppSignal, Sentry, Bugsnag...).

Ne demander que ce qui reste ambigu — aucun outil détecté pour une catégorie, ou plusieurs candidats sans moyen de choisir. Regrouper en une seule question. Ne jamais reposer une question déjà répondue. Cette étape ne compte pas dans les rounds de questions.

## Workflow

### 1. Identifier la feature
Si le nom ou le périmètre n'est **pas donné explicitement**, le demander avant toute recherche sur GitHub ou dans les tickets — ne jamais deviner ni partir en recherche exploratoire sur une supposition.

Une fois identifiée : confirmer le parcours exact, les rôles concernés (FO/BO), le ticket, la version du brief.

**Si le repo n'est pas accessible** (droits, connecteur en panne) → **mode dégradé** plutôt que blocage : générer depuis le brief et les tickets seuls, et le signaler en tête du fichier de sortie. Ne jamais présenter des scénarios brief-only comme confrontés au code.

### 2. Extraire le comportement attendu
Brief (initial + édité) + ticket(s) → règles métier, critères d'acceptation, cas limites anticipés, décisions tranchées vs questions ouvertes. Noter si la feature est **un ajout neuf** ou **une modification d'un comportement existant** (sert à l'étape 7 pour la régression).

### 3. Extraire le comportement réel (code)
Parcourir uniquement le code pertinent à la feature (routes/controllers, modèles, composants frontend), ancré sur : nom de la feature, PR/commits du ticket, routes du parcours. Repérer : validations et limites, gestion d'erreur et messages, comportements silencieux, breakpoints/composants responsive si la feature touche le FO.

**Ne pas consulter les specs/tests existants du repo** : cela biaiserait la comparaison brief ↔ code. Seul le code applicatif fait foi pour "ce qui existe".

### 4. Vérifier les erreurs déjà connues
Si un monitoring est accessible, chercher les erreurs récentes liées à la feature. Chaque erreur devient un scénario de résilience (étape 7) ou un point de vigilance (étape 9) si non testable simplement.

**Toujours tracer le résultat dans le contexte du fichier de sortie**, y compris quand rien n'est trouvé ("Monitoring vérifié : aucune erreur récente" / "Monitoring non accessible").

### 5. Détecter les divergences non documentées
Lister ce qui diverge entre attendu (étape 2) et réel (étape 3) sans avoir été tranché quelque part (brief, ticket, commentaire). Écart mineur et non ambigu → note de vigilance. Écart qui change le comportement testable → question de contexte.

### 6. Round 1 — questions (max 5, seulement si nécessaire)
Un seul message, numérotées, chacune avec le contexte minimal (extrait brief/ticket vs extrait code). Attendre la réponse. Aucune divergence non documentée → passer à l'étape 7 et le dire explicitement.

Réponse partielle : traiter les points répondus, et basculer les points restés sans réponse en points de vigilance (étape 9) plutôt que de relancer.

### 7. Rédiger les scénarios

**Lire `references/regles-redaction.md` et `assets/template-scenarios.md` avant de commencer.** Lire aussi `assets/exemple-scenarios.md` en cas de doute sur la granularité.

Règles (détail et pièges dans `references/regles-redaction.md`) :
- Chaque scénario est **exécutable seul**, sans dépendre de l'état laissé par un autre.
- Une entité **mutée** par un scénario = une fixture dédiée à ce scénario, sur tout le fichier.
- **Pré-conditions explicites** : rôle et ses conditions précises, feature flags, état des données.
- **Ordre = logique métier du parcours**, jamais la criticité.
- **Criticité dans le titre** entre parenthèses : `Critique` / `Majeure` / `Mineure`.
- **Étapes et résultats attendus en deux listes numérotées parallèles**.
- **Mutualiser les parcours** : le moins de scénarios possible sans perdre de couverture.
- **Scénario de régression** si la feature modifie un comportement existant ; sinon le dire explicitement.
- **Scénario feature flag désactivé** si la feature est derrière un flag ; sinon le dire explicitement.
- **Responsive mutualisé** dans les scénarios existants, jamais dans un scénario dédié.

Couverture attendue : chemin nominal, cas limites (données vides/max, doublons, valeurs invalides, actions concurrentes, permissions insuffisantes), résilience (erreur réseau, timeout, quota, échec silencieux, erreurs de l'étape 4), responsive si la feature touche le FO (mobile + desktop minimum).

Volume indicatif pour calibrer : **6 à 12 scénarios** pour une feature de taille moyenne (un parcours, 2-3 rôles). Sortir de cette fourchette est légitime, mais au-delà de ~15 vérifier d'abord qu'il n'y a pas un découpage à mutualiser.

### 8. Round 2 — questions (max 5, seulement si nécessaire)
Uniquement si la rédaction a fait émerger de **nouvelles** divergences non documentées. Mêmes règles que le round 1.

### 9. Points de vigilance
Hors scénarios formels : ce qui mérite un œil humain en recette mais n'est pas testable simplement par un CDP — dette technique, zone fragile, dépendance externe, erreur de monitoring non reproductible, écart mineur code/brief, question restée sans réponse.

### 10. Tableau de couverture final
Une ligne par contrainte technique ou fonctionnelle relevée en étapes 2 à 4. **Seules les contraintes observables par un CDP** — observable = vérifiable via UI, API, panel admin ou monitoring, sans lire le code. Contrainte vérifiable uniquement dans le code : chercher à la rendre observable autrement (admin, logs, dashboard) et la reformuler en critère d'acceptation ; sinon, ne pas la mettre dans le tableau.

## Format de sortie

Un fichier Markdown par feature : `scenarios-{slug-feature}.md`, structuré selon `assets/template-scenarios.md`.

## Checklist avant de livrer

- [ ] **Reprendre les 11 règles de l'étape 7 une par une** et vérifier chaque scénario contre chacune — c'est la vérification principale, ne pas la survoler
- [ ] Balayage transverse des fixtures : chaque entité mutée est absente de **tous** les autres scénarios, y compris non adjacents
- [ ] Table "Données de test à préparer" complète et cohérente avec les pré-conditions déclarées
- [ ] Couverture : nominal, cas limites, résilience — les trois présents
- [ ] Mode indiqué (standard / dégradé), avec avertissement en tête si dégradé
- [ ] Monitoring tracé explicitement, même si rien n'a été trouvé
- [ ] Version du brief et ticket(s) référencés dans le contexte
- [ ] Tableau de couverture : une ligne par contrainte des étapes 2 à 4, chacune vérifiable sans lire le code, chacune pointant vers un scénario ou justifiée en point de vigilance
