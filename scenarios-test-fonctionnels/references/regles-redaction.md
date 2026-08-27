# Règles de rédaction — détail et pièges

Référence de l'étape 7 de `SKILL.md`. Chaque section reprend une règle, son *pourquoi*, et le piège concret qui la rend nécessaire.

---

## Indépendance des scénarios

**Règle** — chaque scénario est exécutable seul, sans dépendre de l'état laissé par un scénario précédent. Interdit : "en reprenant les données du scénario 2".

**Pourquoi** — un scénario dépendant ne peut pas être rejoué isolément en recette (un CDP qui veut revérifier le scénario 5 après un fix doit rejouer 1 à 4), ni automatisé plus tard.

---

## Données de test : réutiliser par défaut, dupliquer par exception

**Le coût de préparation des données est le premier poste de temps en recette**, très loin devant l'exécution des scénarios. Une donnée de test par scénario transforme un fichier de 10 scénarios en une demi-journée de saisie au BO avant même de commencer. La règle par défaut est donc la **réutilisation**, et chaque duplication doit se justifier.

### Classer la mutation avant de dupliquer

| Type | Exemples | Réutilisable ? |
|---|---|---|
| **Nulle** | consultation, filtre, export, impersonation, vidage de cache, **soumission refusée** | Toujours. L'entité sort du scénario dans l'état où elle est entrée. |
| **Additive** | gagne une souscription, un rattachement, une invitation | Oui, sauf si un autre scénario assère un **décompte ou une liste exacte** portant sur cette entité. |
| **Réversible via l'UI** | archiver puis désarchiver, activer puis désactiver, dans le même scénario | Oui, si le scénario remet explicitement l'état d'origine en dernière étape. |
| **Destructive ou exclusive** | archivage définitif, suppression, changement de rattachement, passage à un statut irréversible | Non. Donnée de test dédiée, exclusive sur **tout** le fichier. |

Seule la dernière ligne justifie une donnée de test dédiée. C'est beaucoup plus rare qu'il n'y paraît à la rédaction.

**Le gisement principal : les scénarios de cas bloquants.** Un scénario qui vérifie qu'une action est refusée (dépassement de montant, doublon, permission insuffisante, données invalides) ne mute **rien** — la soumission a échoué. Ces scénarios n'ont jamais besoin de donnée de test dédiée, et ce sont souvent les plus nombreux.

### Réutiliser le sujet coûteux, dupliquer l'objet bon marché

Toutes les données de test ne coûtent pas le même prix. Un investisseur avec KYC validé demande un parcours d'onboarding complet ; un projet ou une enveloppe se crée en deux minutes dans le BO.

