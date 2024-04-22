# Grains & Co.

L'objectif de ce TD est de valider l'utilisation des notions de Technologies Logicielles vu en cours (Python, Git, Test et Docker) et de démontrer votre capacité à organiser votre travail. Concernant Git, les critères d'évaluation sont : la création d'issues et de branches, le recours à des Pull Requests, pas de commit réalisés directement dans la branche principale, réalisation de commits atomiques.

Le thème de ce TD est une entreprise de grains de café. Vous devrez créer:

        - un script python `seeder.py` permettant d'insérer des sachets de grains de café dans une base de donnée;
        - un script python `coffee.py` permettant de récupérer des informations sur les sachets de grains de café disponibles et à réaliser des traitements su la base de données.

Le premier traitement que l'on souhaite réaliser consiste à itérer sur la base de données et tester si les données associées aux grains de café sont valides.

# Consignes

Créez un dépôt **privé** nommé TD_COFFEE_GRAINS sur Github.

Ajoutez moi à ce dépôt

Publiez **régulièrement** votre code et ses modifications dans le dépôt !

# Exercice 1 - Implantation des classes et tests

Vous allez débuter le projet en créant :

- une classe `Grain` qui  :
        - encapsule une origine, un arome, une année de récolte et une valeur de corps;
        - possède une méthode publique `is_valid` qui retourne `True` si les données sont valides, `False` sinon.
Les données sont valides si :
        - l'origine est parmi "Vietnam", "Kenya", "Colombie" ou "Guatemala";
        - l'arome est parmi "Floral", "Fruite", "Chocolate", "Epice";
        - l'année de récolte est un entier non divisible par 5;
        - la valeur de corps est un entier entre 1 et 9.

- une classe `Sachet` qui encapsule un poids en grammes, un type de paquet, une liste de grains, une composition en grains telle que :
        - le poids est un multiple de 500 inférieur ou égale à 2000;
        - le type de paquet est "pure" ou "mixte";
        - la liste de grains est composée d'une instance de chaque type de grain composant le sachet, e.g. un sachet pure peut être composé d'un unique `Grain("Vietnam", "Floral", 2020, 6)` et un sachet mixte d'une collection de différents grains;
        - la composition reflète le pourcentage de chaque grain dans le sachet, e.g. `[30, 40, 30]` (la somme doit être égale à 100) 

Dans cet exercice, il est attendu que vous réalisiez des **tests unitaire** associés aux deux classes.

# Exercice 2 - Utilisation de Docker pour peupler une base de donnée MongoDB avec un script Python (procédure interactive)

Ici, nous allons jouer avec deux images Docker qui devrons pouvoir communiquer entre elles.
Pour cela, nous allons utiliser Docker pour créer un réseau que nous nommerons `mynet`

```
docker network create mynet
```

Pour les utilisateurs de Windows, il est possible que cet appel ne fonctionne pas. Dans ce cas, passez par la commande

```
docker network create --driver nat mynet
```

Commencez par lancer un premier conteneur basé sur l'image `mongo` que vous nommerez `mongodb` et que vous connecterez au réseau `mynet` (utilisez l'option `--network mynet`).

Ensuite, lancez un second conteneur basé sur l'image `python:alpine` que vous nommerez `python-mongo` et que vous connecterez au réseau `mynet`. Afin d'éviter que le conteneur de l'image python ne se ferme automatiquement, il faudra utiliser le mode "foreground" (-t, -i ou -it).

Dans le conteneur `python-mongo` installez la paquet `pymongo` que nous allons utiliser pour remplir la base de donnée.
Utilisez l'interpréteur python du conteneur, pour peupler la base de données avec des informations associées aux grains (origine, arôme, année, corps)  en vous inspirant du code suivant :

