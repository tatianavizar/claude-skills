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

---

## Mutualisation des parcours

**Règle** — objectif : le moins de scénarios possible sans perdre de couverture. Un scénario peut valider plusieurs règles à la fois tant que le déroulé reste lisible étape par étape.

**À mutualiser** — quand le même rôle enchaîne naturellement plusieurs actions liées dans une même session : un admin qui crée un CGP *puis* renvoie son invitation ; qui archive *puis* vérifie l'effet de l'archivage. Un scénario par action fait gonfler le fichier sans ajouter de couverture réelle.

**Extrême inverse à éviter** — le scénario fleuve illisible qui mélange des rôles ou des objectifs sans rapport.

---

## Scénario de régression

**Règle** — si la feature modifie un comportement existant (constaté à l'étape 2), inclure un scénario dédié qui vérifie que l'ancien comportement encore attendu fonctionne toujours.

**Si la feature est entièrement nouvelle** — l'indiquer explicitement ("pas de régression pertinente — feature entièrement nouvelle") plutôt que d'omettre la section : une section absente ne se distingue pas d'un oubli.

---

## Scénario feature flag désactivé

**Règle** — si la feature est conditionnée par un feature flag, inclure un scénario qui vérifie le comportement flag OFF : 404 ou redirection sur toutes les URLs concernées, jamais une 500.

**Pourquoi** — au même titre que le responsive, ce n'est pas un cas optionnel à inclure "si le temps le permet" : c'est l'état dans lequel la feature part en prod avant activation.

**Si pas de flag** — l'indiquer explicitement.

---

## Responsive

**Règle** — vérifié **mutualisé** dans les scénarios existants (une étape mobile ajoutée à un scénario de souscription ou d'invitation), jamais dans un scénario dédié séparé. Mobile + desktop au minimum, si la feature touche le FO.

**Pourquoi** — un scénario responsive séparé rejoue le même parcours une deuxième fois : coût de recette doublé pour la même couverture.
