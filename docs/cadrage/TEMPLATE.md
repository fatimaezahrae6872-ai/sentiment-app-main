# Fiche de cadrage : Triage des appels d'urgence

> Remplir chaque rubrique en une à trois phrases. Les rubriques marquées (obligatoire) sont vérifiées par `pytest`.

## Besoin en une phrase (obligatoire)

Classer automatiquement chaque appel d'urgence par niveau de gravité, afin que le centre d'appels envoie les secours dans le bon ordre.

## Utilisateur final (obligatoire)

Les opérateurs du centre d'appels d'urgence, qui reçoivent une file ordonnancée au lieu d'un flux brut.

## Approche retenue (obligatoire)

Cocher une seule case :

- [x] Règles métier
- [ ] Machine learning
- [ ] RAG
- [ ] Modèle génératif seul

## Justification (obligatoire, citer au moins deux critères de la grille : (a) données étiquetées, (b) vérifiabilité, (c) coût par requête, (d) conséquence d'une erreur)

(b) Les protocoles de triage médicaux existent déjà et sont stables, donc des règles explicites suffisent et chaque décision reste justifiable devant l'opérateur. (d) Un appel grave classé mineur peut coûter une vie : le système doit obéir à des règles vérifiables, pas à une boîte noire.

## Données nécessaires et leur origine (obligatoire)

Les protocoles officiels de triage (fiches de régulation médicale), fournis par le centre lui-même. Aucun enregistrement d'appel n'est nécessaire, et aucune donnée réelle de santé n'est utilisée dans le projet.

## Métrique de succès et seuil d'acceptation (obligatoire)

La sensibilité (rappel) sur les appels critiques, mesurée sur un jeu de cas annotés par le médecin régulateur ; acceptation à partir de 1,0 sur les cas critiques et 0,80 d'exactitude globale.

## Conséquence d'une erreur et validation humaine prévue (obligatoire)

 Un mauvais classement peut retarder des secours vitaux ; l'opérateur valide donc chaque niveau proposé et peut le corriger avant tout envoi.

## Risques éthiques ou de confidentialité (obligatoire)

Les appels contiennent des données de santé et personnelles très sensibles : traitement en interne, anonymisation, et aucune conservation de l'audio.

## Approche écartée et pourquoi (facultatif)

Machine learning : impossible faute d'assez d'appels étiquetés par gravité, et le manque d'explicabilité serait inacceptable pour une décision vitale.
