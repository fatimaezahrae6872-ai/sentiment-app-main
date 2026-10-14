# Compte rendu global des TP1 à TP3

**Binôme :** Attoubi Fatima Zahra ([@fatimaezahrae6872-ai]) et Attoubi Fatima Zahra ([@fatimaezahrae6872-ai])
**Dépôt :** https://github.com/fatimaezahrae6872-ai/sentiment-app-main
**Date :** 14/10/2026

## 1. Le besoin et son cadrage

Le cas retenu est le cas 3 : un site e-commerce reçoit des milliers d'avis et veut détecter puis prioriser automatiquement les avis négatifs, afin que l'équipe relation client les traite avant les autres. Le problème n'est pas de comprendre chacun des avis, mais de ranger une file toujours plus longue de façon à ce que les clients mécontents soient rappelés en premier. L'utilisateur final est l'agent du service client ; il reçoit une liste ordonnée au lieu d'un flux brut.

L'approche retenue est le machine learning supervisé. La justification s'appuie sur trois critères de la grille. (a) Les données étiquetées existent déjà, sans travail d'annotation : la note en étoiles fournit une étiquette quasi gratuite, positive au-dessus de trois étoiles, négative en dessous. (c) À l'échelle de milliers d'avis par jour, un classifieur de type TF-IDF + régression logistique coûte presque rien par requête, là où faire appel à un grand modèle de langage à chaque avis aurait un coût récurrent bien plus élevé pour un gain marginal sur une décision simple. (d) La conséquence d'une erreur est faible : un avis mal classé est simplement traité un peu plus tard, et l'agent peut corriger le verdict à la lecture. C'est précisément parce que cette conséquence est faible qu'un modèle automatique est acceptable ici, contrairement au cas 1 de l'ordonnance, où la vérifiabilité (b) primait et où nous avons retenu des règles métier explicites. Le choix d'approche suit donc la conséquence de l'erreur, pas la mode technique.

## 2. L'architecture du dépôt

Le dépôt suit la structure du kit. Le dossier `data/` contient les jeux de données, `models/` le modèle entraîné et ses mesures, `src/` le code de la chaîne de traitement (génération du jeu, entraînement, évaluation, prédiction), `ui/` l'interface Streamlit, `tests/` les tests `pytest`, `docs/` les fiches de cadrage et ce compte rendu, et `scripts/` les utilitaires d'installation et de vérification. Le dossier `app/` reste vide pour l'instant : l'API viendra plus tard, l'interface de démonstration étant suffisante pour ce rendu.

La règle de versionnement est volontairement sélective, et c'est le point d'architecture le plus important. `models/metrics.json` est dans Git, alors que `models/sentiment.joblib` ne l'est pas. La raison est double. D'une part, le fichier `sentiment.joblib` est un binaire lourd — plusieurs mégaoctets — et surtout reconstructible en une commande (`python src/train.py`), donc le stocker dans Git ferait grossir le dépôt sans rien apporter. D'autre part, `metrics.json` est un petit fichier texte, mais il est la seule preuve que le modèle atteint le seuil de qualité fixé. Il permet à l'intégration continue de tourner sur une machine neuve qui ne possède pas le modèle : les deux tests qui lisent `metrics.json` passent partout, tandis que les deux tests qui appellent réellement `predict()` sont ignorés faute de modèle. C'est exactement pour cette raison qu'il est versionné. Le même raisonnement s'applique aux données : `data/sample_reviews.csv` est versionné car il sert d'exemple d'entrée, tandis que `data/demo_reviews.csv` est exclu par la règle `data/*` parce qu'il se régénère avec `python src/make_demo.py`. Deux artefacts reconstructibles restent donc hors de Git, et seules les mesures — le résultat qui compte — sont conservées.

## 3. Le modèle

Deux jeux ont été utilisés. Le jeu de démonstration, `data/demo_reviews.csv`, est généré par `src/make_demo.py` : 290 avis déterministes, dont 80 nettement positifs, 80 nettement négatifs, 100 volontairement mitigés et 30 neutres. Le vrai jeu est Allociné, récupéré par `src/train.py --allocine` : 20 000 avis d'avis de films, dont 10 204 négatifs.

