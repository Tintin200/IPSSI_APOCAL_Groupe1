# Analyse de risques

## Méthode

Chaque risque est évalué avec une matrice probabilité × d'impact.

Échelle :

- Probabilité : 1 faible, 2 moyenne, 3 forte
- Impact : 1 faible, 2 moyen, 3 fort
- Criticité = Probabilité × Impact

## Matrice des risques

| ID | Risque | Probabilité | Impact | Criticité | Action préventive | Backlog lié |
|---|---|---:|---:|---:|---|---|
| R-01 | Saturation des serveurs à cause du grand nombre d'élèves connectés | 3 | 3 | 9 | Réaliser des tests de charge et prévoir du cache | PB-08, PB-09 |
| R-02 | Non-conformité RGAA empêchant l'adoption par l'État | 3 | 3 | 9 | Réaliser un audit RGAA et corriger les problèmes prioritaires | PB-03, PB-04, PB-05 |
| R-03 | Mauvaise traduction de l'interface ou des réponses IA | 2 | 3 | 6 | Externaliser les textes et tester les réponses multilingues | PB-06, PB-07 |
| R-04 | Dépendance trop forte à un seul fournisseur LLM | 2 | 3 | 6 | Prévoir un fournisseur IA de secours | PB-10 |
| R-05 | Retard du projet à cause des nouvelles exigences | 3 | 2 | 6 | Reprioriser le backlog et reporter les éléments non essentiels | PB-03 à PB-10 |
| R-06 | Dégradation de l'expérience utilisateur pendant la montée en charge | 2 | 2 | 4 | Surveiller les performances et prioriser les pages critiques | PB-08, PB-09 |

## Risques prioritaires

Les risques les plus critiques sont :

- R-01 : saturation des serveurs ;
- R-02 : non-conformité RGAA ;
- R-03 : mauvaise gestion multilingue ;
- R-05 : retard du projet.

Ces risques sont directement liés à des actions préventives intégrées au backlog.