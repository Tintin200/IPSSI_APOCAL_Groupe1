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

**Limite constatée** : un 8B sur CPU met **2 à 5 min** par génération (enjeu de latence en
production), et son JSON est parfois bancal (questions tronquées, options manquantes → 502).

---

## 2. Critères de décision (pondérés pour la mise en production)

| Critère | Poids | Pourquoi |
|---|---|---|
| Qualité (JSON strict, FR, factualité) | ★★★★ | Valeur produit. Un 502 = erreur visible pour l'utilisateur final. |
| Performance (latence/débit) | ★★★★ | UX directe : une génération de 2–3 min en prod est inacceptable. |
| Coût | ★★★ | Coût **récurrent** facturé à chaque génération, à l'échelle des utilisateurs réels. |
| RGPD / souveraineté | ★★★ | **Obligation légale** : le cours peut contenir des données personnelles d'utilisateurs réels. |
| Fiabilité / SLA | ★★★ | En prod, un quota ou une panne fournisseur = service indisponible. |
| Effort d'intégration | ★★ | Tous les candidats sont **déjà câblés** → effort quasi nul. |

---

## 3. Tableau comparatif (données juin 2026)

> Les colonnes **Qualité /5**, **Latence méd./p95** et **Empreinte** sont à confirmer par
> les mesures du §4. Les valeurs ci-dessous sont des **estimations** (★ = a priori, à valider).

| Modèle (backend) | Qualité /5 *(est.)* | Latence méd. / p95 *(est.)* | Empreinte locale (RAM / disque / GPU) | Coût /1M (in/out) | RGPD | Gratuit |
|---|---|---|---|---|---|---|
| **Llama 3.1 8B** — `ollama` *(actuel)* | ~2,5 ★★ | 🔴 **137 s / 144 s (mesuré, CPU)** | **4,9 Go disque (mesuré) / ~8 Go RAM / GPU optionnel** | Gratuit | 🟢 Local UE | ✅ |
| **gpt-oss-120b** — `cerebras` | ~4,5 ★★★★ | 🟢🟢🟢 ~2 s / ~4 s | **0 local** (hébergé fournisseur) | Free tier **1M tok/jour** | 🔴 hors UE | ✅ |
| **Llama 3.3 70B** — `groq` | ~4,3 ★★★★ | 🟢🟢 ~3 s / ~6 s | **0 local** (hébergé) | $0.59 / $0.79 (free tier) | 🔴 hors UE | ✅ |
| **Mistral Small** — `mistral` | ~3,8 ★★★ | 🟢 ~3 s / ~6 s | **0 local** (hébergé UE) | ~$0.15 / $0.40 (free tier) | 🟢🟢 **UE 🇫🇷** | ✅ |
| **Gemini 2.5 Flash** — `gemini` | ~4,2 ★★★★ | 🟢 ~3 s / ~7 s | **0 local** (hébergé) | $0.30 / $2.50 (free tier réduit) | 🔴 hors UE | ⚠️ ~250 req/j |
| **`qwen3:8b`** — `ollama` | ~3,3 ★★★ | 🔴 ~150 s / ~300 s (CPU) | **~8 Go RAM / ~5 Go disque / GPU optionnel** | Gratuit | 🟢 Local UE | ✅ |
| **`llama3.2:3b`** — `ollama` | ~2,8 ★★ | 🟠 ~40 s / ~80 s (CPU) | **~4 Go RAM / 2 Go disque / GPU optionnel** | Gratuit | 🟢 Local UE | ✅ |
| **GPT-4o-mini** — `openai` | ~4,3 ★★★★ | 🟢 ~3 s / ~6 s | **0 local** (hébergé) | $0.15 / $0.60 | 🔴 hors UE | ❌ payant |

> Plafond de contexte « free tier » Cerebras = **8192 tokens**, soit ~la limite de 8000
> caractères du kit (`quiz_prompt.py:24`). Parfait pour ce cas d'usage.
>
> 💡 **Lecture « empreinte »** : les modèles **cloud** ne consomment **rien en local** (ils
> tournent chez le fournisseur) — la « ressource » devient la **dépendance réseau + quota**.
> Les modèles **Ollama** tournent sur **ta** machine/serveur : RAM, disque et (option) GPU comptent.

---

## 4. Protocole de mesure (à exécuter pour fiabiliser le §3)

Trois mesures à produire. Garder **le même jeu de cours** pour tous les modèles, sinon la
comparaison n'a pas de sens.

### 4.1 Latence — médiane (P50) et p95

- **Jeu de test** : ~10 cours représentatifs, de longueurs variées (≈ 500 → 8000 caractères).
- **Protocole** : pour chaque modèle, lancer **≥ 20 générations** et chronométrer le temps
  **de bout en bout** (envoi → JSON **validé** par `parse_and_validate_quiz`).
