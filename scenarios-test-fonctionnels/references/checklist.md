# Checklist avant de livrer

Dernière étape avant de remettre le fichier. À lire et à parcourir intégralement — chaque item correspond à un défaut déjà constaté sur une sortie réelle, pas à une précaution théorique.

Les items sont classés par gravité : les trois premiers sont ceux qui produisent un fichier de recette **faussement rassurant**, c'est-à-dire pire qu'un fichier incomplet.

## Fiabilité des vérifications

- [ ] **Reprendre les règles de l'étape 7 une par une** et vérifier chaque scénario contre chacune — vérification principale, ne pas la survoler
- [ ] **Traçabilité du tableau : pour chaque ligne ✓, retrouver le résultat attendu numéroté qui l'assère.** Aucun résultat correspondant, ou résultat portant sur autre chose → compléter le scénario, ou passer la ligne en ✗ avec renvoi en point de vigilance. Toute ligne ✓ portant une réserve ("à vérifier", "non testé explicitement") devient ✗
- [ ] Chaque résultat attendu énonce un état observable après l'action, aucun ne reformule son étape
- [ ] Aucun résultat groupé sur plusieurs étapes ("1–5. chaque adresse renvoie...")

## Coût de préparation

- [ ] Table "Données de test à préparer" : toutes les entités citées en pré-conditions y figurent (comptes, projets, enveloppes, réseaux), colonnes **Muté par** et **Préparation** renseignées
- [ ] **Chasse aux données de test superflues** : pour chaque entrée mono-scénario, classer la mutation (nulle / additive / réversible / destructive). Seule une mutation destructive justifie une entrée dédiée — sinon fusionner. Supprimer toute entrée citée dans aucune pré-condition
- [ ] Balayage transverse : chaque entité mutée de façon destructive est absente de **tous** les autres scénarios, y compris non adjacents

## Exécutabilité

- [ ] Aucun nom de classe, méthode, attribut, exception, job ou token CSS dans le corps du fichier — uniquement dans l'annexe technique
- [ ] Aucun conditionnel dans les étapes ("si ce mécanisme est exposé")
- [ ] Champ **Exécutable par** renseigné sur chaque scénario, et tous les `avec un dev` regroupés dans leur section avec ce qu'il faut obtenir de l'équipe
- [ ] Aucun point de vigilance utilisé pour évacuer un scénario simplement pénible à monter

## Couverture

- [ ] Nominal, cas limites, résilience — les trois présents, dont au moins un cas de défaillance asynchrone ou d'infrastructure si la feature en contient
- [ ] Permissions : chaque action réservée à un rôle est testée en absence de point d'entrée (corps de la recette) **et** en refus de soumission (corps si jouable, sinon section dev)
- [ ] Régression : soit un scénario, soit une mention d'absence explicite — pas les deux
- [ ] Responsive mutualisé dans des scénarios existants, aucun scénario responsive dédié
- [ ] Tableau de couverture : une ligne par contrainte relevée en étapes 2 à 4, chacune vérifiable sans lire le code

## Contexte du fichier

- [ ] Mode indiqué (standard / dégradé), avec avertissement en tête si dégradé
- [ ] Monitoring tracé explicitement, même si rien n'a été trouvé
- [ ] Version du brief et ticket(s) référencés dans le contexte
