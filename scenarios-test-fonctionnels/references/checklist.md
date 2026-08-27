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
- [ ] Chaque valeur de **Préparation** est réalisable par le CDP (`existant en staging`, `à créer au BO`, `à créer au BO à chaque run`) — aucune instruction destinée à un développeur, et tout état non créable au BO est signalé au CDP au lieu d'être masqué
- [ ] Noms lisibles partout ("Investisseur A", "Projet A") — aucun identifiant technique en kebab-case entre accents graves, et aucun suffixe de lettre là où une seule entité du type existe
- [ ] **Chasse aux données de test superflues** : pour chaque entrée mono-scénario, classer la mutation (nulle / additive / réversible / destructive). Seule une mutation destructive justifie une entrée dédiée — sinon fusionner. Supprimer toute entrée citée dans aucune pré-condition
- [ ] Balayage transverse : chaque entité mutée de façon destructive est absente de **tous** les autres scénarios, y compris non adjacents

## Exécutabilité

- [ ] Aucun nom de classe, méthode, attribut, exception, job ou token CSS dans le corps du fichier — uniquement dans l'annexe technique
- [ ] Aucun conditionnel dans les étapes ("si ce mécanisme est exposé")
- [ ] Aucun point de vigilance utilisé pour évacuer un scénario simplement pénible à monter

## Couverture

- [ ] Nominal, cas limites, résilience — les trois présents, avec au moins un cas de défaillance jouable depuis l'interface
- [ ] Permissions : pour chaque action réservée à un rôle, l'absence du point d'entrée est vérifiée dans un scénario ; le refus de soumission figure dans la table des contraintes non jouables
- [ ] **Régression : la revue des surfaces partagées a été faite** (redirection après connexion, tableau de bord existant, navigation commune, tunnel de paiement, emails, listes et filtres, permissions des rôles existants) — un scénario si l'une est touchée, sinon le constat de la revue. Jamais "feature entièrement nouvelle" sans cette revue, et jamais les deux formes
- [ ] Responsive mutualisé dans des scénarios existants, aucun scénario responsive dédié
- [ ] Tableau de couverture : une ligne par contrainte relevée en étapes 2 à 4, chacune vérifiable sans lire le code
- [ ] Table des contraintes non jouables remplie, avec pour chacune le verdict de couverture automatisée — et les contraintes ni couvertes en recette ni en test signalées au CDP dans le message de remise

## Contexte du fichier

- [ ] Mode indiqué (standard / dégradé sans code / dégradé sans attendu), avec l'avertissement correspondant en tête du fichier. En mode sans attendu, la liste des règles métier à confirmer par le CDP est présente
- [ ] **Section "Erreurs de monitoring liées à la feature" présente**, même vide, avec une ligne par erreur : effet observable, date, occurrences, traitement. Jamais réduite à une mention dans le contexte
- [ ] Brief et ticket(s) référencés, avec la version si le brief est versionné — sinon le dire
