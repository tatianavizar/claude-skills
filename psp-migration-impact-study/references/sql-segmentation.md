# Requêtes SQL de segmentation (à adapter)

Ces requêtes viennent d'une base Rails/PostgreSQL réelle. **Adapte les noms de
tables et colonnes** au projet : vérifie toujours le schéma (`db/schema.rb` ou
équivalent) avant de lancer. Donne ces requêtes au PM pour qu'il les exécute ;
ne te connecte pas à la base de production toi-même.

Hypothèses de nommage dans les exemples (remplace par les vrais) :
- `users_profiles` : profils investisseurs, avec `type` (STI :
  `Natural`/`Legal`), `wallet_id`, `kyc_validated_at`, `lw_onboarding_status`,
  `wallet_balance_in_cents`, `user_id`.
- `users` : `first_name`, `last_name`, `birthdate`, `native_country`,
  `tax_residency_country`.
- `payment_operations` : `owner_type`/`owner_id` (polymorphe), `category`,
  `status`, `created_at`. (Noms de catégories/statuts en **chaîne**.)
- `subscriptions` : `users_profile_id`, `project_id`, `amount` (en euros),
  `paid_at`, `canceled_at`.
- `projects` : `status` (`draft/published/closed/succeeded/failed/canceled/reimbursed`).
- `sdd_mandates` ⋈ `bank_accounts` (owner polymorphe) : `status` (chaîne).
- Échéanciers : `lending_investor_terms` (`subscription_id`, `due_on`, `paid_at`,
  `remaining_capital_end_of_period`, `remaining_capital_beginning_of_period`).

## Définitions retenues (à expliciter dans le socle)

- **Validé** = `kyc_validated_at IS NOT NULL` (proxy ; peut diverger du statut
  d'onboarding).
- **Actif opérationnel** = ≥1 opération nécessitant un KYC validé (investir /
  retirer) réussie depuis une date pivot (ex. mise en production du nouvel
  onboarding).
- **Receveur futur** = souscription payée non annulée dans un projet non encore
  remboursé (versements à venir).
- **Wallet à maintenir** = actif ∪ receveur futur ∪ solde > 0.
- **Preuve de vivant (liveness)** = déterministe via le statut d'onboarding si le
  nouveau tunnel (V3) inclut toujours un liveness et l'ancien (V2) jamais.

## 1. Volumes & statuts (type × validation)

```sql
SELECT up.type,
       CASE WHEN up.kyc_validated_at IS NOT NULL THEN 'valide' ELSE 'non_valide' END AS statut_kyc,
       COUNT(*) AS comptes
FROM users_profiles up
WHERE up.type IN ('Users::NaturalProfile','Users::LegalProfile')
GROUP BY up.type, statut_kyc
ORDER BY up.type, statut_kyc;
```

## 2. Segmentation « wallet à maintenir » vs dormants (+ ROLLUP total)

```sql
WITH actifs_ope AS (
  SELECT DISTINCT owner_id AS profile_id
  FROM payment_operations
  WHERE owner_type = 'Users::Profile'
    AND category IN ('subscription','debit_account')  -- investir OU retirer
    AND status   = 'succeeded'
    AND created_at >= '2025-10-01'                     -- date pivot
),
receveurs_futurs AS (
  SELECT DISTINCT s.users_profile_id AS profile_id
  FROM subscriptions s
  JOIN projects p ON p.id = s.project_id
  WHERE s.users_profile_id IS NOT NULL
    AND s.paid_at   IS NOT NULL
    AND s.canceled_at IS NULL
    AND p.status NOT IN ('draft','failed','canceled','reimbursed')
)
SELECT
  COALESCE(up.type,'TOTAL')                                            AS type,
  COUNT(*)                                                             AS valides,
  COUNT(*) FILTER (WHERE ao.profile_id IS NOT NULL)                    AS actifs_ope,
  COUNT(*) FILTER (WHERE rf.profile_id IS NOT NULL)                    AS receveurs_futurs,
  COUNT(*) FILTER (WHERE COALESCE(up.wallet_balance_in_cents,0) > 0)   AS avec_solde,
  COUNT(*) FILTER (WHERE ao.profile_id IS NOT NULL
                      OR rf.profile_id IS NOT NULL
                      OR COALESCE(up.wallet_balance_in_cents,0) > 0)   AS wallet_a_maintenir,
  COUNT(*) FILTER (WHERE ao.profile_id IS NULL
                     AND rf.profile_id IS NULL
                     AND COALESCE(up.wallet_balance_in_cents,0) = 0)   AS dormants
FROM users_profiles up
LEFT JOIN actifs_ope       ao ON ao.profile_id = up.id
LEFT JOIN receveurs_futurs rf ON rf.profile_id = up.id
WHERE up.type IN ('Users::NaturalProfile','Users::LegalProfile')
  AND up.kyc_validated_at IS NOT NULL
GROUP BY ROLLUP (up.type)
ORDER BY up.type NULLS LAST;
```

## 3. Preuve de vivant sur toute la base validée (proxy liveness)

Cadre l'analyse sur **tous les validés**, pas seulement les actifs (demande
fréquente de la direction).

```sql
SELECT
  CASE WHEN up.lw_onboarding_status = 'accepted'          THEN 'V3 (liveness present)'
       WHEN COALESCE(up.lw_onboarding_status,'') = ''      THEN 'legacy V2 (pas de liveness)'
       ELSE 'V3 en cours / incomplet' END                 AS statut_liveness,
  COUNT(*) AS comptes
FROM users_profiles up
WHERE up.type IN ('Users::NaturalProfile','Users::LegalProfile')
  AND up.kyc_validated_at IS NOT NULL
GROUP BY statut_liveness
ORDER BY comptes DESC;
```