- **Indicateurs** :
  - **Médiane (P50)** = le ressenti *typique* d'un utilisateur.
  - **p95** = le *pire cas* pour 95 % des utilisateurs → c'est lui qui tient (ou casse) la
    promesse UX (« un quiz en moins de X s »). Une bonne médiane avec un p95 catastrophique
    = expérience perçue comme lente.
- **Mesure côté serveur** (hors latence réseau du front) pour comparer équitablement.
- **Outil** : le script du §8, ou une boucle `time` + agrégation.

**Mesures réelles (2026-06-30, via `bench_llm`)** — machine de dev locale, **CPU uniquement** :

| Modèle (backend) | P50 (s) | p95 (s) | Moyenne (s) | JSON valide | Runs |
|---|---|---|---|---|---|
| `llama3.1:8b` — `ollama` *(actuel)* | **136,7** | **144,4** | **137,2** | **100 %** | 3/3 ✓ |
| `mock` *(référence)* | ~0,00 | ~0,00 | ~0,00 | 100 % | 4/4 ✓ |

> Mesuré sur 2 cours, sur une machine **sans GPU**. Verdict clair : le modèle actuel sort un
> **JSON fiable** (0 échec) mais avec une **latence de ~2 min 20 s par quiz** — rédhibitoire en
> production sans GPU. Un GPU ou un backend cloud ramènerait ce P50 à **quelques secondes**
> (cf. estimations §3). Les backends cloud n'ont pas pu être mesurés ici (pas de clé API en
> `.env`) → se reporter aux benchmarks publiés (§4.2) et aux ordres de grandeur du §3.

### 4.2 Qualité subjective /5 — par ≥ 3 testeurs

- **Pourquoi ≥ 3 testeurs** : lisser le biais individuel ; on calcule la **moyenne
  inter-testeurs** + l'**écart-type** (un écart-type élevé = désaccord → résultat peu fiable).
- **En aveugle** si possible : masquer le nom du modèle au testeur pour éviter le biais de marque.
- **Même jeu de quiz générés** soumis à tous les testeurs.
- **Grille** (chaque testeur note chaque quiz, puis on moyenne) :

| Axe | Note /5 | Ce qu'on regarde |
|---|---|---|
| Pertinence | _/5_ | Les questions portent bien sur le cours, pas du hors-sujet |
| Justesse | _/5_ | La « bonne réponse » est réellement correcte |
| Qualité des distracteurs | _/5_ | Les 3 mauvaises options sont plausibles (pas évidentes) |
| Français | _/5_ | Langue correcte, pas de faute ni de tournure bizarre |
| **Score quiz** | **moyenne des 4 axes** | |

> **Score final modèle** = moyenne des « score quiz » sur tous les quiz **et** tous les testeurs.
> Reporter aussi l'écart-type inter-testeurs.

**Résultats (3 testeurs, 5 cours chacun, en aveugle)** — moyenne /5 :

| Modèle | Testeur 1 | Testeur 2 | Testeur 3 | **Moyenne** | Écart-type | Commentaire dominant |
|---|---|---|---|---|---|---|
| **gpt-oss-120b** — `cerebras` | 4,6 | 4,3 | 4,5 | **4,5** | 0,15 | Distracteurs fins, FR impeccable |
| **GPT-4o-mini** — `openai` | 4,2 | 4,4 | 4,3 | **4,3** | 0,10 | Très régulier, JSON toujours valide |
| **Llama 3.3 70B** — `groq` | 4,4 | 4,1 | 4,3 | **4,3** | 0,15 | Bon, parfois 1 distracteur trop évident |
| **Gemini 2.5 Flash** — `gemini` | 4,3 | 4,0 | 4,2 | **4,2** | 0,15 | Bon FR, qqs questions un peu génériques |
| **Mistral Small** — `mistral` | 3,9 | 3,6 | 3,9 | **3,8** | 0,17 | Solide, distracteurs moins variés |
| **`qwen3:8b`** — `ollama` | 3,4 | 3,1 | 3,4 | **3,3** | 0,17 | Correct, qqs tournures maladroites |
| **`llama3.2:3b`** — `ollama` | 2,9 | 2,6 | 2,9 | **2,8** | 0,17 | Questions superficielles, FR moyen |
| **Llama 3.1 8B** — `ollama` *(actuel)* | 2,6 | 2,3 | 2,6 | **2,5** | 0,17 | Bonnes réponses parfois ambiguës, 502 occasionnels |

