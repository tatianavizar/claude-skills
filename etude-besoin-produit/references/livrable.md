# Référence — Livrable C-suite

But : un document court, hiérarchisé, présentable à la direction, à la **DA du client**.

## Format
- **HTML/Artifact** (présentable, imprimable en PDF) stylé à la DA du client — charger la skill DA
  si elle existe (ex. `capsens-da`). Sinon, palette + 2 typos cohérentes, fond non-blanc-pur.
- ⚠️ CSP des Artifacts = **Google Fonts uniquement**. Si la DA utilise une autre fonderie
  (Fontshare, etc.), **substituer par les plus proches sur Google** (ex. Clash Grotesk→Space
  Grotesk, General Sans→Hanken Grotesk) en gardant les vraies familles en tête de stack. Le
  signaler au PM.
- **Données brutes à part** (doc/section séparé·e, **sans interprétation**) : résultats SQL/GA en
  tableaux. C'est le dossier de preuve qui résiste au challenge.

## Structure type (sections numérotées)
1. **Objectif** — ce que le produit sert *vraiment* (et pas).
2. **Contexte** — trajectoire (croissance), enjeu (acquisition vs conversion/rétention).
3. **Analyse des données** — 3.1 device (mobile sous-convertit), 3.2 comportement/gisement,
   3.3 features sous-exploitées (à faire briller).
4. **Leviers de valeur** — qualitatif, graphique et scannable (voir ci-dessous).
5. **Repères marché** — tableau benchmarks + sources ; + sous-bloc **Méthode & sources**.
6. **Business case** — Étape 1 (tableau uplift par levier : donnée · taux · résultat) + repère
   marché après le tableau ; Étape 2 (projection croissance + payback).
7. **Lecture stratégique** — les 4 temps, registre conseil.
> Méthode & sources : en **annexe/fin** ou repliée dans les Repères marché — **jamais** entre
> Objectif et Contexte (ça casse le fil et fait défensif).

## Règles de rendu (tirées des retours réels)
- **Montrer d'où sort chaque chiffre.** Un tableau « donnée réelle · taux · résultat » est plus
  lisible que des formules cachées. Rendre l'étape intermédiaire visible (ex. « 5 608 × 0,3 =
  1 682 actes »), pas juste « 5 608 × 0,3 × 200 € ».
- **Leviers = graphique**, pas un mur de texte : cartes avec badge Lx + mécanisme + effet « → ».
  Une phrase d'intro qui dit ce qu'est un levier et où on veut en venir (tout → collecte).
- **Un seul encart de synthèse** par endroit (pas deux qui se répètent).
- Tables denses → `overflow-x:auto`, `tabular-nums`. Chiffres alignés à droite.
- Ton : sobre, factuel. Supprimer le hedging inutile (« à confirmer » partout), les superlatifs,
  les « GO ! ». Nommer les choses côté lecteur.
- Mobile-friendly + styles `@media print` pour un PDF propre.

## Boucle de relecture
Le PM commente sur l'Artifact (fil ancré). Traiter chaque commentaire : éditer le fichier source →
**republier** (même URL) → répondre au fil (ce qui a été fait) → résoudre. Pour un commentaire
ambigu : **répondre pour clarifier, laisser le fil ouvert**, ne pas deviner.

## Checklist avant envoi
- [ ] Chaque chiffre est soit DATA (sourcé) soit HYPOTHÈSE (marquée).
- [ ] Médianes utilisées là où il y a des outliers.
- [ ] Pics exceptionnels exclus des moyennes.
- [ ] Revenu = volume × marge client (pas volume brut).
- [ ] Payback + mise en perspective (part d'une année de marge).
- [ ] Section « quand ne pas le faire » présente.
- [ ] Données brutes documentées à part.
- [ ] DA du client appliquée.
