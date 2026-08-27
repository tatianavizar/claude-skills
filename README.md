# claude-skills

Skills Claude Code maintenues par [@tatianavizar](https://github.com/tatianavizar).

## Skills

| Skill | Rôle |
|---|---|
| [`scenarios-test-fonctionnels`](scenarios-test-fonctionnels/) | Génère des scénarios de test fonctionnels end-to-end pour une feature, en croisant le code du repo (comportement réel) avec le brief et les tickets (comportement attendu). |

## Installation

Bundle prêt à installer dans [`dist/`](dist/) :

```bash
# Skill utilisateur (toutes les sessions)
unzip -o dist/scenarios-test-fonctionnels.skill -d ~/.claude/skills/

# ou skill projet (ce repo uniquement)
unzip -o dist/scenarios-test-fonctionnels.skill -d .claude/skills/
```

Depuis les sources, sans passer par le bundle :

```bash
cp -R scenarios-test-fonctionnels ~/.claude/skills/
```

## Conventions

Chaque skill suit la même structure :

```
<nom-de-la-skill>/
  SKILL.md              # frontmatter + workflow — chargé à chaque déclenchement, à garder court
  references/*.md       # règles détaillées, checklist de livraison — lus à l'étape qui les concerne
  assets/*.md           # template de sortie + exemple conforme
```

Règles appliquées :

- **`SKILL.md` reste mince.** Une règle = une ligne. Le *pourquoi* et les pièges vont dans `references/`, lus seulement à l'étape qui en a besoin.
- **Source unique.** Une règle est énoncée à un seul endroit. Les templates ne contiennent que des placeholders, pas une recopie des règles.
- **Un exemple conforme par skill.** Un template vide ne transmet pas la granularité attendue.
- **La checklist finale renvoie aux règles**, elle ne les duplique pas ; elle n'ajoute que les vérifications transverses.
- **`description` en frontmatter** = quoi + quand, avec les formulations indirectes et les déclencheurs FR *et* EN.
- **`allowed-tools`** limité au strict nécessaire (lecture + écriture du livrable, pas de Bash arbitraire).

## À faire avant publication

- **Tampon de version dans le livrable** — une ligne `Skill : vN` dans le bloc Contexte du fichier généré, incrémentée à chaque évolution des règles. Sans elle, un fichier de recette relu plus tard n'indique pas à quelles règles il obéit. À poser au moment de figer la première version publiée, pas avant.
- **Premier run réel de bout en bout** — les règles actuelles ont toutes été écrites en auditant des sorties produites par des versions antérieures. Aucune n'a encore tourné dans la version en place.

## Repackager un bundle

```bash
cd <nom-de-la-skill>/.. && zip -r -X ../dist/<nom-de-la-skill>.skill <nom-de-la-skill> -x '*.DS_Store'
```

## Licence

© Tatiana Villamizar Mojica — tous droits réservés. Voir [LICENSE](LICENSE).

Ces skills sont une conception personnelle : le workflow, les règles de rédaction et les exemples sont de l'autrice. Repo public pour référence et portfolio, pas une cession de droits — pas de réutilisation ou de redistribution sans accord écrit.
