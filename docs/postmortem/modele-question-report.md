# Spécification — Modèle `QuestionReport` (boucle de signalement)

> **Statut :** spécifié (document). **Implémentation :** bonus technique post-MVP.
> Objectif : outiller la **boucle de feedback** (promesse J1) en permettant à un utilisateur de
> **signaler une question erronée**, et à l'équipe de mesurer/traiter ces signalements.

## 1. Besoin

Suite au post-mortem (erreurs factuelles signalées par Mme Lefèvre), il manque un **canal de
signalement** persistant. `QuestionReport` capte chaque signalement, relié à la question et à
l'utilisateur, avec un motif et un statut de traitement.

## 2. Emplacement

App Django **`quizzes`** (à côté de `Quiz` et `Question`, cf. `backend/quizzes/models.py`).

## 3. Modèle proposé

```python
class QuestionReport(models.Model):
    """Signalement d'une question jugée erronée par un utilisateur (boucle qualité J4)."""

    class Reason(models.TextChoices):
        FACTUAL = "factual", "Erreur factuelle (mauvaise bonne réponse)"
        AMBIGUOUS = "ambiguous", "Question ambiguë"
        OFF_TOPIC = "off_topic", "Hors-sujet vs le cours"
        LANGUAGE = "language", "Problème de langue / formulation"
        OTHER = "other", "Autre"

    class Status(models.TextChoices):
        OPEN = "open", "À traiter"
        REVIEWING = "reviewing", "En cours de revue"
        RESOLVED = "resolved", "Traité"
        REJECTED = "rejected", "Rejeté (non fondé)"

    question = models.ForeignKey(
        "quizzes.Question", on_delete=models.CASCADE, related_name="reports",
    )
    reporter = models.ForeignKey(
        settings.AUTH_USER_MODEL, on_delete=models.SET_NULL, null=True, related_name="question_reports",
    )
    reason = models.CharField(max_length=20, choices=Reason.choices, default=Reason.FACTUAL)
    comment = models.TextField(blank=True, help_text="Précision libre de l'utilisateur.")
    status = models.CharField(max_length=20, choices=Status.choices, default=Status.OPEN)
    created_at = models.DateTimeField(auto_now_add=True)
    resolved_at = models.DateTimeField(null=True, blank=True)

    class Meta:
        ordering = ["-created_at"]

    def __str__(self) -> str:
        return f"Report<{self.question_id}:{self.reason}:{self.status}>"
```

## 4. Points d'intégration (bonus)

- **Migration** : `python manage.py makemigrations quizzes` → `0003_questionreport`.
- **Endpoint** : `POST /api/quizzes/questions/<question_id>/report/` (authentifié) — crée un
  signalement pour une question d'un quiz **appartenant à l'utilisateur** (isolation, cf. patterns
  de `quizzes/views.py`).
- **Admin** : enregistrer `QuestionReport` dans `quizzes/admin.py` avec `list_filter` sur
  `status`/`reason` pour le traitement par l'équipe.
- **UX** : bouton « Signaler cette question » dans le parcours de révision (frontend).

## 5. KPI de suivi

- Nombre de signalements ouverts / traités.
- Taux de signalements par cours / par matière.
- Délai moyen de traitement.
- Corrélation avec l'audit qualité (les questions ❌ de l'audit sont-elles signalées ?).
