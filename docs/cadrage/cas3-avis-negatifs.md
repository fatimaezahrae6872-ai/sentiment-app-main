# Fiche de cadrage : Priorisation des avis négatifs

> Remplir chaque rubrique en une à trois phrases. Les rubriques marquées (obligatoire) sont vérifiées par `pytest`.

## Besoin en une phrase (obligatoire)
Le besoin est de détecter et de prioriser automatiquement le traitement des avis clients négatifs sur la plateforme e-commerce.

## Utilisateur final (obligatoire)
L'utilisateur final est l'agent du service client chargé de traiter les réclamations et de répondre aux clients.

## Approche retenue (obligatoire)

Cocher une seule case :

- [ ] Règles métier
- [x] Machine learning
- [ ] RAG
- [ ] Modèle génératif seul

## Justification (obligatoire, citer au moins deux critères de la grille : (a) données étiquetées, (b) vérifiabilité, (c) coût par requête, (d) conséquence d'une erreur)
L'approche Machine Learning est retenue car (a) les avis s'étiquettent très facilement grâce à la note en étoiles, et (c) face à des milliers d'avis quotidiens, un classifieur coûte très peu cher.

## Données nécessaires et leur origine (obligatoire)
Les données nécessaires sont l'historique complet des avis laissés par les clients, incluant le texte du commentaire et la note en étoiles correspondante.

## Métrique de succès et seuil d'acceptation (obligatoire)
La métrique choisie est le rappel sur la classe négative, avec un seuil d'acceptation fixé à 0,90.

## Conséquence d'une erreur et validation humaine prévue (obligatoire)
(d) Un avis mal classé a une conséquence peu grave, cela retarde simplement son traitement par l'équipe du service client sans impact critique.

## Risques éthiques ou de confidentialité (obligatoire)
Il faut veiller à l'anonymisation des avis traités pour ne pas exploiter ou exposer les données personnelles (noms, adresses) des clients.

## Approche écartée et pourquoi (facultatif)
L'approche par règles métier a été écartée car la diversité du vocabulaire utilisé dans les avis rendrait les règles trop complexes à maintenir.