Croise ensuite les deux axes (activité × liveness) dans un tableau à double entrée.
Ne les additionne pas : ce sont des populations différentes qui se recoupent.

## 4. Résidence fiscale regroupée

Buckets national / UE hors national / hors UE / US. Convention à expliciter (ex.
territoires d'outre-mer rattachés ou non au national selon le régime fiscal).

```sql
SELECT
  CASE
    WHEN u.tax_residency_country = 'FR'                         THEN 'FR'
    WHEN u.tax_residency_country IN ('GP','MQ','GF','RE','YT')  THEN 'FR'        -- DOM
    WHEN u.tax_residency_country = 'US'                         THEN 'US'
    WHEN u.tax_residency_country IN
      ('AT','BE','BG','HR','CY','CZ','DK','EE','FI','DE','GR','HU','IE','IT',
       'LV','LT','LU','MT','NL','PL','PT','RO','SK','SI','ES','SE')           THEN 'UE hors FR'
    WHEN COALESCE(u.tax_residency_country,'') = ''              THEN '(non renseigne)'
    ELSE 'Hors UE'
  END AS zone_fiscale,
  COUNT(*) AS comptes
FROM users_profiles up
JOIN users u ON u.id = up.user_id
WHERE up.type IN ('Users::NaturalProfile','Users::LegalProfile')
  AND up.kyc_validated_at IS NOT NULL
GROUP BY zone_fiscale
ORDER BY comptes DESC;
```

Attention : « US par résidence fiscale » sous-compte les **US persons** (un citoyen
US résidant ailleurs reste US person). Ne conclus pas « périmètre marginal » sans le
dire ; le statut US person réel est souvent inconnu et à collecter.

## 5. Mandats de prélèvement actifs

```sql
SELECT COUNT(*) AS mandats_actifs
FROM sdd_mandates
WHERE status IN ('5','6');   -- adapter aux codes "actifs" du projet (chaîne)
```

## 6. Multi-wallets par compte

Depuis la base (approximation, wallets trackés) :

```sql
WITH w AS (
  SELECT up.id,
         (CASE WHEN up.wallet_id IS NOT NULL THEN 1 ELSE 0 END)
         + COUNT(DISTINCT s.wallet_id) AS nb_wallets
  FROM users_profiles up
  LEFT JOIN subscriptions s ON s.users_profile_id = up.id AND s.wallet_id IS NOT NULL
  WHERE up.type IN ('Users::NaturalProfile','Users::LegalProfile')
  GROUP BY up.id, up.wallet_id
)
SELECT ROUND(AVG(nb_wallets),1) AS moy, MAX(nb_wallets) AS max,
       SUM(nb_wallets) AS wallets_total, COUNT(*) AS comptes
FROM w;
```

Pour le **vrai** compte (y compris wallets orphelins) et la distinction actif vs
fermé, demander au PSP source un export CSV : par wallet, l'identifiant, la clé de
regroupement compte, le statut (ouvert/fermé), le solde. Le « wallet actif » facturé
relève de la négociation commerciale client/PSP, pas de la mission technique.

## 7. Capital en cours de remboursement (le vrai enjeu financier)

`subscriptions.amount` ou le solde wallet ne sont **pas** le capital en jeu. Le
capital restant dû se calcule sur l'échéancier. Filtre par **CRD > 0 par
souscription** (robuste même si les échéances futures ne sont pas encore générées) :

```sql
WITH dernier_paye AS (
  SELECT DISTINCT ON (subscription_id)
         subscription_id, remaining_capital_end_of_period AS crd
  FROM lending_investor_terms
  WHERE paid_at IS NOT NULL
  ORDER BY subscription_id, due_on DESC, id DESC
),
capital_initial AS (
  SELECT DISTINCT ON (subscription_id)
         subscription_id, remaining_capital_beginning_of_period AS crd
  FROM lending_investor_terms
  ORDER BY subscription_id, due_on ASC, id ASC
),
crd_par_souscription AS (
  SELECT s.id AS subscription_id, s.users_profile_id,
         COALESCE(dp.crd, ci.crd, 0) AS crd
  FROM subscriptions s
  JOIN projects p ON p.id = s.project_id
  LEFT JOIN dernier_paye    dp ON dp.subscription_id = s.id
  LEFT JOIN capital_initial ci ON ci.subscription_id = s.id
  WHERE s.paid_at IS NOT NULL AND s.canceled_at IS NULL
    AND p.status NOT IN ('draft','failed','canceled','reimbursed')
    AND EXISTS (SELECT 1 FROM lending_investor_terms t WHERE t.subscription_id = s.id)
)
SELECT COUNT(*) AS souscriptions_crd_positif,
       COUNT(DISTINCT users_profile_id) AS investisseurs_concernes,
       ROUND(SUM(crd), 2) AS capital_en_cours
FROM crd_par_souscription
WHERE crd > 0;
```

Vérifie l'unité (€ vs centimes) par l'ordre de grandeur : `capital_en_cours` divisé
par le nombre de souscriptions doit donner un montant plausible par souscription.

Important à expliquer au client : ce capital **n'est pas forcément sur la
plateforme** aujourd'hui. Si les porteurs de projet ont déjà retiré les fonds et
re-créditent au moment de chaque remboursement, ce montant est un **volume de flux
futurs** qui transitera après migration. L'enjeu est donc que les wallets des
investisseurs concernés restent fonctionnels pour encaisser ces remboursements.
