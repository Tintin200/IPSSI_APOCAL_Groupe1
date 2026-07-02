# Registre des activités de traitement — EduTutor IA

**Base légale :** RGPD **Art. 30** · **Version :** 1.0 · **Date :** 2026-07-01

> Le registre recense les traitements de données personnelles réalisés par EduTutor IA. Il est tenu
> à jour par le responsable du traitement et présentable à la CNIL sur demande.

## Responsable du traitement

- **Entité :** EduTutor IA — projet APOCAL'IPSSI 2026, Groupe 1 (à remplacer par la société EduTutor
  et son SIREN lors de la mise en production).
- **Contact référent données :** `dpo@edututor.example` *(à définir)*.

## Traitements recensés

### T1 — Gestion des comptes utilisateurs

| Champ | Valeur |
|---|---|
| Finalité | Inscription, authentification, gestion du profil |
| Base légale | Exécution du contrat (Art. 6-1-b) |
| Personnes concernées | Utilisateurs inscrits (étudiants, enseignants) |
| Données | Email, nom, prénom, mot de passe (haché), statut de vérification email |
| Destinataires | Équipe EduTutor ; hébergeur (sous-traitant) ; Brevo (emails transactionnels) |
| Transfert hors UE | Non |
| Durée de conservation | Durée de vie du compte + 30 j (cf. politique de rétention) |
| Mesures de sécurité | Mots de passe hachés, token d'authentification, HTTPS, isolation par utilisateur |

### T2 — Génération de quiz à partir d'un cours

| Champ | Valeur |
|---|---|
| Finalité | Produire 10 QCM à partir d'un cours fourni par l'utilisateur (F3) |
| Base légale | Exécution du contrat (Art. 6-1-b) |
| Personnes concernées | Utilisateurs inscrits |
| Données | Texte du cours (`source_text`, peut contenir des données personnelles), quizz, scores, historique |
| Destinataires | Équipe EduTutor ; **fournisseur LLM** (sous-traitant, cf. ADR-J2) |
| Transfert hors UE | **Non** avec le choix principal **Mistral** (hébergé UE). *Oui* si bascule Cerebras/Groq → à encadrer par CCT + information |
| Durée de conservation | Durée de vie du compte (suppression en cascade) |
| Mesures de sécurité | Isolation par utilisateur, validation stricte des sorties LLM, prompt structuré (défense injection J3) |

### T3 — Emails transactionnels (validation, réinitialisation)

| Champ | Valeur |
|---|---|
| Finalité | Vérifier l'email, permettre la réinitialisation du mot de passe |
| Base légale | Exécution du contrat / intérêt légitime (sécurité) |
| Données | Email, jeton temporaire |
| Sous-traitant | **Brevo** (envoi d'emails) — DPA à jour requis |
| Transfert hors UE | Non (Brevo, UE) |
| Durée de conservation | Le temps de l'envoi ; pas d'archivage côté produit |

### T4 — Demandes d'exercice des droits (SAR)

| Champ | Valeur |
|---|---|
| Finalité | Traiter les demandes d'accès / portabilité / effacement (Art. 15, 20, 17) |
| Base légale | Obligation légale (Art. 6-1-c) |
| Données | Email du demandeur, type de demande, dates, statut |
| Durée de conservation | 36 mois (preuve de conformité) |

## Sous-traitants (récapitulatif)

| Sous-traitant | Rôle | Localisation | Encadrement |
|---|---|---|---|
| Hébergeur (VPS/cloud) | Hébergement application + base | UE | Contrat + DPA |
| Brevo | Emails transactionnels | UE | DPA |
| **Mistral** (LLM principal) | Génération des QCM | **UE 🇫🇷** | DPA — pas de transfert hors UE |
| Cerebras / Groq (LLM alternatif) | Génération des QCM (option perf) | **Hors UE** | CCT + information utilisateurs **si activé** |

> **Cohérence avec l'ADR-J2 :** le registre reflète le choix LLM principal **Mistral (UE)**. Toute
> bascule vers un fournisseur hors UE doit **mettre à jour ce registre** (ligne transfert hors UE) et
> la politique de confidentialité avant activation en production.
