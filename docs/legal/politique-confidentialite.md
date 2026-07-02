# Politique de confidentialité

*Dernière mise à jour : 2026-07-01*

Cette politique explique comment EduTutor IA collecte, utilise et protège les données personnelles
de ses utilisateurs, conformément au RGPD.

## 1. Responsable du traitement

Le responsable du traitement est l'équipe éditrice d'EduTutor IA (cf. mentions légales). Contact du
référent données : `dpo@edututor.example`.

## 2. Données personnelles collectées

- **Compte** : email, nom, prénom, mot de passe (stocké **haché**, jamais en clair).
- **Contenus** : cours déposés (texte collé ou extrait de PDF), qui peuvent contenir des données
  personnelles selon ce que l'utilisateur y insère.
- **Usage** : quiz générés, réponses données, scores et historique de progression.
- **Technique** : jeton d'authentification (stocké dans le `localStorage` du navigateur), logs
  techniques (sans le contenu des cours).

## 3. Finalités du traitement

- Créer et gérer votre compte, vous authentifier.
- Générer des quiz à partir de vos cours (fonction principale du service).
- Suivre votre progression (scores, révision des erreurs).
- Assurer la sécurité du service et vous envoyer les emails nécessaires (validation, réinitialisation).

## 4. Base légale

Le traitement repose sur l'**exécution du contrat** (fourniture du service que vous demandez, RGPD
Art. 6-1-b) et, pour la sécurité, sur l'**intérêt légitime** de l'éditeur. Les emails de sécurité
relèvent d'une **obligation** de protection du compte.

## 5. Durée de conservation

Vos données sont conservées **pendant la durée de vie de votre compte**, puis supprimées. Le détail
figure dans notre [politique de rétention](./politique-retention.md) : suppression sous 30 jours après
demande, purge des sauvegardes sous 30 jours, comptes inactifs supprimés/anonymisés après 24 mois.

## 6. Destinataires des données

- L'**équipe EduTutor** (accès strictement nécessaire).
- Nos **sous-traitants** : hébergeur (UE), Brevo (emails, UE), et le **fournisseur du modèle d'IA**
  qui génère les quiz. Le fournisseur principal retenu est **Mistral, hébergé dans l'Union
  européenne** (voir §7).

Vos données ne sont **jamais vendues** à des tiers.

## 7. Transferts hors Union européenne

Avec la configuration par défaut (**Mistral**, hébergé en 🇫🇷 UE), **aucune donnée n'est transférée
hors de l'Union européenne**. Si l'éditeur venait à utiliser un fournisseur d'IA situé hors UE
(par ex. Cerebras ou Groq), ce transfert serait encadré par des **clauses contractuelles types** et
signalé dans la présente politique avant toute activation.

## 8. Vos droits

Vous disposez des droits d'**accès** (Art. 15), de **rectification**, d'**effacement** (Art. 17),
de **portabilité** (Art. 20) et d'**opposition**. Concrètement :

- **Accès / portabilité** : vous pouvez obtenir l'ensemble de vos données dans un format réutilisable
  (JSON). Fonction d'export en libre-service prévue ; à défaut, la demande est traitée sous 48 h.
- **Effacement** : la suppression de votre compte efface **définitivement** toutes vos données
  (compte, cours, quiz, historique), y compris dans les sauvegardes sous 30 jours.

Pour exercer vos droits : `dpo@edututor.example`.

## 9. Cookies

Voir la [politique de gestion des cookies](./politique-cookies.md).

## 10. Contact & réclamation

Référent données : `dpo@edututor.example`. Vous pouvez également introduire une réclamation auprès de
la **CNIL** (www.cnil.fr).