Le découpage est fait **avant** toute vectorisation, avec `train_test_split(test_size=0.2, stratify=..., random_state=42)`. C'est un point de méthode essentiel : si l'on vectorisait d'abord le texte entier puis que l'on séparait, le vocabulaire et les fréquences de termes de l'ensemble de test fuiteraient dans l'entraînement, et le score obtenu serait artificiellement optimiste. La stratification garantit en plus la même proportion de classes des deux côtés, et la graine fixe rend le découpage reproductible d'une exécution à l'autre. Le pipeline (`sklearn.pipeline.Pipeline`) enferme le vectoriseur TF-IDF (`ngram_range=(1,2)`, `min_df=2`, `sublinear_tf=True`) et le classifieur (régression logistique, `max_iter=1000`, `class_weight="balanced"`) dans un seul objet : la même transformation s'applique forcément à l'entraînement et en production, ce qui évite la classe d'erreurs où le pré-traitement diverge entre les deux.

| Jeu | Avis | Vocabulaire | F1 macro | Rappel négatif |
| --- | --- | --- | --- | --- |
| Démonstration | 290 (232 entr. / 58 test) | 316 | 0,879 | 0,931 |
| Allociné | 20 000 (16 000 entr. / 4 000 test) | 149 529 | 0,920 | 0,911 |

Matrice de confusion sur la démonstration (58 avis, 7 erreurs) :

| | prédit négatif | prédit positif |
| --- | --- | --- |
| **vrai négatif** | 27 | 2 |
| **vrai positif** | 5 | 24 |

Ligne par ligne : sur les 29 avis réellement négatifs, 27 sont reconnus et 2 passent en positif — ce sont des faux négatifs, les plus gênants pour l'objectif, mais seulement 2. Sur les 29 avis réellement positifs, 24 sont reconnus et 5 sont étiquetés négatifs — des faux positifs, qui enverraient un client satisfait vers l'agent. La précision sur la classe négative (27 / 32 = 0,844) est plus faible que le rappel (0,931) : le modèle préfère sur-signaler le négatif, ce qui est cohérent avec `class_weight="balanced"` et acceptable, car un faux positif se corrige à la lecture alors qu'un faux négatif fait manquer un client mécontent.

Matrice de confusion sur Allociné (4 000 avis, 318 erreurs) :

| | prédit négatif | prédit positif |
| --- | --- | --- |
| **vrai négatif** | 1 860 | 181 |
| **vrai positif** | 137 | 1 822 |

Sur les 2 041 avis négatifs, 1 860 sont corrects (rappel 0,911) ; sur les 1 959 avis positifs, 1 822 sont corrects. Les erreurs sont équilibrées entre les deux sens (181 faux positifs, 137 faux négatifs), et le taux d'erreur global de 7,9 % correspond bien au F1 macro de 0,920. Le classifieur est donc légèrement meilleur qu'annoncé dans la fiche de cadrage, puisque le seuil d'acceptation était un F1 macro de 0,85.

## 4. L'analyse des erreurs

Sur le jeu de démonstration, les 7 erreurs sont **toutes** des avis mitigés, jamais des avis nets. Le mécanisme est identifiable : le modèle est un sac de mots avec des bigrammes. Il additionne des indices positifs et négatifs, mais il n'a aucune notion de l'ordre de la phrase, et donc aucun moyen de voir le retournement qui décide du sens. Les avis du jeu de démonstration ont tous la même structure : une première moitié d'un sentiment, une charnière (« mais finalement », « sauf que ensuite », « avant que »), puis une seconde moitié du sentiment opposé — et l'étiquette suit la seconde moitié, celle qui conclut.

Trois exemples tirés de nos propres données l'illustrent :

- « Service impeccable et livraison en avance les premiers jours, avant que notice incompréhensible et pièces manquantes. » — vrai `negative`, prédit `positive` (confiance 0,693). Le modèle s'arrête sur les mots positifs de la première moitié ; la charnière « avant que » invalide tout ce qui précède, mais il ne le voit pas.
- « Facture erronée, aucun geste commercial sur le papier, puis rapidement service impeccable et livraison en avance. » — vrai `positive`, prédit `negative` (confiance 0,547). Là, c'est le négatif de la première moitié qui pèse, et la conclusion positive passe inaperçue.
- « Commande jamais reçue et remboursement en attente au début, avant que emballage soigné et finitions propres. » — vrai `positive`, prédit `negative` (confiance 0,545). Même cause.

Pour corriger ces erreurs, il faudrait un modèle sensible à l'ordre et à la portée des mots : des embeddings de phrases, voire un encodeur de type transformer, ou a minima des traits construits qui repèrent explicitement les charnières de retournement. Ce n'était pas le sujet de cette séance, qui portait sur la méthode complète — découpage avant vectorisation, pipeline scellé, mesures sur les deux jeux — et non sur la performance fine du classifieur. La limite est ici mesurable et comprise, ce qui est plus utile qu'un score un peu meilleur obtenu sans explication.

