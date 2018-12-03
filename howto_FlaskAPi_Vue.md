# Flask API und Vue Webpage mit Bootstrap
# "Restful User-Service" - Taskdescription

## Teil 1 - Einführung Flask
Diese Übung gibt eine Einführung in die Verwendung von Restful-Services.

## Ziele
Das Verständnis von zustandslosen Verbindungen um Daten leicht administrieren und verteilen zu können.

## Voraussetzungen
* Grundverständnis von Python
* Lesen und Umsetzen von APIs
* Erstellung von Netzwerkverbindungen
* Automatisiertes Testen mittels Unit- und System-Tests

## Detailierte Ausgabenbeschreibung
Implementieren Sie eine einfache Userdatenbank, welche ein Erstellen, das Ausgeben, Updaten und Löschen über eine ReST-Schnittstelle ermöglicht. Verwenden Sie dazu eine einfache JSON-Struktur (z.B.: {id:1; username:"mmatouschek"; email:"mmatouschek@student.tgm.ac.at"; picture:"..."}). Die einzelnen User sollen nach dem Beenden des Server-Dienstes persistent gespeichert werden.  

Überprüfen Sie die CRUD-Funktionen mittels Unit- und System-Tests.  


## Bewertung
Gruppengrösse: 1 Person
### Anforderungen **überwiegend erfüllt**
+ CRUD Befehle mittels ReST-Schnittstelle auf lokaler Netzwerkschnittstelle
+ Benutzerphoto per Link
+ Ausführung und Dokumentation der Implementierung

### Anforderungen **zur Gänze erfüllt**
+ Benutzerphoto als Base64-Encoding
+ persistente JSON-Datenbasis 

# Ausführung von Teil 1
Nimmt die Quelle 1 als Vorlage.
Hier wird eine REST API geschrieben. Diese soll dann im Teil 2 auf ein Webinterface abgebildet werden.

Man muss zuerst alle Module/Dependencies installieren. 
```
$ pip install flask
$ pip install flask_sqlalchemy 
$ pip install flask_marshmallow
$ pip install marshmallow-sqlalchemy
$ pip install flask_restful
$ pip install flask_cors
```
Und dann noch ein GIT reposisitory anlegen, dazu einfach im Root Ordner des Projekts
`git init`
fürs pushen
```
git add .
git commit -m "Init project"
git push origin master
```
Nun kann man mit der Arbeit beginnen. Man erstellt eine File "flask_server" und import dann alle Module.
```
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_marshmallow import Marshmallow
from flask_restful import reqparse, abort, Api, Resource
from flask_cors import CORS
import os
```
Flask erstellt eine Instanz der Webschnittstelle, request ist da um Daten anzufordern, und jsonify die Response Objekte in JSON Format umzuwandeln. SQAlchemy um auf die Datenbank zuzugreifen und Marshmallow um alles zu serializen. Cors ist dazu da um eine Schnittstelle zwischen der Vue Webpage (teil 2) und der API zu schaffen.
```
app = Flask(__name__)
CORS(app)
api = Api(app)
basedir = os.path.abspath(os.path.dirname(__file__))
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///' + os.path.join(basedir, 'crud.sqlite')
db = SQLAlchemy(app)
ma = Marshmallow(app)
```
SQlAlchemy und Marshmallow werden zur der Flask Applikation hinzugefügt. Der Rest oben ist nur um eine Instanze der Flask Webappilkation zu schaffen und eine URI zu erstellen.
```
class User(db.Model):
    """"
    Specifing the parameters a user has
    """
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120), unique=True)
    picture = db.Column(db.String(200), unique=False)

    def __init__(self, username, email, picture):
        """
        Init the data that is inputted
        ID isn't a parameter, as it is set by the system depending on how many users already exists
        :param username: Username
        :param email: Email of the User
        :param picture: Custom Picture of the User
        """
        self.username = username
        self.email = email
        self.picture = picture
```
Hier wird das Model "User" definiert und ihre Felder eingestellt. In der Init werden dann die Werte hinzugefügt die dann später vom User gesetzt werden.
```
class UserSchema(ma.Schema):
    """
    Fields to expose
    """
    class Meta:
        # Fields to expose
        fields = ('username', 'email', 'picture', 'id')


user_schema = UserSchema()
users_schema = UserSchema(many=True)
```
Hier wird die Struktur der Reponse definiert die dann auf die Webschnittstelle ausgelesen wird. Die JSON response sind dann eben alle Fields (username, email, ...)
```
class UserAdd(Resource):
    """
    User Add Method
    """
    def post(self):
        """
        This API is called by the a http POST request
        to add an user
        :return: the fields in a json format
        """
        username = request.json['username']
        email = request.json['email']
        picture = request.json['picture']

        new_user = User(username, email, picture)

        db.session.add(new_user)
        db.session.commit()

        return user_schema.jsonify(new_user)
```
Nun kommen die richtige implementation um einen neuen User in die Datenbank anzulegen. Wir legen dazu nur die Methode an, aber die Route wie "/user" wird erst am Ende implementiert. Es wird die Klasse UserAdd aufgerufen und es hat eine post Methode. Am Endpoint wird mittels der HTTP Methode POST eine anfrage geschickt und die def post(self) wird dabei aufgerufen. Zuerst werden die Daten requestet. Danach wird ein neuer User mit den Daten vom request erstellt. Diese wird der Datenbank hinzugefügt über .add und .commit.
Als response wird dann der neue User als JSON response angezeigt.
# Quellen
1. https://medium.com/python-pandemonium/build-simple-restful-api-with-python-and-flask-part-2-724ebf04d12
2. [Flask ReST](https://flask-restful.readthedocs.io/en/latest/quickstart.html#full-example)
3. [Sqlite with Python](https://docs.python.org/3/library/sqlite3.html)
