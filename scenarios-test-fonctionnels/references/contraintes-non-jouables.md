# Contraintes non jouables en recette manuelle

Référence de l'étape 11 de `SKILL.md`. À lire une fois les scénarios rédigés, pas avant.

Concerne les contraintes réelles qu'un CDP ne peut pas exercer depuis l'interface : basculer un feature flag, rendre un service externe injoignable, interrompre un traitement de fond, soumettre une action depuis un rôle non autorisé.

**Pourquoi les sortir du fichier de recette** — un scénario que personne ne jouera dans la session alourdit la lecture, et son résultat attendu ne sera jamais coché. Le fichier de recette doit décrire ce qui va effectivement être fait. Mais supprimer la contrainte serait pire : elle disparaîtrait du radar alors que c'est souvent la plus risquée.

## 1. Mesurer la couverture automatisée

Chercher dans les tests du repo si la contrainte est déjà vérifiée : tests de policy pour les permissions, tests de requête pour les routes protégées, tests de job pour les traitements de fond, tests dédiés pour les feature flags.

C'est **le seul endroit de la skill où les tests du repo sont lus**, et pour un usage précis : mesurer une couverture existante. Jamais pour déduire ce que la feature est censée faire — cette interdiction (étape 3) reste entière.

## 2. Restituer honnêtement

Une table dédiée dans le livrable, une ligne par contrainte : la contrainte, le verdict de couverture, la suite à donner. Et une ligne correspondante au tableau de couverture, en `✗`, puisque aucun résultat attendu du fichier ne l'assère.

- **Couverte par un test automatisé** → rien à jouer manuellement, référence du test en annexe technique. C'est une bonne nouvelle, elle doit être visible : sans ça, le CDP croit à un trou.
- **Non couverte** → trou réel, ni en recette ni en test. À signaler au CDP dans le message de remise : c'est souvent la demande la plus utile à porter à l'équipe, surtout quand la contrainte correspond à une erreur déjà vue en monitoring.

## 3. Proposer les scénarios détaillés, ne pas les imposer

Après avoir remis le fichier de recette, demander au CDP s'il veut en plus des scénarios détaillés pour ces contraintes, à jouer avec un développeur. S'il accepte : fichier séparé `scenarios-{slug-feature}-dev.md`, un scénario par contrainte, avec pour chacun **ce qu'il faut obtenir de l'équipe** (interrupteur d'activation, moyen de couper le service, accès pour rejouer une soumission).

Ne jamais les intégrer au fichier principal, même en fin de document : ils ne seront pas joués dans la même session.

---

## Les trois cas habituels

### Feature flag désactivé

**Contrainte** — si la feature est conditionnée par un flag, le comportement flag OFF doit être vérifié : page introuvable ou redirection sur toutes les adresses concernées, jamais une erreur serveur.

**Pourquoi ça compte** — au même titre que le responsive, ce n'est pas un cas optionnel à traiter "si le temps le permet" : c'est l'état dans lequel la feature part en production avant activation.

**Couverture automatisée à chercher** — un test dédié au flag, ou des tests de requête sur les routes du périmètre avec le flag inactif.

**Si la feature n'est pas derrière un flag** — l'indiquer explicitement dans le livrable, pour que l'absence ne se confonde pas avec un oubli.

### Refus de la soumission par un rôle non autorisé

**Contrainte** — un `Accès refusé` sur une page ne prouve rien sur la protection de l'action correspondante. Une interface qui cache le bouton tout en acceptant la soumission est une escalade de privilèges classique, invisible si on ne teste que la navigation.

**Ce qui reste en recette** — l'absence du point d'entrée (bouton, lien, champ) pour le rôle restreint est un scénario normal, à l'étape 7.

**Deux exceptions à tenter avant de renoncer** — le formulaire laissé ouvert dans un onglet puis soumis après changement de compte ; le lien d'action copié depuis la session d'un rôle autorisé. Si l'une des deux marche, c'est un scénario de recette.

**Couverture automatisée à chercher** — tests de policy sur l'action, tests de requête retournant un refus pour le rôle non autorisé. C'est le cas où la couverture existe le plus souvent : la vérifier vaut mieux que de faire jouer une requête manuelle au CDP.

### Défaillance d'infrastructure ou de traitement de fond

**Contrainte** — service externe injoignable, délai d'attente dépassé, traitement de fond interrompu en cours d'exécution. Ce qui compte n'est pas que ça échoue, mais **ce que l'interface montre** quand ça échoue : un message explicite, ou un compteur bloqué et un état incohérent.

**Ce qui reste en recette** — les défaillances déclenchables depuis l'interface (quota atteint, données incohérentes, double soumission) sont des scénarios normaux, à l'étape 7.

**Couverture automatisée à chercher** — tests du job ou du service qui simulent l'échec et vérifient l'état laissé derrière.

**Piège** — un traitement asynchrone qui réussit est facile à tester ; celui qui meurt à mi-chemin laisse quoi à l'écran ? Un compteur qui tourne indéfiniment, une entrée d'historique fantôme, ou un message clair ? C'est la question qui compte, et elle n'est presque jamais posée. Si rien ne la couvre, c'est le trou à remonter en priorité.
