# Règles de rédaction — détail et pièges

Référence de l'étape 7 de `SKILL.md`. Chaque section reprend une règle, son *pourquoi*, et le piège concret qui la rend nécessaire.

---

## Indépendance des scénarios

**Règle** — chaque scénario est exécutable seul, sans dépendre de l'état laissé par un scénario précédent. Interdit : "en reprenant les données du scénario 2".

**Pourquoi** — un scénario dépendant ne peut pas être rejoué isolément en recette (un CDP qui veut revérifier le scénario 5 après un fix doit rejouer 1 à 4), ni automatisé plus tard.

---

## Fixtures et entités mutées

**Règle** — si un scénario modifie l'état d'une entité de test (archivage, suppression, changement de statut, désactivation...), cette entité lui est exclusive sur **l'ensemble du fichier**, pas seulement vis-à-vis des scénarios qui le suivent.

**Pourquoi** — deux scénarios non adjacents peuvent muter la même fixture sans que ça saute aux yeux à la rédaction. Le réflexe : une entité mutée = une fixture dédiée, point final.

**Piège fréquent** — deux rôles ou deux interfaces qui doivent chacun tester la même action de mutation (ex. archivage déclenché depuis le BO *et* depuis le portail manager, parce que ce sont deux chemins de code / policies distincts) : créer une fixture séparée pour chacun plutôt que de la partager.

**Colonne `Muté par` obligatoire** dans la table "Données de test à préparer" : un seul scénario, ou `—` pour une entité en lecture seule sur tout le fichier. Deux scénarios dans la case = défaut à corriger.

**Piège de l'étiquette "lecture seule"** — un rôle acteur est presque toujours muté, même quand on le pense passif. Un CGP qui invite un investisseur consomme son quota d'invitations journalières et gagne un rattachement ; un responsable qui se fait réaffecter un investisseur change de périmètre. Avant d'écrire `—`, se demander : après ce scénario, une assertion d'un autre scénario portant sur cette entité (un décompte, une liste, un quota) donnerait-elle encore le même résultat ?

**Entités non-rôles** — la table doit lister les projets, enveloppes, réseaux et comptes externes cités en pré-conditions, pas seulement les comptes utilisateurs. Une pré-condition qui référence un identifiant absent de la table est une fixture que personne ne préparera.

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

---

## Permissions : lecture et écriture

**Règle** — pour chaque action réservée à un rôle, vérifier **l'absence du point d'entrée** (bouton, lien, champ) *et* **le refus de la soumission** par un rôle non autorisé.

**Pourquoi** — un `Accès refusé` sur un GET ne prouve rien sur la policy de l'action correspondante. Une interface qui cache le bouton tout en acceptant le POST est un scénario d'escalade de privilèges classique, et il est invisible si on ne teste que la navigation.

**Comment le rendre observable par un CDP** — rejouer la soumission depuis le rôle non autorisé : formulaire d'un autre rôle laissé ouvert dans un onglet puis soumis après changement de compte, ou appel direct de l'URL de l'action. Si aucune de ces manipulations n'est réalisable en recette, le noter en point de vigilance à destination de l'équipe back plutôt que de marquer la contrainte couverte.

---

## Mutualisation des parcours

**Règle** — objectif : le moins de scénarios possible sans perdre de couverture. Un scénario peut valider plusieurs règles à la fois tant que le déroulé reste lisible étape par étape.

**À mutualiser** — quand le même rôle enchaîne naturellement plusieurs actions liées dans une même session : un admin qui crée un CGP *puis* renvoie son invitation ; qui archive *puis* vérifie l'effet de l'archivage. Un scénario par action fait gonfler le fichier sans ajouter de couverture réelle.

**Extrême inverse à éviter** — le scénario fleuve illisible qui mélange des rôles ou des objectifs sans rapport.

---

## Scénario de régression

**Règle** — si la feature modifie un comportement existant (constaté à l'étape 2), inclure un scénario dédié qui vérifie que l'ancien comportement encore attendu fonctionne toujours.

**Si la feature est entièrement nouvelle** — l'indiquer explicitement ("pas de régression pertinente — feature entièrement nouvelle") plutôt que d'omettre la section : une section absente ne se distingue pas d'un oubli.

**Une seule forme, jamais les deux.** Un fichier qui contient à la fois un scénario titré "Régression" et une section disant "feature entièrement nouvelle, pas de régression" se contredit — le lecteur ne sait plus si le scénario doit être joué. Vérifier aussi que le scénario titré régression en est bien une : un scénario qui teste l'intégration entre deux nouvelles parties de la feature (ex. un investisseur finalise une souscription créée par son CGP, les deux étant nouveaux) est un **scénario d'intégration**, à renommer et à laisser dans la numérotation normale.

---

## Scénario feature flag désactivé

**Règle** — si la feature est conditionnée par un feature flag, inclure un scénario qui vérifie le comportement flag OFF : 404 ou redirection sur toutes les URLs concernées, jamais une 500.

**Pourquoi** — au même titre que le responsive, ce n'est pas un cas optionnel à inclure "si le temps le permet" : c'est l'état dans lequel la feature part en prod avant activation.

**Si pas de flag** — l'indiquer explicitement.

---

## Responsive

**Règle** — vérifié **mutualisé** dans les scénarios existants (une étape mobile ajoutée à un scénario de souscription ou d'invitation), jamais dans un scénario dédié séparé. Mobile + desktop au minimum, si la feature touche le FO.

**Pourquoi** — un scénario responsive séparé rejoue le même parcours une deuxième fois : coût de recette doublé pour la même couverture.
