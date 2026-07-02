# Post-mortem *blameless* — Erreurs factuelles signalées dans les quiz générés

**Date :** 2026-07-02 · **Rédacteurs :** Équipe EduTutor IA (Groupe 1) · **Sévérité :** Moyenne (qualité produit, pas de fuite de données)

> **Principe *blameless*.** On analyse le **système et le processus**, pas les personnes. L'objectif
> n'est pas de désigner un coupable mais de comprendre *pourquoi* l'erreur a pu se produire et
> *comment* l'empêcher de se reproduire. Une erreur qui atteint l'utilisateur est une défaillance de
> nos garde-fous, pas d'un individu.

## 1. Résumé

Mme Lefèvre, enseignante utilisatrice, a signalé que **plusieurs questions générées contenaient des
erreurs factuelles** (bonne réponse incorrecte, ou énoncé ambigu). Le service produit bien 10 QCM
valides *structurellement* (4 options, 1 bonne réponse), mais **rien ne garantit l'exactitude
factuelle** du contenu, et **aucun canal simple ne permettait à l'utilisateur de signaler** une
question erronée. La promesse J1 d'une « boucle de feedback » n'était donc pas réellement outillée.

## 2. Chronologie (timeline)

| Moment | Événement |
|---|---|
| J1 | Persona Mme Lefèvre ajoutée ; besoin d'une boucle de feedback identifié mais non priorisé. |
| J2 | Bascule du modèle LLM (latence) — focus performance, pas exactitude. |
| J3 | Durcissement sécurité (prompt injection) : la **validation valide la *structure*, pas la *véracité***. |
| Mer. soir | Livraison MVP : quiz fonctionnels, exactitude non auditée. |
| Jeu. matin | Mme Lefèvre signale des erreurs factuelles sur ses cours. |
| Jeu. | Ouverture du présent post-mortem + audit qualité 50 questions. |

## 3. Impact

- **Utilisateur :** perte de confiance d'une utilisatrice clé (enseignante = cible stratégique J1).
- **Produit :** risque de réputation (« l'IA se trompe »), aligné avec le différenciateur raté
  « pédagogie ancrée, pas hallucinée ».
- **Aucun** impact sécurité / données personnelles.

## 4. Causes (analyse *5 Whys*)

1. *Pourquoi des erreurs factuelles atteignent l'utilisateur ?* → Aucune vérification d'exactitude
   après génération.
2. *Pourquoi ?* → La validation post-LLM (J3) ne contrôle que la **structure** (schéma JSON, 4 options).
3. *Pourquoi ?* → L'exactitude était supposée « suffisante » avec un bon modèle, sans mesure.
4. *Pourquoi ?* → Pas d'**audit qualité** ni de **boucle de signalement** dans le périmètre MVP.
5. *Pourquoi ?* → La boucle de feedback J1 a été identifiée mais dépriorisée face aux perturbations
   techniques (J2/J3).

**Cause racine :** absence d'un dispositif *produit* de mesure et de remontée de la qualité factuelle
(la génération était traitée comme « terminée » dès que la structure était valide).

## 5. Ce qui a bien fonctionné

- La validation structurelle (J3) a évité les quiz malformés — l'incident est resté circonscrit à
  l'exactitude.
- Le signalement de l'utilisatrice a été pris au sérieux et tracé rapidement.
- L'ancrage « enseignant-first » (J1) a fait remonter le problème par la bonne personne.

## 6. Actions correctives (→ backlog)

| Action | Type | Priorité | Estimation |
|---|---|---|---|
| Modèle **`QuestionReport`** + bouton « signaler cette question » | Correctif produit | **Must** | 5 pts |
| **Audit qualité** récurrent sur un échantillon de questions (cf. audit 50Q) | Process | **Should** | 3 pts |
| Consigne de prompt renforcée : « n'affirme que ce qui est explicitement dans le cours » | Correctif LLM | **Should** | 3 pts |
| Piste **RAG** (chaque réponse traçable à une source du cours) | Évolution R2 | **Could** | 13 pts |
| Boucle de feedback affichée dans le parcours (J1 tenue) | Correctif UX | **Must** | 3 pts |

## 7. Enseignements

- **« Structurellement valide » ≠ « factuellement correct ».** Nos garde-fous couvraient la forme,
  pas le fond.
- Une **boucle de feedback** n'est réelle que si elle est **outillée** (un canal de signalement),
  pas seulement mentionnée dans un persona.
- La qualité se **mesure** (audit échantillon) avant d'être affirmée.
