# ADR-004 — Architecture de passage à l'échelle et de résilience

**Statut :** Proposé — à valider avec le PO / la DG
**Date :** 2026-07-02
**Contexte :** Perturbation J4 — succès national, plateforme d'État (RGAA, i18n, montée en charge)
**Référence :** artefacts J4 (`docs/cadrage/` : story map, backlog EP-10/11/12/13, release planning)

## 1. Contexte

Après un pic de trafic (passage télé) et l'intérêt de l'État pour faire d'EduTutor la plateforme de référence des lycées, trois exigences non négociables émergent : **scalabilité/résilience**, **accessibilité RGAA** et **internationalisation**. Les serveurs ont failli tomber ; l'architecture actuelle (mono-instance) ne tient pas la charge nationale. La génération LLM est le point chaud (latence, dépendance fournisseur).

## 2. Options envisagées

| Axe | Options | Décision |
|---|---|---|
| Montée en charge | Scale vertical (grosse machine) **vs** horizontal (instances sans état + autoscaling) | **Horizontal** : services stateless + autoscaling |
| Génération LLM | Appel synchrone bloquant **vs** file d'attente asynchrone | **File asynchrone** (worker) + UX spinner/polling |
| Base de données | Instance unique **vs** managée + réplicas lecture | **Managée + réplicas lecture** + cache Redis |
| Dépendance LLM | Fournisseur unique **vs** fournisseur de secours | **Fallback** (Ollama/mock) + file d'attente |
| Statique | Servi par l'app **vs** CDN | **CDN** pour le statique |

## 3. Décision retenue

Migrer vers une architecture **horizontalement scalable et résiliente** : services **sans état** derrière un autoscaling, **cache Redis**, **file d'attente asynchrone** pour la génération LLM (découplage du web), **base managée + réplicas de lecture**, **CDN** pour le statique, **observabilité** (métriques/logs/alertes) et **fournisseur LLM de secours**. Chaque brique majeure de la migration fera l'objet d'un ADR daté dédié lors de sa mise en œuvre.

## 4. Justification

10× d'utilisateurs = 10× la charge et l'exposition. Le découplage de la génération LLM (async) évite qu'un pic bloque le web. Le stateless + autoscaling absorbe les pics ; le cache et les réplicas soulagent la base ; le fournisseur de secours traite le risque de dépendance externe (identifié R-04 dans l'analyse de risques J4). L'essentiel étant managérial (artefacts + risques + pilotage), la mise en œuvre technique est incrémentale.

## 5. Conséquences

### Positives
- Tenue de charge à l'échelle nationale ; résilience accrue (fallback, réplicas).
- Génération LLM non bloquante pour le reste de l'application.

### Négatives
- Complexité opérationnelle et coût cloud accrus (à budgéter, risque R-03 coût).
- Introduit des composants (Redis, worker, CDN) à exploiter et superviser.

### À surveiller
- Tests de charge + budget d'alerte cloud (risques R-01 saturation, R-03 coût — cf. annexe backlog).
- Accessibilité RGAA et i18n traitées comme exigences liées mais distinctes (epics EP-11/EP-12).
- Documenter chaque migration (réplicas, cache, file) par un ADR daté dédié.
