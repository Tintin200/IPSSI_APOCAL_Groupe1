# Email professionnel — Réponse à une demande d'accès (SAR, RGPD Art. 15)

> Modèle d'email de réponse au demandeur. À personnaliser (nom, dates). Ton : professionnel,
> factuel, respectueux du délai légal (réponse sous 1 mois — ici traitée sous 48 h conformément
> à l'engagement de service).

---

**Objet :** Votre demande d'accès à vos données personnelles — EduTutor IA

Bonjour [Prénom Nom],

Nous accusons réception de votre demande d'accès à vos données personnelles, reçue le [date],
au titre de l'article 15 du Règlement général sur la protection des données (RGPD). Nous vous
remercions de votre confiance et vous confirmons que votre demande a été **prise en compte**.

**Ce que nous vous transmettons.** Vous trouverez ci-joint l'ensemble des données personnelles que
nous détenons vous concernant, dans un format **structuré et lisible par machine** (JSON), afin que
vous puissiez les réutiliser librement (droit à la portabilité, article 20). Cet export contient :

- les informations de votre **compte** (email, nom, prénom, date d'inscription, statut de vérification) ;
- l'ensemble de vos **cours déposés** et des **quiz générés** (énoncés, options, bonnes réponses) ;
- votre **historique de scores** et de progression.

**Finalités et durée de conservation.** Ces données sont utilisées uniquement pour vous fournir le
service (création de compte, génération de quiz, suivi de progression). Elles sont conservées pendant
la durée de vie de votre compte, conformément à notre politique de rétention, disponible sur demande.

**Vos autres droits.** Vous pouvez à tout moment demander la **rectification** de vos données, leur
**effacement** (article 17 — la suppression de votre compte efface définitivement l'ensemble de vos
données, y compris dans nos sauvegardes sous 30 jours), ou vous **opposer** à certains traitements.
Pour exercer ces droits, il vous suffit de répondre à cet email.

**Réclamation.** Si vous estimez que vos droits ne sont pas respectés, vous pouvez introduire une
réclamation auprès de la CNIL (www.cnil.fr).

Nous restons à votre disposition pour toute question complémentaire.

Bien cordialement,

**[Prénom Nom]**
Référent données personnelles — EduTutor IA
dpo@edututor.example

*Pièce jointe : export-donnees-[utilisateur]-[AAAAMMJJ].json*

---

## Notes internes (ne pas envoyer)

- **Délai :** demande reçue le [date] → réponse envoyée le [date] (**< 48 h**, bien en deçà du mois légal).
- **Vérification d'identité :** confirmer que le demandeur est bien le titulaire du compte (email de
  connexion) avant tout envoi — anti-usurpation.
- **Isolation :** l'export ne contient **que** les données du compte concerné (filtrage par utilisateur).
- **Traçabilité :** consigner la demande dans le journal SAR (qui / quand / type / statut / date de réponse).
