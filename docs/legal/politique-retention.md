# Politique de rétention des données — EduTutor IA

**Version :** 1.0 · **Date :** 2026-07-01 · **Responsable :** Équipe EduTutor IA (Groupe 1, APOCAL'IPSSI 2026)

> Cette politique définit **combien de temps** chaque donnée personnelle est conservée, **comment**
> elle est supprimée ou anonymisée, et **qui** en est responsable. Elle applique le principe RGPD de
> **limitation de la conservation** (Art. 5-1-e) : on ne garde une donnée que le temps nécessaire à
> la finalité pour laquelle elle a été collectée.

## 1. Durées de conservation par catégorie de données

| Donnée | Finalité | Durée de conservation | Point de départ |
|---|---|---|---|
| **Compte** (email, nom, prénom, mot de passe haché) | Authentification, gestion du compte | **Durée de vie du compte**, puis suppression sous **30 jours** après demande de suppression | Dernière connexion |
| **Comptes inactifs** | — | Suppression / anonymisation après **24 mois** d'inactivité, après email de relance | Dernière connexion |
| **Cours déposés** (`source_text`, PDF extrait) | Génération des QCM | **Durée de vie du compte**. L'utilisateur peut supprimer un quiz (et son cours) à tout moment | Création du quiz |
| **Quizz & historique de scores** | Suivi de progression | **Durée de vie du compte** | Création du quiz |
| **Token d'authentification** (DRF + `localStorage`) | Session | Invalidé à la déconnexion, au changement de mot de passe et à la suppression du compte | Émission |
| **Logs applicatifs** (techniques, sans contenu de cours) | Sécurité, débogage | **12 mois** glissants, puis purge automatique | Écriture du log |
| **Journal des demandes RGPD** (audit trail SAR) | Preuve de conformité | **36 mois** | Réception de la demande |
| **Emails transactionnels** (validation, reset) | Sécurité du compte | Non archivés côté produit (délégués au prestataire Brevo, cf. registre) | Envoi |

## 2. Suppression, anonymisation et archivage

- **Effacement à la demande (Art. 17)** : la suppression de compte est **définitive et en cascade** —
  le profil, tous les quizz, tous les cours (`source_text`) et toutes les questions associées sont
  supprimés. Aucune copie « soft-delete » n'est conservée en base de production.
- **Sauvegardes** : les sauvegardes chiffrées de la base sont **tournantes sur 30 jours**. Une donnée
  supprimée disparaît donc au plus tard **30 jours** après l'effacement, quand la dernière sauvegarde
  la contenant est écrasée. Ce délai est communiqué au demandeur.
- **Anonymisation** : pour les comptes inactifs supprimés en masse, les données de progression peuvent
  être **anonymisées** (dissociées de toute personne identifiable) plutôt que détruites, si elles
  servent des statistiques agrégées — jamais réversibles.
- **Purge automatique** : les logs et le journal SAR sont purgés par tâche planifiée selon les durées
  du §1.

## 3. Responsabilités et révision

- **Responsable du traitement** : l'équipe EduTutor IA (à transférer à l'entité juridique EduTutor
  lors de la mise en production).
- **Application** : la politique est mise en œuvre par des tâches de purge planifiées + la logique
  d'effacement en cascade côté application.
- **Sous-traitants** : les durées imposées aux prestataires (hébergeur, Brevo pour l'email, fournisseur
  LLM) sont alignées sur ce document via les contrats/DPA (cf. [`registre-traitements.md`](./registre-traitements.md)).
- **Révision** : cette politique est revue **au moins une fois par an** et à chaque évolution majeure
  du produit (nouveau traitement, nouveau sous-traitant, changement de fournisseur LLM).

> **Lien LLM ↔ rétention :** le fournisseur de génération retenu est **Mistral** (hébergé UE, cf.
> [`../adr/ADR-J2-grp1.md`](../adr/ADR-J2-grp1.md)). Le `source_text` transmis pour générer un quiz
> n'est **pas conservé** par le fournisseur au-delà du traitement de la requête (à confirmer au DPA).
> En cas de bascule sur un fournisseur hors UE (Cerebras/Groq), ce transfert devra être tracé ici et
> au registre.
