---
name: scenarios-test-fonctionnels
description: Génère des scénarios de test fonctionnels end-to-end pour une feature, en croisant le code du repo GitHub de l'app (comportement réel) avec le brief fonctionnel et les tickets (comportement attendu). Utiliser dès qu'un CDP demande de "créer des scénarios de test", "scénarios de recette", "tester cette feature", "couvrir fonctionnellement", "vérifier les cas limites", "tableau de couverture" — mais aussi pour des formulations indirectes comme "qu'est-ce qu'on doit vérifier avant de livrer", "comment on valide que ça marche", "on est prêts pour la recette ?", même sans le mot "scénario". Déclencher aussi en anglais ("test scenarios", "QA plan", "acceptance testing", "how do we test this", "coverage table"), ou si la personne partage un lien de feature/brief/ticket en demandant comment le tester.
license: © Tatiana Villamizar Mojica. Tous droits réservés — conception et rédaction de l'autrice.
allowed-tools: Read, Grep, Glob, Write, WebFetch, Bash(git log:*), Bash(git show:*), Bash(git diff:*), mcp__*, mcp__claude_ai_Slite__*, mcp__claude_ai_Trello__*, mcp__claude_ai_Linear__*, mcp__claude_ai_Sentry__*
---

# Génération de scénarios de test fonctionnels

## Principe

Le **code applicatif** fait foi pour "ce qui existe réellement". Le **brief + les tickets** font foi pour "ce qui est attendu". Les questions posées pendant le workflow servent uniquement à faire trancher des **divergences non documentées** entre l'attendu et le réel — jamais des questions d'opinion ou de conception sur la feature elle-même.

## Fichiers de cette skill

| Fichier | Quand le lire |
|---|---|
| `references/regles-redaction.md` | **Obligatoire** avant l'étape 7 — les 14 règles, la couverture exigée, le volume |
| `assets/template-scenarios.md` | **Obligatoire** avant l'étape 7 — structure du fichier de sortie |
| `references/contraintes-non-jouables.md` | **Obligatoire** à l'étape 11 — ce qui sort de la recette et comment le restituer |
| `references/checklist.md` | **Obligatoire** à l'étape 12 — vérification avant livraison |
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

**Si le repo n'est pas accessible** (droits, connecteur en panne) → **mode dégradé "sans code"** plutôt que blocage : générer depuis le brief et les tickets seuls, et le signaler en tête du fichier de sortie. Ne jamais présenter des scénarios brief-only comme confrontés au code.

**Si le brief n'est pas accessible ou est vide** (outil en panne, carte de ticket sans contenu, droits manquants) → le dire au CDP et lui demander une source de comportement attendu avant de continuer : un autre document, les critères d'acceptation du ticket, ou une description à l'oral. S'il n'y en a aucune, passer en **mode dégradé "sans attendu"** : les scénarios décrivent alors ce que le code fait, pas ce qu'il devrait faire. C'est le mode le plus risqué — un bug présent dans le code y devient un résultat attendu. Le signaler en tête du fichier, et remplacer les questions de divergence (étapes 6 et 8) par une liste de règles métier à confirmer par le CDP.

