# Compte rendu — Benchmark LLM (choix du modèle de génération de QCM)

**Statut** : 🟡 Proposition (à valider avec le PO)
**Date** : 2026-06-30
**Périmètre** : remplacer / confirmer le modèle qui génère les 10 QCM (feature F3)
**Tâche évaluée** : à partir d'un cours (≤ 8000 caractères), produire **un JSON strict
de 10 questions × 4 options × 1 bonne réponse**, **en français**.

---

## 1. Point de départ — le modèle actuel

| Élément | Valeur | Source dans le repo |
|---|---|---|
| Modèle | **Llama 3.1 8B Instruct** (`llama3.1:8b`) | `.env.example:29` |
| Backend | `ollama` (local, gratuit) | `.env.example:25` |
| Paramètres | `temperature=0.4`, sortie `format=json` | `backend/llm/services/ollama_client.py:48-50` |
| Garde-fou | validation stricte → HTTP 502 si JSON invalide | `backend/llm/services/quiz_prompt.py:61` |

> 💡 Le kit expose **9 backends** via un *factory pattern* (`backend/llm/services/factory.py:39`).
> Changer de modèle = modifier `LLM_BACKEND` / `*_MODEL` dans `.env`. **Aucun code à écrire.**

**Limite constatée** : un 8B sur CPU met **2 à 5 min** par génération (cf. perturbation J2 « latence »),
et son JSON est parfois bancal (questions tronquées, options manquantes → 502).

---

## 2. Critères de décision (pondérés pour CE projet)

| Critère | Poids | Pourquoi |
|---|---|---|
| Qualité (JSON strict, FR, factualité) | ★★★★ | C'est la valeur produit. Un 502 = écran d'erreur pour l'étudiant. |
| Performance (latence/débit) | ★★★★ | Perturbation J2. Démo devant jury = pas d'attente de 3 min. |
| Coût | ★★★ | Projet étudiant → privilégier gratuit / free tier. |
| RGPD / souveraineté | ★★★ | Perturbation J3-bis : le cours peut être une donnée personnelle. |
| Effort d'intégration | ★★ | Tous les candidats sont **déjà câblés** → effort quasi nul. |

---

## 3. Tableau comparatif (données juin 2026)

| Modèle (backend) | Qualité | Débit | Coût /1M (in/out) | RGPD | Gratuit |
|---|---|---|---|---|---|
| **Llama 3.1 8B** — `ollama` *(actuel)* | ★★ | 🔴 2–5 min/CPU | Gratuit | 🟢 Local UE | ✅ |
| **gpt-oss-120b** — `cerebras` | ★★★★ | 🟢🟢🟢 ~2000 t/s | Free tier **1M tok/jour** | 🔴 hors UE | ✅ |
| **Llama 3.3 70B** — `groq` | ★★★★ | 🟢🟢 ~250 t/s | $0.59 / $0.79 (free tier) | 🔴 hors UE | ✅ |
| **Mistral Small** — `mistral` | ★★★ | 🟢 rapide | ~$0.15 / $0.40 (free tier) | 🟢🟢 **UE 🇫🇷** | ✅ |
| **Gemini 2.5 Flash** — `gemini` | ★★★★ | 🟢 rapide | $0.30 / $2.50 (free tier réduit) | 🔴 hors UE | ⚠️ ~250 req/j |
| **`qwen3:8b`** — `ollama` | ★★★ | 🔴 CPU | Gratuit | 🟢 Local UE | ✅ |
| **GPT-4o-mini** — `openai` | ★★★★ | 🟢 rapide | $0.15 / $0.60 | 🔴 hors UE | ❌ payant |

> Plafond de contexte « free tier » Cerebras = **8192 tokens**, soit ~la limite de 8000
> caractères du kit (`quiz_prompt.py:24`). Parfait pour ce cas d'usage.

---

## 4. Trois scénarios selon la priorité du PO

### Scénario A — « Démo qui claque » (qualité + vitesse, RGPD secondaire)
➡️ **Cerebras `gpt-oss-120b`** (ou `llama-3.3-70b`)
Gratuit (1M tok/jour), **le plus rapide du marché**, qualité ≫ 8B. Génération en ~2 s.
*Alternative équivalente :* Groq `llama-3.3-70b-versatile`.

### Scénario B — « Souverain mais moderne » (RGPD prioritaire)
➡️ **Mistral `mistral-small-latest`**
**Seul fournisseur cloud hébergé en UE 🇫🇷.** Bon JSON natif, rapide, free tier.
L'argument défendable à l'oral sur la perturbation J3-bis.

### Scénario C — « Zéro donnée ne sort du serveur » (souveraineté absolue)
➡️ **Ollama `qwen3:8b`** (upgrade local du modèle actuel)
Gratuit, 100 % local, meilleur suivi d'instructions/JSON que Llama 3.1 8B.
Latence CPU inchangée → à coupler avec une UX (spinner) ou `llama3.2:3b` si RAM/latence critiques.

---

## 5. Recommandation

Pour un projet **étudiant noté en une semaine**, et pour maximiser l'effet démo
tout en montrant un arbitrage :

1. **Choix principal : Cerebras `gpt-oss-120b`** (Scénario A) — gratuit, rapide, qualité, et son
   plafond de contexte tombe pile sur la limite du kit.
2. **À documenter comme alternative : Mistral Small** (Scénario B) pour l'axe RGPD.
3. **Repli souverain : `qwen3:8b` local** (Scénario C) si la contrainte « données » durcit en J3-bis.

> Ce qui est noté, ce n'est pas le modèle « gagnant » mais **le fait d'avoir arbitré**
> qualité / performance / coût / souveraineté. Ce tableau est le matériau de l'ADR attendu en J2.

---

## 6. Risques à mentionner (le jury aime les voir identifiés)

- **Free tier ≠ SLA** : quotas mouvants, pas de garantie de dispo. Prévoir un *fallback* `mock` ou `ollama`.
- **RGPD** : tout backend cloud hors UE envoie le cours hors UE (sauf Mistral). À tracer dans le registre de traitements.
- **Validation post-LLM** : garder `parse_and_validate_quiz` quel que soit le modèle — c'est elle qui protège du prompt injection (J3).
- **Dépendance externe** : un modèle cloud rend la démo tributaire du réseau ; Ollama reste le filet local.

---

## 7. Étape suivante recommandée

Lancer un **mini-benchmark réel** (même cours envoyé à 2-3 backends) mesurant :
**latence réelle** + **taux de JSON valide** (% de générations qui passent la validation).
Cela remplace les estimations de ce tableau par des chiffres reproductibles pour le rapport.

---

### Sources (consultées le 2026-06-30)
- Groq — <https://groq.com/pricing>
- Cerebras free tier — <https://www.getaiperks.com/en/ai/cerebras-free-tier-guide> · limites <https://inference-docs.cerebras.ai/support/rate-limits>
- Mistral — <https://mistral.ai/pricing/>
- Gemini — <https://ai.google.dev/gemini-api/docs/rate-limits>
- OpenRouter (modèles `:free`) — <https://costgoat.com/pricing/openrouter-free-models>
- OpenAI — <https://developers.openai.com/api/docs/pricing>
</content>
</invoke>
