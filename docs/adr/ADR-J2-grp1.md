# ADR-001 — Choix du backend LLM pour la génération de QCM

**Statut :** Accepté par l'équipe — à valider avec le PO
**Date :** 2026-06-30
**Contexte :** Perturbation J2 — latence inacceptable
**Feature concernée :** F3 — Génération automatique de 10 QCM via LLM
**Référence :** Compte rendu `docs/cadrage/benchmark-llm.md` (benchmark des 8 backends — source de vérité)

## 1. Contexte

Le modèle actuellement configuré pour générer les QCM est **Llama 3.1 8B Instruct via Ollama local**. Ce choix est gratuit, local et cohérent avec l'objectif de souveraineté des données, mais il pose un problème majeur de performance : sur CPU, la génération d'un quiz prend **2 à 5 minutes** (mesuré à **P50 ≈ 137 s** sur machine sans GPU, cf. benchmark §4.1). Son JSON est en outre parfois invalide, ce qui déclenche un HTTP 502 visible pour l'utilisateur final.

La génération de quiz est une fonctionnalité centrale d'un produit destiné à de **vrais utilisateurs**. Une latence de plusieurs minutes est inacceptable en production et dégrade directement l'expérience. Le cours peut par ailleurs contenir des **données personnelles** : le choix du fournisseur a donc aussi une dimension RGPD. L'équipe doit choisir un backend LLM adapté à la mise en production, en tenant compte de la qualité du JSON, de la performance, du coût récurrent, du RGPD et de l'effort d'intégration.

## 2. Options envisagées

| Option | Avantages | Inconvénients | Verdict |
|---|---|---|---|
| Garder **Llama 3.1 8B via Ollama** | Gratuit, local, souverain, aucun transfert de données | Latence rédhibitoire (P50 ≈ 137 s CPU), JSON parfois invalide | Rejeté comme choix principal |
| Utiliser **Mistral `mistral-small-latest`** | **Seul fournisseur cloud hébergé en UE 🇫🇷** (RGPD by design), bon JSON natif, rapide (~3 s), coût faible (~$0.15/1M), offre payante avec SLA, déjà câblé par le factory pattern | Dépendance cloud + réseau, un cran sous Cerebras en latence/qualité brute | **Retenu comme choix principal** |
| Utiliser **Cerebras `gpt-oss-120b`** (ou Groq 70B) | Le plus rapide du marché (~2 s), qualité ≫ 8B, free tier généreux | Données envoyées **hors UE** (transfert à justifier RGPD), free tier sans SLA | Alternative performance |
| Utiliser **Qwen 3 8B via Ollama** | Gratuit, local, souverain, meilleur suivi d'instructions que Llama 3.1 8B | Latence CPU toujours problématique → exige un GPU | Fallback souverain |

## 3. Décision retenue

Nous décidons d'utiliser **Mistral `mistral-small-latest`** comme backend principal pour la génération de QCM.

Ce choix suit la recommandation n°1 du benchmark (`docs/cadrage/benchmark-llm.md` §6) : c'est le meilleur compromis pour un produit en production destiné à de vrais utilisateurs européens. Le backend local **Ollama** reste conservé comme solution de repli (`fallback`), **Cerebras `gpt-oss-120b` / Groq 70B** est documenté comme alternative si la priorité devient la performance brute (résidence des données non bloquante), et **Qwen 3 8B** comme option 100 % souveraine si le PO impose que rien ne quitte notre infrastructure.

## 4. Justification

Le modèle actuel est cohérent sur le plan souveraineté mais sa latence (P50 ≈ 137 s) et ses JSON parfois invalides le disqualifient comme choix par défaut. Parmi les alternatives mesurées, Mistral apporte le meilleur équilibre pour la **mise en production** :

- **RGPD by design** : seul fournisseur cloud hébergé en UE 🇫🇷 → **pas de transfert hors UE à justifier**, alors que le cours peut contenir des données personnelles. C'est le critère décisif pour un produit à vrais utilisateurs européens.
- **Qualité & fiabilité JSON** : bon JSON natif, latence de l'ordre de la seconde → suppression des 502 dus au 8B local.
- **Coût & SLA** : coût faible (~$0.15/1M) et fournisseur proposant une **offre payante avec engagement de service** — indispensable en prod.
- **Effort d'intégration nul** : le kit expose déjà 9 backends via un *factory pattern*. Changer de modèle = modifier `LLM_BACKEND` / `*_MODEL` dans `.env`, aucun code à écrire — la bascule est **feature-flaggée** par configuration.

Cerebras/Groq restent plus rapides et sont retenus **en alternative** : si le PO acte que la résidence des données n'est pas bloquante, la bascule ne coûte qu'une ligne de configuration.

## 5. Conséquences

### Positives

- Génération de QCM rapide (de ~2 min à ~3 s) et exploitable en production.
- **Conformité RGPD by design** : données traitées en UE, pas de transfert hors UE à documenter pour le choix principal.
- JSON plus fiable → moins de HTTP 502 visibles pour l'utilisateur.
- Bascule et *fallback* peu coûteux grâce à la configuration existante des backends LLM.

### Négatives

- Dépendance à un fournisseur externe et à la connexion réseau.
- Free tier sans garantie de disponibilité ni SLA → réservé au dev/staging.
- Un cran sous Cerebras/Groq en latence et qualité brute.

### À surveiller

- Conserver la validation stricte du JSON généré côté backend (`parse_and_validate_quiz`).
- Prévoir un *fallback* `ollama` ou `mock` en cas d'indisponibilité de Mistral.
- Passer à l'**offre payante avec SLA** avant la mise en production réelle.
- Mesurer après bascule la latence réelle (P50/p95) sur le jeu de cours de référence et vérifier que le temps de génération respecte CA3 (< 60 s, objectif ~quelques secondes).
- Si le PO bascule sur Cerebras/Groq (hors UE), **tracer le transfert hors UE** dans le registre de traitements + DPA fournisseur.
