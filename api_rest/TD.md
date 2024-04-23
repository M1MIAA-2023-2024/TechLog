L'objectif de cd TD est de vous donner une introduction aux API Rest. Vous ne traiterez pas l'asynchronisme ni les aspects liés à la sécurité / authentification / autorisation.

# Exercice 1 - Service de santé

Créez un modèle `Patient` qui sera utilisé pour stocker des données de santé associées à un patient. Les informations à stocker pour chaque patient sont: le nom, le prénom et le numéro de sécurité social (aussi noté *ssn*). Le numéro renseigné doit pouvoir être décrypté de la manière suivante:
- le premier chiffre vaut 1 si c'est un homme, 2 si c'est une femme;
- le deuxième et troisième chiffre renseignent l'année de naissance;
- le quatrième et cinquième chiffre indiquent le mois de naissance;
- le sixième et septième chiffre indiquent le département de naissance;
- les chiffres huit à dix représentent l'identifiant du pays de naissance;
- les chiffres onze à treize indiquent l'indice de naissance;
- les deux derniers chiffres représentent une clef de contrôle, i.e. complément à 97 du nombre formé par les treizes premiers chiffres du *ssn* modulo 97. 

Une fois créé, réalisez une API Rest (avec `fastapi`) permettant d'accéder à une base de données `mongodb` et fournissant les points de terminaison suivants:

- [GET] patients
- [POST] patients
- [GET] patients/ssn
- [DELETE] patients/ssn
- [UPDATE] patients/ssn
- [POST] patients/ssn

Dans le cas de l'endpoint `[POST] patients/ssn`, assurez vous qu'une `HTTPException` soit retournée lorsqu'un patient avec le même *ssn* est déjà enregistré.

**Indications** : vous utiliserez deux conteneurs, le premier remplira une base de donnée (cf. TD Docker), le second exposera l'API Rest sur le port 3000. Il est attendu que votre modèle soit défini dans un fichier `model_patient.py`, que votre application RestAPI soit définie dans un fichier `app.py`.

**Note** : utilisez des `validator` pour vous assurer que le *ssn* est valide (https://pydantic-docs.helpmanual.io/usage/validators/).

# Exercice 2 - Paramétrisation

Retravaillez l'endpoint `[GET] patients/ssn` pour qu'il affiche plus d'informations. Celui-ci devra permettre d'afficher (si spécifié) les informations suivantes :

- le sexe;
- l'année de naissance;
- le moi de naissance;
- le département de naissance;
- le numéro insee;
- l'identifiant d'enregistrement.

Etendez l'endpoint `[POST] patients` pour que celui-ci n'accepte que des patients né dans en Essonne (91).

# Exercice 3 - Livraison

Rédigez un docker compose permettant de lancer le tout.

# Exercice 4 - Changement de backend

Modifiez votre API pour que celle-ci fonctionne avec un stockage des données basé sur un fichier json et non plus une base de données. Pour vous aider, vous pourriez utiliser les appels suivant :

```python
import json

def read_json(json_path):
  with open(json_path, "r") as f:
    return json.loads(f.read())

def write_json(json_path, data):
  with open(json_path, "w") as f:
    json.dump(data, f, indent=4)
```

Modifiez votre docker compose pour pouvoir livrer votre nouvelle version