Donc : **un seul investisseur actif qui souscrit sur trois projets différents**, jamais trois investisseurs. Quand deux scénarios entrent en conflit sur une entité chère, faire varier l'objet de l'action (le projet, l'enveloppe, le montant) plutôt que le sujet.

### Vérifications à faire sur la table

- **Données de test inutilisées** : toute entité déclarée dans la table mais citée dans aucune pré-condition est à supprimer. Elles s'accumulent quand la rédaction évolue et gonflent le coût de prep pour rien.
- **Données de test mono-scénario** : chaque entité dont la colonne `Muté par` ne cite qu'un scénario doit passer le test de la classification ci-dessus. Si la mutation n'est pas destructive, fusionner avec une donnée de test existante.
- **Colonne `Muté par`** : un seul scénario, ou `—` si l'entité n'est jamais mutée de façon destructive. Deux scénarios destructifs dans la case = défaut.
- **Colonne `Préparation`** : `existant en staging` / `BO` / `à créer avant chaque run`. Rend le coût visible avant de commencer, et distingue ce qui est réutilisable d'un run à l'autre.
- **Entités non-rôles** : projets, enveloppes, réseaux, comptes externes cités en pré-conditions doivent figurer dans la table. Une pré-condition qui référence un identifiant absent est une donnée de test que personne ne préparera.

### Pièges

**L'étiquette "lecture seule" appliquée à un rôle acteur** — un CGP qui invite un investisseur consomme son quota journalier et gagne un rattachement. C'est de l'additif, donc réutilisable, mais pas `—` : avant d'écrire `—`, se demander si une assertion d'un autre scénario portant sur cette entité (décompte, liste, quota) donnerait encore le même résultat.

**Deux chemins de code pour la même mutation destructive** — un archivage déclenché depuis le BO *et* depuis un portail distinct passe par deux policies différentes et doit être testé deux fois. Là, deux données de test sont nécessaires : c'est la seule duplication qu'on ne peut pas éviter.

**Une pré-condition trop précise crée une donnée de test** — "aucune souscription en attente sur `projet-nominale`" oblige à un investisseur vierge sur ce projet. Souvent, changer de projet cible suffit à réutiliser un investisseur existant.

---

## Nommer les données de test

**Règle** — des noms lisibles : "Investisseur A", "Collaborateur B", "Projet A", "Enveloppe presque vide". Jamais d'identifiant technique en kebab-case entre accents graves.

**Pourquoi** — `` `investisseur-pour-souscription` `` répété huit fois dans une pré-condition et quatre fois dans les étapes transforme un scénario en extrait de code. La personne qui exécute lit une phrase, pas une variable. Le poids visuel des accents graves fait perdre le fil du parcours.

**Comparaison**

| ❌ Identifiant technique | ✅ Nom lisible |
|---|---|
| "Sélectionner `investisseur-pour-souscription`, `projet-nominale`, montant 1 000 €, et `enveloppe-nominale`" | "Sélectionner Investisseur A, Projet A, un montant de 1 000 €, et Enveloppe A" |
| "`cgp-collab-actif` connecté ; réseau `reseau-tantiem` contenant `cgp-responsable`" | "Collaborateur A connecté ; son réseau contient aussi le Responsable" |

**Comment garder la traçabilité** — la table des données de test porte le nom lisible en première colonne et décrit l'état exact à préparer. Le lien se fait par le nom, pas par un identifiant. Quand un rôle suffit à lever l'ambiguïté (un seul admin dans tout le fichier), écrire "l'admin" plutôt que de lui inventer une étiquette.

**Suffixes de lettres** — n'ajouter A, B, C que lorsque plusieurs entités du même type coexistent. Un fichier avec un seul investisseur écrit "l'investisseur".

---

## Pré-conditions

**Règle** — rôle de test avec ses conditions précises, feature flags actifs, état des données requis.

**Pourquoi** — "un investisseur" ne suffit pas : selon son KYC, son solde ou son statut averti/non averti, le comportement testé change. Le CDP qui exécute doit pouvoir préparer l'environnement sans deviner.

**Exemples** — ✅ "investisseur avec KYC validé et solde suffisant" · ❌ "un investisseur".

---

## Ordre des scénarios

**Règle** — ordonner selon la logique métier naturelle du parcours (créer un profil avant de tester son archivage), jamais par criticité.

**Pourquoi** — l'ordre et la criticité répondent à deux questions différentes : l'ordre sert à exécuter, la criticité sert à prioriser l'effort. Les mélanger rend le fichier illisible à l'exécution.

---

## Criticité

**Règle** — dans le titre de chaque scénario, entre parenthèses :

| Niveau | Périmètre |
|---|---|
| `Critique` | sécurité, permissions, données sensibles/réglementées, flux financier |
| `Majeure` | chemin nominal, cas limites majeurs |
| `Mineure` | UI, responsive, cosmétique |

**Pourquoi** — la criticité = gravité des conséquences si le bug passe en prod, **pas** la complexité du scénario. Elle sert à prioriser l'effort de recette quand le temps manque, pas à ordonner le fichier.

---

## Étapes et résultats attendus

**Règle** — deux listes numérotées parallèles : étape 1 → résultat attendu 1, étape 2 → résultat attendu 2, etc. Pas seulement un résultat global à la fin.

**Pourquoi** — c'est ce qui permet à quelqu'un d'autre que le rédacteur de valider pas à pas, sans ambiguïté sur ce qui est censé se passer à chaque étape. Un résultat global ne dit pas *où* ça a cassé.

**Pas de résultat groupé** — `1–5. Chaque URL renvoie un 404` est interdit : 5 étapes = 5 entrées. Le groupement se produit surtout quand les étapes se ressemblent (une liste d'URLs, une liste de champs) — c'est justement là qu'un cas particulier passe inaperçu.

---

## Un résultat attendu n'est pas une reformulation de son étape

**Règle** — le résultat énonce un **état observable après l'action**, jamais l'action.

**Pourquoi** — un résultat qui répète l'étape est une case à cocher vide : le CDP la coche parce qu'il a fait l'action, sans avoir rien vérifié. C'est un faux positif silencieux dans le fichier de recette.

**Exemples**

| ❌ Tautologie | ✅ État observable |
|---|---|
| "Formulaire soumis avec un montant dépassant le disponible" | "Message d'erreur citant le montant disponible restant ; aucune souscription créée ; retour sur le formulaire avec les valeurs saisies" |
| "L'arrêt de l'impersonation se déclenche correctement" | "La bannière d'impersonation disparaît, la session revient sur le compte CGP, redirection vers la liste des investisseurs" |
| "L'export est lancé" | "Retour visuel immédiat (spinner), l'interface reste utilisable, une entrée apparaît dans l'historique des exports" |

**Test rapide** — si le résultat peut s'écrire en réutilisant le verbe de l'étape au participe passé, c'est une tautologie.

---

## Traçabilité entre scénarios et tableau de couverture

**Règle** — chaque ligne ✓ du tableau doit correspondre à un **résultat attendu numéroté** qui vérifie littéralement la contrainte. Voir l'étape 10 de `SKILL.md` pour la procédure.

**Pourquoi** — c'est le défaut le plus grave possible sur un fichier de recette : le tableau annonce "couvert", le CDP fait confiance, et la contrainte n'a jamais été exercée. Sur des lignes permissions ou flux financier, ça envoie un bug en prod avec une trace écrite disant qu'il a été testé.

**Formes que prend le défaut**

- **La contrainte porte sur l'après, le résultat sur l'avant.** "Le montant de l'enveloppe est décrémenté après souscription" → le scénario montre le montant disponible *dans le formulaire, avant validation*, et ne le recontrôle jamais. Il manque une étape "rouvrir la liste des enveloppes" et son résultat.
- **Le critère invente une vérification que le scénario ne fait pas.** "Bouton absent pour le Collaborateur (sc1)" alors que sc1 ne teste que les liens de la sidebar et l'accès à une URL. Un lien de sidebar absent ne dit rien de l'absence d'un bouton dans une liste.
- **Le ✓ porte sa propre réserve.** "✓ (non testé explicitement : à vérifier sur le fichier)". La parenthèse dit exactement que c'est un ✗.
- **Le scénario cité teste le bon sujet avec le mauvais rôle.** Une contrainte de cloisonnement vérifiée uniquement côté rôle privilégié.

---

## Résilience : le minimum exigé

**Règle** — au moins un scénario de défaillance infra/asynchrone, plus un scénario par erreur remontée par le monitoring à l'étape 4. Si la feature contient un job de fond, un webhook, une notification temps réel ou un export, le cas "le traitement échoue" est obligatoire.

**Pourquoi** — les cas limites métier (dépassement de montant, doublon, valeur invalide) viennent naturellement à la rédaction parce qu'ils sont dans le brief. Les défaillances techniques n'y sont pas, donc elles sautent — alors que c'est précisément la partie que personne n'a testée en développement.

**Piège** — un export asynchrone "qui marche" est facile à tester ; un export dont le job meurt à mi-chemin laisse quoi à l'écran ? Un spinner infini, une entrée d'historique fantôme, ou un message clair ? C'est la question qui compte, et elle n'est presque jamais posée.

**Où le mettre** — un cas de défaillance déclenchable depuis l'interface (quota atteint, données incohérentes, double soumission) est un scénario de recette normal. Un cas qui demande de couper un service ou de tuer un traitement passe à l'étape 11 : on vérifie d'abord s'il est couvert par un test automatisé, et le scénario détaillé n'est rédigé que si le CDP le demande.

---

## Permissions : lecture et écriture

**Règle** — pour chaque action réservée à un rôle, vérifier **l'absence du point d'entrée** (bouton, lien, champ) *et* **le refus de la soumission** par un rôle non autorisé.

**Pourquoi** — un `Accès refusé` sur une page ne prouve rien sur la protection de l'action correspondante. Une interface qui cache le bouton tout en acceptant la soumission est une escalade de privilèges classique, invisible si on ne teste que la navigation.

**Découpage par exécutant** :
- **Absence du point d'entrée** → scénario de recette normal : se connecter avec le rôle restreint, vérifier que le bouton, le lien ou le champ n'apparaît pas dans les écrans concernés.
- **Refus de la soumission** → généralement pas jouable par un CDP. Deux exceptions à tenter d'abord : le formulaire laissé ouvert dans un onglet puis soumis après changement de compte, et le lien d'action copié depuis la session d'un rôle autorisé. Si aucune ne marche, la contrainte passe à l'étape 11 — c'est typiquement le cas où un test de policy existe déjà côté repo, et où le vérifier vaut mieux que de faire jouer une requête manuelle au CDP.

---

## Contraintes non jouables en recette manuelle

Référence de l'étape 11. Concerne les contraintes réelles qu'un CDP ne peut pas exercer depuis l'interface : basculer un feature flag, rendre un service externe injoignable, interrompre un traitement de fond, soumettre une action depuis un rôle non autorisé.

**Pourquoi les sortir du fichier de recette** — un scénario que personne ne jouera dans la session alourdit la lecture, et son résultat attendu ne sera jamais coché. Le fichier de recette doit décrire ce qui va effectivement être fait. Mais supprimer la contrainte serait pire : elle disparaîtrait du radar alors que c'est souvent la plus risquée.

### 1. Mesurer la couverture automatisée

Chercher dans les tests du repo si la contrainte est déjà vérifiée : tests de policy pour les permissions, tests de requête pour les routes protégées, tests de job pour les traitements de fond, tests dédiés pour les feature flags.

C'est **le seul endroit de la skill où les tests du repo sont lus**, et pour un usage précis : mesurer une couverture existante. Jamais pour déduire ce que la feature est censée faire — cette interdiction (étape 3) reste entière.

### 2. Restituer honnêtement

Une table dédiée dans le livrable, une ligne par contrainte : la contrainte, le verdict de couverture, la suite à donner. Et une ligne correspondante au tableau de couverture, en `✗`, puisque aucun résultat attendu du fichier ne l'assère.

- **Couverte par un test automatisé** → rien à jouer manuellement, référence du test en annexe technique. C'est une bonne nouvelle, elle doit être visible : sans ça, le CDP croit à un trou.
- **Non couverte** → trou réel, ni en recette ni en test. À signaler au CDP dans le message de remise : c'est souvent la demande la plus utile à porter à l'équipe, surtout quand la contrainte correspond à une erreur déjà vue en monitoring.

### 3. Proposer les scénarios détaillés, ne pas les imposer

Après avoir remis le fichier de recette, demander au CDP s'il veut en plus des scénarios détaillés pour ces contraintes, à jouer avec un développeur. S'il accepte : fichier séparé `scenarios-{slug-feature}-dev.md`, un scénario par contrainte, avec pour chacun **ce qu'il faut obtenir de l'équipe** (interrupteur d'activation, moyen de couper le service, accès pour rejouer une soumission).

Ne jamais les intégrer au fichier principal, même en fin de document : ils ne seront pas joués dans la même session.

---

## Vocabulaire : le livrable ne cite jamais de code

**Règle** — aucun nom de classe, de méthode, d'attribut, d'exception, de job, de token CSS ou de fichier dans le corps du livrable. Les références techniques vont en annexe, dans une section destinée à l'équipe back.

**Pourquoi** — l'étape 3 fait lire le code, donc son vocabulaire remonte naturellement dans la rédaction. Mais la personne qui exécute ne l'a pas lu : `is_manager? == true` ne lui dit pas comment mettre une donnée de test dans cet état, et une pré-condition qu'on ne sait pas préparer bloque le scénario entier.

**Traductions**

| ❌ Code | ✅ Observable |
|---|---|
| `is_manager? == true` | "CGP avec le rôle Responsable dans le BO" |
| "quota `daily_invitations_number` > 0" | "réseau autorisant au moins une invitation par jour (à vérifier au BO avant de commencer)" |
| "fond de couleur `new-primary-20`" | "fond coloré, texte et icônes lisibles" |
| "`Distributors::BudgetsController#show` → `ActionController::UnknownFormat`" | "l'ouverture du détail d'une enveloppe a échoué une fois le 18/08 — voir annexe technique" |
| "IndexQuery avec scopes `completed`/`created`" | "les filtres Finalisé / Non finalisé ont été corrigés depuis le signalement" |

**Annexe technique** — regrouper en fin de fichier les références de code, classes d'exception et noms de jobs qui justifient un point de vigilance. Utile à l'équipe back, invisible pour qui exécute.

---

## Aucun conditionnel dans les étapes

**Règle** — une étape ne contient jamais "si ce mécanisme est exposé", "si le bouton existe", "le cas échéant". Une étape est une instruction exécutable sans arbitrage.

**Pourquoi** — un conditionnel dans une étape signifie que la rédaction n'a pas tranché et refile la question à l'exécutant, qui a moins d'information qu'elle. Il ne saura pas décider, et le résultat attendu devient inévaluable.

**Que faire à la place** — l'incertitude est une divergence non documentée : elle part en question au round 1 ou 2 (étapes 6 et 8), et à défaut de réponse elle devient un point de vigilance. Le livrable, lui, affirme.

---

## Mutualisation des parcours

**Règle** — objectif : le moins de scénarios possible sans perdre de couverture. Un scénario peut valider plusieurs règles à la fois tant que le déroulé reste lisible étape par étape.

**À mutualiser** — quand le même rôle enchaîne naturellement plusieurs actions liées dans une même session : un admin qui crée un CGP *puis* renvoie son invitation ; qui archive *puis* vérifie l'effet de l'archivage. Un scénario par action fait gonfler le fichier sans ajouter de couverture réelle.

**Extrême inverse à éviter** — le scénario fleuve illisible qui mélange des rôles ou des objectifs sans rapport.

---

## Scénario de régression

**Règle** — si la feature modifie un comportement existant (constaté à l'étape 2), inclure un scénario dédié qui vérifie que l'ancien comportement encore attendu fonctionne toujours.

**"Feature entièrement nouvelle" est presque toujours faux.** Une feature qui ajoute un espace, un rôle ou un tunnel à une application existante modifie des **surfaces partagées**. Avant de conclure à l'absence de régression, passer cette liste et dire ce qu'on a constaté :

| Surface partagée | Question à se poser |
|---|---|
| Redirection après connexion | Les utilisateurs existants atterrissent-ils toujours au même endroit ? |
| Tableau de bord existant | Un bloc, une card ou un compteur y a-t-il été ajouté ? |
| Navigation commune | Un lien, un menu ou un libellé a-t-il changé pour les rôles existants ? |
| Tunnel de paiement / signature | Peut-il désormais être atteint par un chemin nouveau ? |
| Emails déjà envoyés | Un envoi existant a-t-il été modifié, ou un nouveau s'ajoute-t-il au même déclencheur ? |
| Listes et filtres existants | Une colonne, un filtre ou un tri a-t-il été ajouté à un écran partagé ? |
| Modèle de permissions | Un rôle existant a-t-il gagné ou perdu un accès ? |

**Ce que vérifie le scénario de régression** — le parcours pré-existant, déclenché **sans passer par la nouvelle feature**, fonctionne à l'identique. Exemple : un espace CGP est ajouté, et les souscriptions peuvent maintenant être créées par un CGP. La régression n'est pas "l'investisseur finalise une réservation créée par son CGP" — ça, c'est de l'intégration entre deux parties nouvelles. La régression, c'est "**l'investisseur souscrit de lui-même, sans aucun CGP impliqué**, et son parcours est inchangé" : même tableau de bord, même tunnel, mêmes emails, aucune card parasite.

**Une seule forme, jamais les deux.** Un fichier qui contient à la fois un scénario titré "Régression" et une mention "feature entièrement nouvelle, pas de régression" se contredit — le lecteur ne sait plus si le scénario doit être joué. Et un scénario qui teste l'intégration entre deux parties nouvelles est un **scénario d'intégration**, à renommer.

**Si aucune surface partagée n'est touchée** — le dire en citant la revue ("aucune régression : la feature n'ajoute ni redirection, ni bloc sur un écran existant, ni email sur un déclencheur existant") plutôt qu'en affirmant "feature entièrement nouvelle". La première formulation se vérifie, la seconde non.

---

## Scénario feature flag désactivé

**Règle** — si la feature est conditionnée par un feature flag, inclure un scénario qui vérifie le comportement flag OFF : 404 ou redirection sur toutes les URLs concernées, jamais une 500.

**Pourquoi** — au même titre que le responsive, ce n'est pas un cas optionnel à inclure "si le temps le permet" : c'est l'état dans lequel la feature part en prod avant activation.

**Si pas de flag** — l'indiquer explicitement.

---

## Responsive

**Règle** — vérifié **mutualisé** dans les scénarios existants (une étape mobile ajoutée à un scénario de souscription ou d'invitation), jamais dans un scénario dédié séparé. Mobile + desktop au minimum, si la feature touche le FO.

**Pourquoi** — un scénario responsive séparé rejoue le même parcours une deuxième fois : coût de recette doublé pour la même couverture.
