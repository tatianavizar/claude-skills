# Référence — Data interne (GA + SQL/Metabase)

Prouver le besoin = partir de la data réelle, pas d'opinions. Deux sources complémentaires.

## A. Google Analytics (GA4)

**Prérequis** : demander la liste des events/conversions trackés (sans events de funnel, pas de
mesure de complétion). Période : **12 mois glissants**. Noter le **taux de consentement cookies**
(GA sous-compte → données minorantes). Filtrer trafic interne/bots. Comparer **par device**.

| Export | Quoi | Sortie utile |
|---|---|---|
| **A — Device/OS** | Rapport Technologie · dim. Device category + OS + Mois | part mobile %, split iOS/Android, tendance |
| **B — Entonnoirs par device** ⭐ | Exploration entonnoir, breakdown Device category | taux de complétion + points de chute mobile vs desktop (= l'argument clé) |
| **C — Comportement/acquisition** | nouveaux vs connus · source/medium · events clés | proxy intentions, contribution du canal email |
| **D — Contribution email** | Acquisition, medium = email | perf du canal actuel même sans accès à l'ESP |

**À retenir** : GA = **ratios** (device, conversion relative). Pas les volumes absolus (il sous-compte).
Chaque entonnoir ne vaut que si les events des étapes existent — sinon, le mesurer est impossible.

## B. Base SQL / Metabase

GA ne connaît pas la vérité métier (€, souscriptions exactes, fidélité). Ces requêtes sont des
**patterns à adapter** au schéma du client. Toujours confirmer les noms de tables/colonnes + unités
(€ vs cents) + le statut qui « autorise » l'action avant de chiffrer.

### Photo du gisement (segments = cibles de réactivation)
```sql
WITH act AS (  -- activité par user (adapter: souscriptions payées, commandes, etc.)
  SELECT u.id AS user_id,
         count(a.id) FILTER (WHERE a.done_at IS NOT NULL AND a.canceled_at IS NULL) AS nb_actes
  FROM users u LEFT JOIN actes a ON a.user_id = u.id GROUP BY u.id
)
SELECT count(*) AS total,
       count(*) FILTER (WHERE nb_actes > 0)  AS actifs,
       count(*) FILTER (WHERE nb_actes = 0)  AS jamais_actifs,
       count(*) FILTER (WHERE nb_actes >= 2) AS recurrents
FROM act;
```
+ croiser avec le statut « éligible » (ex. KYC validé) pour isoler les **dormants qualifiés** :
éligibles mais jamais passés à l'acte = la cible la plus directe.

### Fidélité par cohorte (le meilleur benchmark = ta propre data)
% qui refont un 2ᵉ acte dans les 12 mois de leur 1er, par cohorte d'année. Donne le **taux
organique** — ton produit doit *lever* ce taux, pas le créer.
```sql
WITH f AS (SELECT user_id, min(done_at) AS first FROM actes
           WHERE done_at IS NOT NULL AND canceled_at IS NULL GROUP BY user_id)
SELECT date_part('year', f.first)::int AS cohorte, count(*) AS n,
  round(100.0*count(*) FILTER (WHERE EXISTS (
    SELECT 1 FROM actes a WHERE a.user_id=f.user_id
      AND a.done_at > f.first AND a.done_at < f.first + interval '12 months'))/count(*),1) AS pct_2e_12m
FROM f GROUP BY 1 ORDER BY 1;
```

### Ticket / valeur unitaire (médiane, pas moyenne)
```sql
SELECT count(*) AS n,
  round(avg(amount)::numeric,0)                                         AS moyen,
  round(percentile_cont(0.5) WITHIN GROUP (ORDER BY amount)::numeric,0) AS median,
  min(amount), max(amount)
FROM actes WHERE done_at IS NOT NULL AND canceled_at IS NULL;
```
Pour une **activation** (1er acte), utiliser le **premier ticket** (row_number sur date, rn=1).
Retenir la **médiane** pour le business case (la moyenne est gonflée par les gros tickets).

### Revenu & marge
Collecte/CA par an + **marge réelle du client** (×taux). ⚠️ Le business case se raisonne sur la
marge, pas le volume. Si la marge est « connue en off », la marquer HYPOTHÈSE.

### Features sous-exploitées (gisement d'upsell)
Compter l'adoption d'une feature à forte valeur (abonnement récurrent, parrainage, auto-X) et son
montant moyen → candidat levier. Vérifier le **dénominateur** (vs base totale ou vs actifs).

## Pièges classiques (vus en vrai)
- Une colonne « cache » (`cached_*`) souvent vide → prendre la source réelle.
- Un gros compteur (ex. 144 actes) = power-user légitime OU compte test OU mécanique auto →
  diagnostiquer (top-20, scoper 12 mois, exclure l'automatique) avant de conclure.
- Un marqueur « legacy » (ancien système de validation) gonfle un segment → utiliser le statut
  **qui autorise réellement** l'action aujourd'hui.
- Une table « intention » peut être une file d'allocation technique, pas un panier abandonné →
  vérifier le cycle de vie dans le code avant d'en faire un segment.
