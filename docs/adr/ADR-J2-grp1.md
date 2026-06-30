# ADR-001 — Choix du backend LLM pour la génération de QCM

**Statut :** Accepté par l'équipe — à valider avec le PO  
**Date :** 2026-06-30  
**Contexte :** Perturbation J2 — latence inacceptable  
**Feature concernée :** F3 — Génération automatique de 10 QCM via LLM

## 1. Contexte

Le modèle actuellement configuré pour générer les QCM est **Llama 3.1 8B Instruct via Ollama local**. Ce choix est gratuit, local et cohérent avec l'objectif de souveraineté des données, mais il pose un problème majeur de performance : sur CPU, la génération d'un quiz peut prendre **2 à 5 minutes**. Le retour beta-test J2 signale déjà qu'une attente de **45 secondes** donne l'impression que l'application est cassée.

La génération de quiz est une fonctionnalité centrale du MVP. Une latence trop élevée dégrade directement l'expérience utilisateur et met en risque la démonstration devant le jury. L'équipe doit donc choisir un backend LLM plus adapté, tout en tenant compte de la qualité du JSON généré, du coût, du RGPD et de l'effort d'intégration.

## 2. Options envisagées

| Option | Avantages | Inconvénients | Verdict |
|---|---|---|---|
| Garder **Llama 3.1 8B via Ollama** | Gratuit, local, souverain, aucun transfert de données | Latence trop élevée, JSON parfois invalide, mauvaise expérience utilisateur | Rejeté comme choix principal |
| Utiliser **Cerebras `gpt-oss-120b`** | Très rapide, bonne qualité attendue, JSON plus fiable, free tier adapté, intégration déjà prévue par le factory pattern | Données envoyées vers un service cloud hors UE, dépendance réseau et quota free tier | Retenu comme choix principal |
| Utiliser **Mistral Small** | Fournisseur européen/français, bon compromis qualité/performance/RGPD, free tier | Moins rapide que Cerebras, dépendance cloud | Alternative RGPD |
| Utiliser **Qwen 3 8B via Ollama** | Gratuit, local, meilleur suivi d'instructions que Llama 3.1 8B | Latence CPU toujours problématique | Fallback souverain |

## 3. Décision retenue

Nous décidons d'utiliser **Cerebras `gpt-oss-120b`** comme backend principal pour la génération de QCM, afin de réduire fortement le temps de génération et de sécuriser la démonstration du MVP.

Le backend local **Ollama** reste conservé comme solution de repli, et **Mistral Small** est documenté comme alternative si la contrainte RGPD devient prioritaire.

## 4. Justification

Ce choix répond directement à la perturbation J2 : la priorité immédiate est de rendre la génération utilisable et démontrable. Le modèle actuel est cohérent sur le plan souveraineté, mais il ne répond pas au besoin d'une génération rapide. Cerebras apporte le meilleur compromis pour un hackathon noté en une semaine : vitesse, qualité de sortie, coût nul via free tier et intégration faible, car le projet permet déjà de changer de backend LLM via configuration.

L'équipe accepte le compromis suivant : **performance et qualité de démo prioritaires à court terme**, avec une surveillance explicite du risque RGPD et une alternative Mistral/Qwen si le PO impose une contrainte plus stricte sur les données.

## 5. Conséquences

### Positives

- Génération de QCM beaucoup plus rapide et mieux adaptée à l'usage étudiant.
- Risque réduit d'abandon utilisateur pendant l'attente.
- Meilleure fluidité pour la démonstration MVP.
- Bascule peu coûteuse techniquement grâce à la configuration existante des backends LLM.

### Négatives

- Dépendance à un fournisseur externe et à la connexion réseau.
- Free tier sans garantie de disponibilité ni SLA.
- Risque RGPD : les contenus de cours peuvent quitter le serveur local / l'UE.

### À surveiller

- Conserver la validation stricte du JSON généré côté backend.
- Prévoir un fallback `ollama` ou `mock` en cas d'indisponibilité de Cerebras.
- Documenter le transfert éventuel de données dans les éléments RGPD.
- Mesurer après bascule la latence réelle sur le cours de référence et vérifier que le temps de génération est inférieur ou égal à 15 secondes.