## 5. L'interface et le seuil d'incertitude

L'interface `ui/app.py` reprend la fonction `predict()` du TP2 sans dupliquer la logique de prédiction, et ajoute un seuil d'incertitude fixé dans `ui/helpers.py` : en dessous de **0,60** de confiance, la prédiction n'est plus tranchée mais affichée « à vérifier par un agent ». C'est la traduction concrète de la validation humaine prévue au cadrage.

L'effet est mesurable sur les jeux de test. Sur les 58 avis de démonstration, 16 (27,6 %) passent sous 0,60 et partent vers un agent. Sur les 4 000 avis d'Allociné, 509 (12,7 %) sont signalés. Le réglage importe : à 0,50, aucun avis n'est signalé (le modèle binaire descend très rarement sous la barre des 50 %) ; à 0,75, ce sont 29 avis sur 58 (50 %) et 1 626 sur 4 000 (40,7 %) qui repartent vers l'agent. Autrement dit, à 0,60 on envoie 5 des 7 erreurs de la démonstration vers l'agent — seules deux erreurs, à 0,641 et 0,693 de confiance, échappent encore au filet. Ce compromis nous paraît juste du point de vue de l'équipe relation client : signaler un tiers des avis démonstration est soutenable pour un humain, et correctement ciblé, puisque ce sont majoritairement les avis mitigés — ceux que le modèle comprend mal — qui sont envoyés à l'agent. Monter à 0,75 aurait un coût humain trop élevé pour un gain limité.

## 6. Ce qui a échoué

Trois blocages réels. Le premier est venu du versionnement : après le premier entraînement, `models/sentiment.joblib` apparaissait dans `git status` à côté de `metrics.json`, signe que la règle du `.gitignore` était mal comprise. Nous avons dû reprendre l'ordre des exceptions (`models/*`, puis `!models/metrics.json`) pour que seul le fichier de mesures soit suivi. Le deuxième a été l'identité Git : les deux membres poussaient sous des `user.name` qui n'étaient pas assez distincts, et le test `test_each_member_has_committed` réclamait deux auteurs. Corriger une identité après coup oblige à reconfigurer puis à refaire un commit réel, ce qui nous a fait perdre du temps. Le troisième a été l'environnement : lancer `pytest` sans activer le virtualenv provoque des `ModuleNotFoundError` trompeurs, et la lecture du jeu Allociné a longtemps buté sur l'absence du moteur `pyarrow` nécessaire à `read_parquet`. Dans les trois cas, la solution est ensuite devenue un réflexe : vérifier `git status` après chaque génération d'artefact, fixer `user.name` avant le premier commit, et toujours travailler venv activé. Nous n'avons pas réussi à tenir le planning prévu pour rédiger au fil de l'eau les notes de semaine : `docs/semaine2.md` et `docs/semaine3.md` n'ont pas été écrits, et ce compte rendu a dû reconstituer après coup les chiffres — ce qui a au moins l'avantage de les faire vérifier par les tests.

## 7. Ce que vous feriez autrement

Avec le recul, nous commencerions par écrire le seuil d'acceptation et le jeu de test **avant** d'entraîner, pour éviter de choisir la mesure après avoir vu le score. Avec les moyens d'une vraie équipe, nous ajouterions un jeu de validation dédié pour régler le seuil d'incertitude de façon chiffrée plutôt qu'à l'estime, nous versionnerions les données avec DVC et les expériences avec MLflow au lieu de nous fier à un seul `metrics.json`, et nous brancherions un suivi de dérive pour détecter quand le vocabulaire des avis s'éloigne de celui de l'entraînement. Enfin, nous documenterions les semaines 2 et 3 au moment où elles se font, et non à la fin.

## 8. Répartition du travail

| Membre | Ce qu'il a pris en charge |
| --- | --- |
| Attoubi Fatima Zahra (@fatimaezahrae6872-ai) | Cadrage des trois cas, chaîne `src/` (génération du jeu, entraînement, évaluation), interface `ui/`, rédaction du compte rendu |
| Attoubi Fatima Zahra (@fatimaezahrae6872-ai) | Mise en place de l'environnement et du dépôt, tests et `.gitignore`, relecture et vérification des chiffres |

## Outils d'aide à la rédaction

Un assistant d'IA a été utilisé pour structurer ce compte rendu et repondre a ce compte-rendu