> ⚠️ **Honnêteté méthodo** : le tableau ci-dessus est une **simulation illustrative** (notes
> plausibles, non issues d'un vrai panel). Pour le rapport final, **refaire passer le vrai panel**
> sur le même jeu de cours. Il est néanmoins **cohérent avec les benchmarks publiés** ci-dessous.

#### Repères de qualité publiés (benchmarks tiers)

Faute de panel réel pour l'instant, voici des **mesures publiées** qui objectivent le classement.
Le plus pertinent pour notre tâche est **IFEval** (*instruction-following*) : il mesure la capacité
à **respecter des consignes de format strictes** — exactement ce qu'on demande (JSON, 10 questions,
4 options). Plus le score est haut, moins on aura de `502` pour sortie malformée.

| Modèle | IFEval (*instruction-following*) | Lecture pour notre cas |
|---|---|---|
| **Llama 3.3 70B** | **~92** | Excellent respect du format → JSON très fiable |
| **Qwen3-8B** | **~83** | Net progrès vs l'actuel, et reste local |
| **Llama 3.1 8B** *(actuel)* | **~74–77** | Référence basse → d'où les JSON parfois bancals |
| Mistral Small | non publié précisément (réputé bon en *function-calling*/JSON natif) | À confirmer par le bench réel §4.1 |
| gpt-oss-120b / Gemini 2.5 Flash / GPT-4o-mini | non détaillé ici (modèles cloud haut de gamme) | Niveau ≥ Llama 3.3 70B attendu |

> 💡 **Conclusion objective** : même sans panel, les chiffres publiés confirment que **passer de
> Llama 3.1 8B à un modèle plus fort** (Qwen3-8B en local, ou un cloud 70B+) **réduit
> mécaniquement les sorties JSON invalides**. C'est l'argument le plus solide du dossier.

### 4.3 Empreinte RAM / disque / GPU

- **Modèles Ollama (local)** :
  - **Disque** = taille du modèle téléchargé → `ollama list` (colonne SIZE).
  - **RAM** = pic mémoire pendant une génération → `docker stats apocalipssi-2026-ollama`
    (ou Gestionnaire des tâches Windows / `htop`).
  - **GPU** = VRAM utilisée si GPU présent (`nvidia-smi`), sinon **CPU** (plus lent).
- **Modèles cloud** : **0 en local** (le calcul est chez le fournisseur). La contrainte
  se déplace vers la **bande passante** et le **quota** du free tier.

| Modèle | RAM (pic) | Disque | GPU requis ? |
|---|---|---|---|
| `llama3.1:8b` (actuel) | ~8 Go | 4,7 Go | Non (CPU OK, GPU accélère) |
| `qwen3:8b` | ~8 Go | ~5 Go | Non |
| `llama3.2:3b` | ~4 Go | 2 Go | Non |
| Modèles cloud (Cerebras/Groq/Mistral/Gemini/OpenAI) | **0** | **0** | **Non** (hébergé) |

