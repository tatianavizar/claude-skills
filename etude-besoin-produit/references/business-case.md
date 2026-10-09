# Référence — Business case bottom-up

Objectif : un ROI **traçable ligne par ligne**, défendable face à un décideur. Jamais un uplift
global sorti du chapeau.

## Principe : chaque levier est déroulé
`Collecte/revenu incrémental = DONNÉE RÉELLE × UN SEUL TAUX (hypothèse justifiée) × VALEUR UNITAIRE`

Pour chaque levier, afficher :
- **Donnée réelle** (en gras, sourcée DB/GA) — le volume de la cible.
- **Taux appliqué** (hypothèse, prudente, idéalement étayée par un benchmark ou la cohorte interne).
- **Valeur unitaire** = ticket **médian**.
- **= résultat/an.**

Exemple (rendre l'arithmétique visible, pas un calcul caché) :
> L1 Fréquence — 5 608 récurrents × **+0,3 acte/an** = 1 682 actes de plus × 200 € = **336 k€/an**
> (le « +0,3 » = « ~1 récurrent sur 3 fait 1 acte de plus/an », +8 % vs les 3,7 actes/an actuels)

Typologie de leviers (adapter) : **fréquence** (raccourcir le cycle), **mono→multi** (convertir le
résidu qui ne revient pas), **activation** (dormants qualifiés + conversion mobile récupérée),
**feature sous-exploitée** (auto-X, abonnement), **parrainage** (acquisition qualifiée).

## Scénarios = faire varier le TAUX-CLÉ (pas ×0,5/×1,7 arbitraire)
Trois colonnes : défavorable / base / favorable, chacune = une valeur explicite du taux de chaque
levier. Le total ÷ collecte annuelle = **le % d'uplift**.

## ⚠️ Raisonner sur le revenu du CLIENT, pas le volume
Le gain du client = collecte incrémentale **× sa marge/commission** (ex. 5 %). Oublier ça gonfle le
ROI d'un facteur énorme. Si la marge est informelle → **HYPOTHÈSE à confirmer**.

## Projection sur la croissance (pas sur une année figée)
1. Base de référence explicite = dernier exercice complet (ex. collecte N-1).
2. Projeter sur **3 scénarios de croissance** — tous **sous** la croissance déjà constatée
   (= conservateur). Appliquer l'uplift base à la collecte projetée.
3. `Revenu client/an = collecte projetée × uplift × marge`. Montrer **un calcul-exemple** en clair.
4. **Payback = coût de build ÷ revenu client/an** (cumulé si besoin).

## Mise en perspective du coût
Rapporter le build à **une échelle parlante** : « X k€ = ~N % d'une seule année de marge ». Un payback
de ~1 an sur un coût marginal = argument fort, sans avoir besoin d'un effet spectaculaire.

## Lecture stratégique (registre conseil — 4 temps)
1. **Ce qui est solide** (besoin mesuré, coût modéré, retour ~1 an même prudent).
2. **Ce qui conditionne le résultat** (les 1-2 hypothèses porteuses + dépendance à l'exécution client).
3. **Quand ce ne serait PAS le bon investissement** (honnêteté = crédibilité).
4. **Notre lecture** (reco mesurée ; souvent : *piloter/tester avant de committer le build*).

Ne jamais finir sur un « GO ! » commercial. La décision appartient au client ; le rôle = éclairer.

## Inputs à réclamer avant de figer
- **Coût de build** (chiffrage lead dev).
- **Marge/commission** du client (vraie, pas en off).
- **Collecte/CA de référence** (dernier exercice).
Manquants → hypothèse marquée *ILLUSTRATIF* + scénarios ; signaler que 2-3 chiffres figeront le modèle.
