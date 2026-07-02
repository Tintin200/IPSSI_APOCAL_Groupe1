# Conformité RGPD — Perturbation J3-bis (« La demande inattendue »)

> **Contexte.** En pleine journée du mercredi (sécurité J3 le matin + livraison MVP le soir),
> une **demande formelle d'accès aux données personnelles** (SAR — *Subject Access Request*,
> RGPD **Art. 15**) arrive. La perturbation J3-bis n'est **pas un sprint de code** : l'attendu
> est une **décision documentée** (priorisation + registre + politique), l'implémentation
> technique restant un bonus.

## Note de décision (arbitrage MoSCoW sous pression MVP)

Le mercredi est déjà saturé. On applique l'arbitrage suivant, annoncé au PO :

| Livrable J3-bis | Priorité | Décision |
|---|---|---|
| Politique de rétention écrite ([`politique-retention.md`](./politique-retention.md)) | **Must** | Fait ce jour — pur document, coût faible, obligation légale. |
| Registre des traitements ([`registre-traitements.md`](./registre-traitements.md)) | **Must** | Fait ce jour — cartographie exigée par l'Art. 30. |
| 4 pages légales rédigées (mentions, confidentialité, CGU, cookies) | **Must** | Rédigées en documents prêts à publier (voir ci-dessous). |
| Email pro de réponse au demandeur SAR ([`email-reponse-sar.md`](./email-reponse-sar.md)) | **Must** | Fait ce jour — la demande doit être traitée sous 48 h (cf. grille). |
| Endpoint d'export automatisé filtré par user (Art. 15/20) | **Should** | **Spécifié** ci-dessous ; implémentation = bonus post-MVP (ne bloque pas la livraison de mercredi soir). |

**Ce qu'on diffère et pourquoi :** l'endpoint d'export automatisé est *spécifié* mais son code
est reporté au Sprint 6 (jeudi) pour ne pas mettre en risque la livraison MVP de mercredi 17h45.
En attendant, une demande SAR est honorée **manuellement** (extraction DB filtrée par utilisateur)
dans le délai annoncé — ce qui suffit à la conformité tant que le volume est faible.

## Contenu du dossier `/docs/legal/`

- [`politique-retention.md`](./politique-retention.md) — durées de conservation, purge, archivage (3 sections).
- [`registre-traitements.md`](./registre-traitements.md) — registre des activités de traitement (Art. 30).
- [`email-reponse-sar.md`](./email-reponse-sar.md) — modèle d'email professionnel de réponse au demandeur.
- [`mentions-legales.md`](./mentions-legales.md) — mentions légales (éditeur, hébergeur…).
- [`politique-confidentialite.md`](./politique-confidentialite.md) — politique de confidentialité (RGPD).
- [`cgu.md`](./cgu.md) — conditions générales d'utilisation.
- [`politique-cookies.md`](./politique-cookies.md) — politique de gestion des cookies.

> **Rédigé sur le site (bonus) :** ces 4 pages existent en composants React vierges
> (`frontend/src/pages/legal/`). Le contenu ci-dessus est **prêt à y être collé** — le faire
> est un bonus technique qui ne change pas la conformité documentaire.

## Spécification de l'endpoint d'export (Should — implémentation bonus)

```
GET /api/accounts/export/          (authentifié)
```

- **Isolation stricte** : lit uniquement `request.user` et ses objets liés (`request.user.quizzes`).
  Aucun paramètre ne permet de viser un autre compte → un utilisateur n'exporte que **ses** données.
- **Format** : JSON *machine-readable* (RGPD Art. 20 — pas de PDF), en pièce jointe téléchargeable.
- **Périmètre** : compte (email, nom, prénom, date d'inscription, email vérifié) + quizz
  (titre, texte source, score, date) + questions (énoncé, options, bonne réponse, réponse donnée).
- **Effacement (Art. 17)** : déjà couvert — la suppression de compte (`DELETE /api/accounts/profile/`)
  est en cascade (profil + quizz + questions), aucune donnée résiduelle.

## Traçabilité des demandes (audit trail SAR)

Chaque demande d'accès/effacement est journalisée dans un registre simple : **qui** (email du
demandeur), **quand** (date de réception), **type** (accès / portabilité / effacement),
**statut** (reçue / en cours / traitée), **date de réponse**. Objectif : prouver le respect des
délais en cas de contrôle CNIL (cf. amende Klarna 1,2 M€ pour SAR mal traités).