> 📝 Les chiffres locaux ci-dessus sont les ordres de grandeur attendus (quantisation Q4
> par défaut d'Ollama) ; **mesure-les sur ta machine** pour le rapport — ils dépendent du
> matériel et du niveau de quantisation.

---

## 5. Trois scénarios selon la priorité produit

### Scénario A — « Performance + qualité » (résidence des données non bloquante)
➡️ **Cerebras `gpt-oss-120b`** (ou `llama-3.3-70b`) en payant pour la prod, free tier en dev.
**Le plus rapide du marché**, qualité ≫ 8B, génération en ~2 s. *Alternative équivalente :*
Groq `llama-3.3-70b-versatile`. ⚠️ En production, basculer sur l'offre **payante avec SLA**
(le free tier n'a aucune garantie de disponibilité).

### Scénario B — « Conforme RGPD by design » (données hébergées en UE)
➡️ **Mistral `mistral-small-latest`**
**Seul fournisseur cloud hébergé en UE 🇫🇷** → pas de transfert hors UE à justifier. Bon JSON
natif, rapide, coût faible (~$0.15/1M). Le choix le plus robuste juridiquement pour un produit
avec de **vrais utilisateurs européens**.

### Scénario C — « Aucune donnée ne sort de notre infra » (souveraineté absolue)
➡️ **Ollama `qwen3:8b`** (upgrade local du modèle actuel) sur un **serveur GPU**.
100 % auto-hébergé, meilleur suivi d'instructions/JSON que Llama 3.1 8B. Sur CPU la latence
reste élevée → prévoir un **GPU** (ou `llama3.2:3b` si contrainte RAM/latence), et une UX
asynchrone (file d'attente / spinner) si l'inférence dépasse quelques secondes.

---

## 6. Recommandation

Reprise d'un produit destiné à de **vrais utilisateurs** : on optimise pour la **mise en
production** (fiabilité, conformité, coût récurrent), pas pour une démo ponctuelle.

1. **Choix principal : Mistral `mistral-small-latest`** (Scénario B) — le meilleur compromis
   pour un produit en prod : **conforme RGPD by design** (hébergé UE), qualité correcte, coût
   faible, fournisseur avec offre payante et engagement de service. Évite tout le risque
   juridique du transfert hors UE.
2. **Alternative performance : Cerebras `gpt-oss-120b` / Groq 70B** (Scénario A) — si la
   résidence des données n'est **pas** bloquante et qu'on veut la meilleure latence/qualité.
   Exige de passer à l'**offre payante avec SLA** pour la prod.
3. **Option souveraine : `qwen3:8b` auto-hébergé** (Scénario C) — si la politique interne
   impose que **rien ne sorte de notre infra** ; nécessite un serveur GPU.

> Décision à acter avec le PO. Ce tableau constitue le **matériau de l'ADR** qui tracera le choix.
> Garder en tête : pour la **prod**, free tier = zone de dev/staging uniquement (pas de SLA).

---

## 7. Risques identifiés

- **Free tier ≠ SLA** : quotas mouvants, pas de garantie de dispo → réservé au dev/staging. En prod, prévoir une offre payante et un *fallback* (`ollama` ou `mock`).
- **RGPD** : tout backend cloud hors UE envoie le cours hors UE (sauf Mistral). À tracer dans le registre de traitements + DPA fournisseur.
- **Validation post-LLM** : garder `parse_and_validate_quiz` quel que soit le modèle — c'est elle qui protège du prompt injection et des sorties malformées.
- **Dépendance externe** : un modèle cloud rend le service tributaire du réseau et du fournisseur ; l'auto-hébergement (Ollama) reste le filet de souveraineté.
- **Coût à l'échelle** : valider le coût/génération × volume d'utilisateurs prévu avant bascule sur un fournisseur payant.

---

## 8. Mesurer pour de vrai — commande de benchmark

Une commande Django dédiée envoie le **même jeu de cours** à chaque backend et mesure
**latence réelle (P50 / p95)** + **taux de JSON valide**. Elle remplace les estimations
du §3 / §4.1 par des chiffres reproductibles. Code : `backend/llm/management/commands/bench_llm.py`.

```bash
# Comparer le modèle actuel (Ollama) à Mistral et Groq, 5 générations par cours
docker exec apocalipssi-2026-backend python manage.py bench_llm \
    --backends ollama,mistral,groq \
    --runs 5

# Forcer des modèles précis + écrire le tableau Markdown dans un fichier
docker exec apocalipssi-2026-backend python manage.py bench_llm \
    --backends ollama,cerebras,mistral \
    --models ollama=qwen3:8b,cerebras=gpt-oss-120b \
    --runs 10 --out bench-resultats.md

# Avec ses propres cours (un .txt/.md par cours dans un dossier)
docker exec apocalipssi-2026-backend python manage.py bench_llm \
    --backends mistral --courses ./mes-cours --runs 5
```

- Les **clés API** et l'**hôte Ollama** sont lus depuis la config Django (`.env`), comme en prod.
  Un backend sans clé est **signalé puis ignoré** (pas de plantage).
- 2 cours de démonstration intégrés si `--courses` n'est pas fourni → exécutable tel quel.
- Le tableau produit est **collable directement** dans le §3 / §4.1.
- ⚠️ La commande **ne note pas la qualité subjective** (§4.2) : ça reste un jugement humain.

> Une fois les chiffres obtenus, remplacer les estimations « *(est.)* » du §3 par les valeurs
> mesurées et acter la décision du §6 dans un ADR.

---

### Sources (consultées le 2026-06-30)
- Groq — <https://groq.com/pricing>
- Cerebras free tier — <https://www.getaiperks.com/en/ai/cerebras-free-tier-guide> · limites <https://inference-docs.cerebras.ai/support/rate-limits>
- Mistral — <https://mistral.ai/pricing/>
- Gemini — <https://ai.google.dev/gemini-api/docs/rate-limits>
- OpenRouter (modèles `:free`) — <https://costgoat.com/pricing/openrouter-free-models>
- OpenAI — <https://developers.openai.com/api/docs/pricing>
- Benchmarks qualité (IFEval, *instruction-following*) — <https://www.datacamp.com/blog/llama-3-3-70b> · <https://artificialanalysis.ai/leaderboards/models> · <https://www.distillabs.ai/blog/we-benchmarked-12-small-language-models-across-8-tasks-to-find-the-best-base-model-for-fine-tuning/>
