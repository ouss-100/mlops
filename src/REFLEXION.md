# Réflexion — TP2

1. Quel est le meilleur des 3 modèles ? Peut-on le prouver ?
   -> Oui pour la traçabilité : `git log --oneline` montre les 3 commits distincts (Essai 1, 2, 3), chacun avec un message explicite, et le notebook de chaque commit conserve ses sorties (Essai 1 = 0.99, Essai 2 = 0.99, Essai 3 = 1.00). Chaque résultat est donc retrouvable et vérifiable. En revanche, l'Essai 3 n'est pas « prouvé meilleur » : l'écart est d'un seul client sur 100, et la fuite de données (la règle de churn dépend de `consommation` et `anciennete`) rend la comparaison peu significative. Git trace les essais, il ne remplace pas un suivi des métriques (MLflow, TP3-4).

2. Peut-on redonner exactement le modèle de l'Essai 1 ?
   -> Oui, pour le code : `git checkout <commit> -- churn_model.ipynb` restaure le notebook de l'Essai 1 (100 arbres, profondeur 5, avec `salaire`) en une commande, sans rien perdre de l'historique. Limite : `model.pkl` est dans `.gitignore`, donc le fichier binaire lui-même n'est pas versionné ; il faut ré-exécuter le notebook pour le régénérer.

3. Un camarade peut-il reproduire notre travail sur une autre machine ?
   -> Oui en grande partie : `git clone` fournit le code, les données (`churn_data.csv`) et tout l'historique. Il doit recréer son `venv` et réinstaller les bibliothèques (`requirements.txt`), puis ré-entraîner le modèle. Les versions exactes des bibliothèques ne sont pas verrouillées : le problème « ça marche chez moi » n'est que partiellement résolu.

## TP3 — Réponses grâce à MLflow

1. Quel est le meilleur modèle ? Peut-on le prouver ?
   -> Oui. Chaque essai est un run MLflow avec ses paramètres et ses 4 métriques. En triant par f1 (interface ou `search_runs`), les essais 1 et 3 sont à égalité (f1 = 0.889) et l'essai 2 est nettement plus faible (f1 = 0.333, recall = 0.2 : il rate 4 départs sur 5), alors que son accuracy (0.96) paraît correcte : l'accuracy seule est trompeuse quand peu de clients partent. À égalité, on préfère l'essai 3, plus simple (sans `salaire`). Réserve : seulement 5 clients qui partent dans le jeu de test, donc ces chiffres restent des indications.

2. Peut-on redonner exactement le modèle de l'Essai 1 ?
   -> Oui : chaque run conserve son propre modèle, rechargeable avec `mlflow.sklearn.load_model("runs:/<id>/model")`, sans ré-entraîner et sans jamais être écrasé.

3. Un camarade peut-il reproduire notre travail sur une autre machine ?
   -> En partie. Le code (Git), les paramètres et les versions de bibliothèques (`requirements.txt` de chaque run) sont connus. Mais `mlruns/` et `mlflow.db` restent sur ma machine (hors Git) : le camarade doit relancer les scripts pour obtenir ses propres runs, et l'environnement identique reste à garantir (Docker, TP5-6).
