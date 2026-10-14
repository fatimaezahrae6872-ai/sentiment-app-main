<<<<<<< HEAD
# Fiche de cadrage : Ordonnances incomplètes en pharmacie
=======
git init
git add -A# Fiche de cadrage : Ordonnances incomplètes en pharmacie
>>>>>>> 6a6f56cd7dbf7523c70c39cfdb7e1e1d53f908d3

> Remplir chaque rubrique en une à trois phrases. Les rubriques marquées (obligatoire) sont vérifiées par `pytest`.

## Besoin en une phrase (obligatoire)
Le besoin principal est de détecter automatiquement les ordonnances médicales incomplètes au comptoir.

## Utilisateur final (obligatoire)
L'utilisateur final de ce système est le pharmacien ou le préparateur en pharmacie.

## Approche retenue (obligatoire)

Cocher une seule case :

- [x] Règles métier
- [ ] Machine learning
- [ ] RAG
- [ ] Modèle génératif seul

## Justification (obligatoire, citer au moins deux critères de la grille : (a) données étiquetées, (b) vérifiabilité, (c) coût par requête, (d) conséquence d'une erreur)
L'approche par règles métier est choisie car (a) nous n'avons pas de données étiquetées au départ, et (b) la vérification doit rester explicable pour le pharmacien.

## Données nécessaires et leur origine (obligatoire)
Les données nécessaires sont les règles médicales strictes définissant les champs obligatoires d'une ordonnance valide.

## Métrique de succès et seuil d'acceptation (obligatoire)
La métrique de succès est de bloquer 100 % des ordonnances présentant un champ obligatoire manquant.

## Conséquence d'une erreur et validation humaine prévue (obligatoire)
(d) Une ordonnance validée à tort a une conséquence très grave pour le patient, une validation humaine par le pharmacien reste donc obligatoire.

## Risques éthiques ou de confidentialité (obligatoire)
Il s'agit de données de santé très sensibles, une grande vigilance est requise et aucune donnée réelle ne doit être utilisée dans ce projet.

## Approche écartée et pourquoi (facultatif)
L'approche Machine Learning est écartée car il faudrait d'abord constituer un grand jeu de données étiqueté.