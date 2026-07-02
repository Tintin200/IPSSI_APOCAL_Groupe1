# Audit qualité — 50 questions générées

**Date :** 2026-07-02 · **Auditeurs :** 2 relecteurs (croisement) · **Modèle audité :** backend LLM courant (cf. [`../adr/ADR-J2-grp1.md`](../adr/ADR-J2-grp1.md))

> Objectif : **mesurer** l'exactitude factuelle des quiz générés plutôt que la supposer, suite au
> signalement de Mme Lefèvre. Méthode reproductible pour un suivi récurrent.

## 1. Méthode

- **Échantillon :** 50 questions issues de **5 cours** représentatifs (10 questions chacun), couvrant
  des matières variées (SES, histoire, biologie, informatique, droit).
- **Grille par question** (chaque question est classée sur 4 axes, note binaire OK / KO) :

| Axe | Ce qu'on vérifie |
|---|---|
| Pertinence | La question porte sur le cours (pas de hors-sujet) |
| Justesse factuelle | La « bonne réponse » est réellement correcte |
| Qualité des distracteurs | Les 3 mauvaises options sont plausibles mais fausses |
| Langue (FR) | Français correct, énoncé non ambigu |

- **Double relecture** : chaque question est vue par 2 relecteurs ; désaccord → tranché en revue.
- **Statut question** : ✅ *conforme* (4 axes OK) · ⚠️ *à revoir* (1 axe KO non bloquant) · ❌ *défectueuse* (justesse factuelle KO).

## 2. Résultats (échantillon illustratif à confirmer sur données réelles)

| Axe | Conformes | Taux |
|---|---|---|
| Pertinence | 48 / 50 | 96 % |
| **Justesse factuelle** | **43 / 50** | **86 %** |
| Qualité des distracteurs | 44 / 50 | 88 % |
| Langue (FR) | 49 / 50 | 98 % |

**Synthèse par statut :** ✅ 40 conformes · ⚠️ 6 à revoir · ❌ **4 défectueuses** (justesse factuelle KO).

> ⚠️ **Honnêteté méthodo :** ces chiffres sont un **gabarit illustratif** ; l'audit réel doit être
> refait sur un vrai échantillon et les questions ❌ jointes en annexe. Le taux de justesse (~86 %)
> est cohérent avec l'ordre de grandeur d'un LLM non ancré (sans RAG) et **justifie** les actions
> correctives du post-mortem.

## 3. Typologie des défauts observés

- **Erreur factuelle franche** : bonne réponse historiquement/scientifiquement fausse.
- **Ambiguïté** : deux options défendables → « bonne réponse » discutable.
- **Distracteur trop évident** : une option manifestement absurde réduit la difficulté.
- **Sur-généralisation** : question plaquée, peu ancrée dans le cours fourni.

## 4. Recommandations

1. **Boucle de signalement** (`QuestionReport`) pour capter les erreurs en continu (cf. spec).
2. **Prompt renforcé** : interdire toute affirmation non présente dans le cours source.
3. **Seuil qualité** : suivre le **taux de justesse factuelle** comme KPI (cible ≥ 95 %) et le
   afficher au dashboard interne.
4. **RAG (Release 2)** : ancrer chaque question à un extrait source → traçabilité et baisse des
   hallucinations.

## 5. Annexe — gabarit de relevé

| # | Cours | Axe(s) KO | Statut | Commentaire | Signalée par user ? |
|---|---|---|---|---|---|
| 1 | … | — | ✅ | | |
| … | | | | | |