```python
import pymongo
# Import de la classe MongoClient qui nous permettra de nous connecter a la base de donnees MongoDB
from pymongo import MongoClient

client = MongoClient(host="A_REMPLIR")

# Acces a la base de donnees "NOM_BASE_DE_DONNEES"
db = client["NOM_BASE_DE_DONNEES"]

# Acces a la collection "NOM_COLLECTION"
col = db["NOM_COLLECTION"]

# Ajout d'un 'fruit' dans la collection
fruit = {
        "nom": "banane",
        "couleur": "jaune"
}
res = col.insert_one(fruit)

# Verification de l'ajout
print(f'Le fruit {res.inserted_id} a bien ete cree')

# Localise et affiche le fruit
col.find_one()
```

En parallèle, vous pouvez vous assurer que la base de donnée à bien été remplie depuis le conteneur `mongo`.
Pour cela, interagissez avec celui-ci en exécutant la commande `mongosh`.
Une fois réalisé, utilisez la base de donnée que vous défini depuis le conteneur python.
`use NOM_BASE_DE_DONNEES`
Ensuite, interrogez la base pour récupérer un élément qui compose la collection que vous avec définit.
`db.NOM_COLLECTION.findOne()`

**Une fois cette étape réalisée, appelez l'enseignant pour qu'il constate que vous avez validés cet exercice !**

# Exercice 3 - Création d'un script Python peuplant la base de données et publication de l'image associée

Ecrivez un script Python `seeder.py` permettant de peupler la base de donnée.
Pour cela, vous allez utiliser le code contenu dans les tests réalisés à l'Exerice 1 et ajouter des entrées valides et non valides.

Une fois le scrip écrit, il ne vous reste plus qu'à définir le Dockerfile associé où l'on doit retrouver:

    - l'installation des dépendances nécessaires
    - la copie des données nécessaires
    - l'exécution du script `seeder.py`

Maintenant, vous allez construire votre image en la taggant par `$DOCKERID/seed_mongo` (où DOCKERID est votre identifiant docker hub).
Puis, après vous être connecté à Docker Hub, vous allez publier votre image.

# Exercice 4 - Création d'un script Python traitant la base de donnée

Ecrivez un script Python `coffee.py` qui doit se connecter à la base de données et regarder que les informations des grains sont valides.
Dans le cas où une donnée n'est pas valide, il doit l'afficher à l'écran.

Note : pour itérer sur une collection obtenus via `pymongo`, vous pouvez vous inspirer du snippet suivant:

```python
fruits = col.find()
for fruit in fruits:
        nom = fruit["nom"]
        couleur = fruit["couleur"]
```

# Exercice 5 - Assemblage des différents conteneurs avec Docker Compose !

Votre fichier `docker-compose.yml` va se composer de trois services :

    - `mongodb` : que vous allez lancer à partir de l'image `mongo:latest`
    - `seed_mongo` :  que vous allez lancer à partir de votre image `$DOCKERID/seed_mongo` (ou de celle d'un de vos camarades) et qui dépendra de `mongodb`
    - `coffee` : que vous allez construire à partir des sources de l'Exercice 4 (il vous faudra faire un nouveau Dockerfile dédié à ce conteneur) et qui dépendra de seed_mongo

Note : dans le cas du conteneur `mongodb`, n'oubliez pas que le service `mongod` utilise le port 27017.

Vérifiez ensuite que tout cela fonctionne :
`docker-compose up --build`
et, depuis un autre terminal
`docker exec -it NOM_DU_CONTENEUR bash`
et enfin l'exécution de votre code python réalisant le traitement sur la base de donnée.

# Exercice 6 - Bonus 

Modifiez `coffee.py` pour en faire une interface en ligne de commande (CLI).
Pour commencer vous pouvez recouvrir vos développements existant, i.e. ajouter une option "--check" permettant de regarder si les informations sont valides.
Ensuite vous pouvez proposer d'autres options :
    - rechercher les grains provenant d'une certaine origine, année, ...
    - supprimer les données invalides dans la base de données
    - ...
