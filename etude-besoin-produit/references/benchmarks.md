# Référence — Benchmarks externes

Tes chiffres internes n'ont de sens que **situés** par rapport au marché. Objectif : dire si une
hypothèse est haute ou basse, avec des sources vérifiables.

## Méthode
1. **Priorité au repère INTERNE** : ta propre data historique (cohortes) bat tout benchmark
   générique. « Notre app doit lever un taux organique déjà à X % » > « le marché dit Y % ».
2. **Benchmarks externes** = pour ce que tu n'as pas en interne (surtout le push/app quand il
   n'existe pas encore). Lancer 1-2 recherches web ciblées (sous-agents), exiger pour CHAQUE
   chiffre : **valeur · source (nom) · URL exacte · année · (générique vs secteur)** + niveau de
   fiabilité : ✅ page lue · ⚠️ agrégateur secondaire · 🟠 marketing/vendeur.
3. **Ne pas mélanger** trois choses différentes : taux de **conversion** (ratio), **uplift de
   revenu causal** (gain), écarts **LTV/fréquence descriptifs** (biais de sélection).

## Bibliothèque de départ (repères trouvés, fintech / e-commerce, 2024-2025)
À revérifier avant diffusion externe — niveaux de fiabilité indiqués.

| Sujet | Repère | Source | Fiab. |
|---|---|---|---|
| Opt-in push | iOS ~48-56 % · Android ~50-67 % (éviter le vieux « 91 % ») | Batch 2025 (FR), OneSignal 2024 (finance) | ✅ |
| CTR push finance | ~7-9 % (parmi les meilleurs secteurs) ; ciblé ~14 % vs blast ~4 % | Batch 2025, CleverTap | ✅ |
| Rétention via push | opt-in ≈ ×2 rétention ; +26 % sur 1ʳᵉ transaction (multicanal) | Airship, CleverTap 2024 | ✅ |
| App vs web mobile (conversion) | ~1,8× à 3× (médian ~3×) ; finance ~18 % payeurs J30 web-to-app | Poq, Criteo, AppsFlyer | ✅/⚠️ |
| **Uplift de revenu au lancement d'une app** | **+12 à +21 %** (e-commerce ; ~+12,5 % net prudent) | Tapcart *Incrementality Index* 2025 | ✅ (éditeur, biais) |
| Parrainage (valeur) | parrainés **+25 % CLV · −18 % churn** (étude banque) | Schmitt–Skiera–Van den Bulte, *J. of Marketing* 2011 | ✅ peer-reviewed |
| Parrainage (conv.) | 3-5 % (top quartile 8 %+) ; CAC ÷3-5 | ReferralCandy, GrowSurf | 🟠 marketing |
| Auto-invest / récurrent | **pas de benchmark public d'adoption** ; néobrokers en font un produit de masse (Trade Republic) | — | — |

## À manier avec prudence
- Les % d'éditeurs referral (Mention Me, GrowSurf, Extole) = fourchette optimiste, pas fait établi.
- Les stats push « historiques » (61 % vs 28 % type Localytics) = anciennes, ordre de grandeur.
- Les multiples LTV « app vs web » (jusqu'à ×5) = biais de sélection, spectaculaires mais trompeurs.

## Honnêteté
S'il n'existe **pas** de benchmark direct pour ton cas (fréquent : « uplift de collecte quand une
plateforme fintech ajoute une app »), le dire, et présenter le repère le plus proche comme un
**proxy** (ex. uplift app e-commerce +12-21 %) pour **situer** ton hypothèse — en la gardant
nettement en dessous = prudente.
