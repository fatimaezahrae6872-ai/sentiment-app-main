# Fiche de cadrage : Questions sur le règlement intérieur

> Remplir chaque rubrique en une à trois phrases. Les rubriques marquées (obligatoire) sont vérifiées par `pytest`.

## Besoin en une phrase (obligatoire)
Le besoin est de répondre de manière automatique et précise aux questions des étudiants concernant le règlement de l'école.

## Utilisateur final (obligatoire)
L'utilisateur final de cette application est l'étudiant de l'école qui cherche une information précise.

## Approche retenue (obligatoire)

Cocher une seule case :

- [ ] Règles métier
- [ ] Machine learning
- [x] RAG
- [ ] Modèle génératif seul

## Justification (obligatoire, citer au moins deux critères de la grille : (a) données étiquetées, (b) vérifiabilité, (c) coût par requête, (d) conséquence d'une erreur)
L'approche RAG est choisie car (b) la réponse doit absolument citer l'article exact, et (a) nous n'avons pas de questions étiquetées au départ. De plus, (c) le volume d'appels est faible, ce qui rend le coût de cette solution acceptable.

## Données nécessaires et leur origine (obligatoire)
Les données nécessaires sont les documents textuels officiels qui constituent le règlement intérieur de l'établissement scolaire.

## Métrique de succès et seuil d'acceptation (obligatoire)
La métrique de succès est le taux de réponses correctes avec une citation exacte évaluée sur un jeu de 20 questions.

## Conséquence d'une erreur et validation humaine prévue (obligatoire)
Si une erreur survient, l'étudiant reçoit une information erronée sur ses droits ou devoirs. Un modèle génératif seul est refusé car il inventerait de faux articles.

## Risques éthiques ou de confidentialité (obligatoire)
Aucun risque majeur n'est identifié si les documents utilisés sont publics et qu'aucune donnée personnelle des étudiants n'est traitée.

## Approche écartée et pourquoi (facultatif)
L'approche par modèle génératif seul a été totalement écartée pour éviter les hallucinations.