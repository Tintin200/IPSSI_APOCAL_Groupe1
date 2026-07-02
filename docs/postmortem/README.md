# Post-mortem & qualité — Perturbation J4 (« Le retour de Mme Lefèvre »)

> **Contexte.** Jeudi, retour de crise de **Mme Lefèvre** (persona enseignante introduite en J1) :
> elle signale des **erreurs factuelles** dans des questions générées. La boucle utilisateur
> promise en J1 est-elle réellement intégrée ? L'attendu J4 combine un volet **managérial**
> (artefacts + risques + pilotage, intégrés directement dans les artefacts de cadrage —
> [`../cadrage/`](../cadrage/) : persona Lucia, story map, backlog + annexe risques)
> et un volet **qualité / crise** documenté ici.

## Contenu du dossier

- [`post-mortem-blameless.md`](./post-mortem-blameless.md) — analyse *blameless* de l'incident (chronologie, causes, actions).
- [`audit-qualite-50-questions.md`](./audit-qualite-50-questions.md) — audit qualité sur 50 questions générées.
- [`email-client.md`](./email-client.md) — réponse professionnelle à Mme Lefèvre.
- [`modele-question-report.md`](./modele-question-report.md) — **spécification** du modèle `QuestionReport` (boucle de feedback), à implémenter en bonus.

## Note de cadrage (grille d'évaluation)

D'après `contexte/grille-evaluation.xlsx`, la **Perturbation 5** notée porte sur
l'**accessibilité / Erasmus / scaling européen** (MAJ des artefacts) — déjà traitée dans
les artefacts de cadrage (`docs/cadrage/`). Les livrables de crise ci-dessus (post-mortem, audit,
`QuestionReport`) proviennent du **scénario détaillé de `consigne.md`** ; ils **renforcent** le
dossier (preuve de boucle utilisateur et de démarche qualité) mais ne sont pas notés ligne à ligne.
Ils sont produits comme **documents** ; l'implémentation du modèle reste un **bonus technique**.
