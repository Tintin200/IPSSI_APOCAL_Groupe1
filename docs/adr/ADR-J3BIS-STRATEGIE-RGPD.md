# ADR-003 — Stratégie de conformité RGPD (privacy by design)

**Statut :** Accepté par l'équipe — à valider avec le PO
**Date :** 2026-07-01
**Contexte :** Perturbation J3-bis — « La demande inattendue » (SAR RGPD Art. 15)
**Référence :** `docs/legal/` (politique de rétention, registre, pages légales)

## 1. Contexte

Une demande formelle d'accès aux données personnelles (SAR, **RGPD Art. 15**) arrive en pleine livraison MVP. Le produit vise de vrais utilisateurs européens et le cours déposé (`source_text`) peut contenir des données personnelles. Il faut une stratégie de conformité tenable, sans mettre en risque la livraison de mercredi soir.

## 2. Options envisagées

| Sujet | Options | Décision |
|---|---|---|
| Export des données | Dump manuel en base **vs** endpoint automatisé filtré par utilisateur | **Endpoint automatisé**, filtré par `request.user` (anti-fuite inter-comptes) |
| Format | PDF **vs** JSON/CSV machine-readable | **JSON** (Art. 20, réutilisable) |
| Effacement | Soft-delete **vs** hard-delete | **Hard-delete en cascade** (Art. 17 : profil + quizz + questions) |
| Résidence des données | Fournisseur LLM UE **vs** hors UE | **UE (Mistral)** par défaut (cf. ADR-001) |
| Priorité MVP | Tout coder maintenant **vs** spécifier + différer | **Spécifier** l'endpoint, traiter la 1ʳᵉ demande manuellement sous 48 h |

## 3. Décision retenue

Adopter une approche **privacy by design** : (1) endpoint d'export `GET /api/accounts/export/` filtré strictement par utilisateur, JSON machine-readable ; (2) effacement en cascade à la suppression de compte ; (3) **politique de rétention** écrite et **registre des traitements** (Art. 30) ; (4) **4 pages légales** rédigées ; (5) fournisseur de génération **hébergé UE** pour éviter tout transfert hors UE. L'implémentation du code d'export est différée (Should) ; la conformité documentaire et le traitement manuel sous 48 h sont livrés immédiatement.

## 4. Justification

Le privacy by design coûte moins cher que de rattraper la conformité sous pression CNIL (cf. amende Klarna 1,2 M€). L'isolation par utilisateur est la protection clé contre les fuites inter-comptes. Le format JSON et l'effacement cascade répondent directement aux Art. 15/17/20. Différer le *code* de l'export (mais pas la *décision*) protège la livraison MVP sans sacrifier la conformité.

## 5. Conséquences

### Positives
- Conformité Art. 15/17/20 traçable ; pas de transfert hors UE avec le choix par défaut.
- Isolation par utilisateur = pas de fuite inter-comptes.

### Négatives
- Traitement manuel des SAR tant que l'endpoint n'est pas codé (acceptable à faible volume).
- Purge des sauvegardes sous 30 j → délai communiqué au demandeur.

### À surveiller
- Journaliser les demandes SAR (audit trail : qui/quand/type/statut).
- Si bascule sur un fournisseur hors UE (cf. ADR-001), **mettre à jour le registre** + CCT.