### 2. Extraire le comportement attendu
Brief (initial + édité) + ticket(s) → règles métier, critères d'acceptation, cas limites anticipés, décisions tranchées vs questions ouvertes. Noter si la feature est **un ajout neuf** ou **une modification d'un comportement existant** (sert à l'étape 7 pour la régression).

### 3. Extraire le comportement réel (code)
Parcourir uniquement le code pertinent à la feature (routes/controllers, modèles, composants frontend), ancré sur : nom de la feature, PR/commits du ticket, routes du parcours. Repérer : validations et limites, gestion d'erreur et messages, comportements silencieux, breakpoints/composants responsive si la feature touche le FO.

**Ne pas consulter les specs/tests existants du repo** pour extraire le comportement attendu ou réel : cela biaiserait la comparaison brief ↔ code. Seul le code applicatif fait foi pour "ce qui existe".

**Une seule exception, à l'étape 11** : les tests automatisés peuvent être lus pour savoir si une contrainte **non jouable en recette manuelle** est déjà couverte. C'est un usage différent — mesurer une couverture existante, pas déduire un comportement.

### 4. Vérifier les erreurs déjà connues
Si un monitoring est accessible, chercher les erreurs récentes liées à la feature.

**Le livrable contient une section dédiée "Erreurs de monitoring liées à la feature"**, avec une ligne par erreur : effet observable, date, nombre d'occurrences, et comment elle est traitée (scénario, point de vigilance, ou hors périmètre). Cette section existe toujours, même vide — "Aucune erreur récente liée à la feature" ou "Monitoring non accessible". Ne jamais la réduire à une mention dans le contexte : c'est l'information que le CDP relit en priorité quand un scénario échoue.

Décrire l'**effet observable**, pas la trace technique : "l'ouverture du détail d'une enveloppe a échoué une fois le 18/08" et non le nom du contrôleur et de l'exception. Les références techniques vont dans l'annexe destinée à l'équipe back.

Chaque erreur est ensuite rattachée à un scénario de résilience (étape 7), ou à un point de vigilance (étape 9) si elle n'est reproductible d'aucune façon.

### 5. Détecter les divergences non documentées
Lister ce qui diverge entre attendu (étape 2) et réel (étape 3) sans avoir été tranché quelque part (brief, ticket, commentaire). Écart mineur et non ambigu → note de vigilance. Écart qui change le comportement testable → question de contexte.

**Cette étape laisse une trace obligatoire dans le livrable** — une ligne du bloc Contexte : `Divergences code ↔ brief : N détectées, tranchées au round 1` ou `aucune divergence non documentée — comparaison faite`. Sans elle, un fichier produit sans avoir comparé le brief au code est indistinguable d'un fichier où la comparaison n'a rien trouvé : les deux passent la checklist et affichent le mode standard. C'est l'étape qui justifie la skill, elle ne peut pas être la seule sans empreinte vérifiable.

### 6. Round 1 — questions (max 5, seulement si nécessaire)
Un seul message, numérotées, chacune avec le contexte minimal (extrait brief/ticket vs extrait code). Attendre la réponse. Aucune divergence non documentée → passer à l'étape 7 et le dire explicitement.

Réponse partielle : traiter les points répondus, et basculer les points restés sans réponse en points de vigilance (étape 9) plutôt que de relancer.

### 7. Rédiger les scénarios

**Ne pas commencer à rédiger avant d'avoir lu `references/regles-redaction.md` et `assets/template-scenarios.md`.** Le premier porte les 14 règles de rédaction, la couverture exigée et le volume à viser ; le second la structure du livrable. Ces règles ne sont pas devinables : chacune vient d'un défaut constaté sur une sortie réelle, et plusieurs vont à l'encontre du réflexe naturel — notamment sur la réutilisation des données de test et sur la régression.

Lire aussi `assets/exemple-scenarios.md` en cas de doute sur la granularité.

Produire ensuite le fichier selon le template : contexte, erreurs de monitoring, données de test, scénarios, points de vigilance, tableau de couverture, contraintes non jouables, annexe technique.

### 8. Round 2 — questions (max 5, seulement si nécessaire)
Uniquement si la rédaction a fait émerger de **nouvelles** divergences non documentées. Mêmes règles que le round 1.

### 9. Points de vigilance
Hors scénarios formels : ce qui mérite un œil humain en recette mais n'est **pas testable du tout** — dette technique, zone fragile, dépendance externe, erreur de monitoring non reproductible, écart mineur code/brief, question restée sans réponse, comportement non implémenté.

Ne jamais utiliser un point de vigilance pour évacuer un scénario simplement pénible à monter.

### 10. Tableau de couverture final
Une ligne par contrainte technique ou fonctionnelle relevée en étapes 2 à 4. **Seules les contraintes observables** — observable = vérifiable via UI, API, panel admin ou monitoring, sans lire le code. Contrainte vérifiable uniquement en lisant le code : chercher à la rendre observable autrement (admin, logs, dashboard) et la reformuler en critère d'acceptation ; sinon, ne pas la mettre dans le tableau.

`✓` signifie **un résultat attendu du fichier livré assère cette contrainte**, rien d'autre. Une contrainte réelle mais non couverte par un scénario du fichier est `✗`, avec renvoi vers l'étape 11 ou vers un point de vigilance.

**Traçabilité — règle dure.** Pour chaque ligne marquée ✓, relire le scénario cité et retrouver le **résultat attendu numéroté** qui assère littéralement la contrainte. Aucun résultat correspondant, ou résultat portant sur autre chose → **compléter le scénario**, ou passer la ligne en ✗. Jamais laisser le ✓.

Une ligne ✓ assortie d'une réserve ("à vérifier", "non testé explicitement") **est un ✗**. Formes que prend le défaut : voir `references/regles-redaction.md`.

### 11. Contraintes non jouables en recette manuelle

Basculer un feature flag, couper un service externe, tuer un traitement de fond, soumettre une action depuis un rôle non autorisé : ces contraintes sortent des scénarios sans disparaître. **Lire `references/contraintes-non-jouables.md`** — procédure complète et les trois cas habituels avec la couverture automatisée à y chercher.

1. **Mesurer la couverture automatisée** — chercher dans les tests du repo si la contrainte est déjà vérifiée. Seul moment où la skill lit les tests.
2. **Restituer dans une table dédiée** + une ligne `✗` au tableau de couverture. Couverte → référence du test en annexe. Non couverte → trou réel, à signaler au CDP dans le message de remise.
3. **Proposer sans les écrire d'office** — après remise du fichier, demander au CDP s'il veut des scénarios détaillés pour ces contraintes, à jouer avec un développeur. Si oui, fichier séparé `scenarios-{slug-feature}-dev.md`. Jamais dans le fichier de recette.

### 12. Vérification avant livraison

**Lire `references/checklist.md` et la parcourir intégralement avant de remettre le fichier.** Ne pas la reconstituer de mémoire : chaque item correspond à un défaut déjà constaté sur une sortie réelle.

Tout item non satisfait se corrige avant livraison. Un item qu'on choisit sciemment de ne pas satisfaire est signalé au CDP dans le message de remise, pas laissé silencieux dans le fichier.

## Format de sortie

Un fichier Markdown par feature : `scenarios-{slug-feature}.md`, structuré selon `assets/template-scenarios.md`.
