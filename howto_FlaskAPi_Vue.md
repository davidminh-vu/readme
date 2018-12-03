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
Nun kann man mit der Arbeit beginnen. Man muss zuerst den SQLite Server erstellen. Man erstellt hierzu eine neue "flask_createserver.py" und fügt folgenden Code ein
```
from server.flask_server import db
#Creates a new Flask Database
db.create_all()
```
Dies erstellt einfach nur eine neue Datenbank. Wenn man möchte kann man dies in eine in eine Methode hinzufügen und dann es mittels der main aufrufen, doch dies ist hier optional.

Man erstellt eine File "flask_server.py" und import dann alle Module.
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
```
class getUsers(Resource):
    def get(self):
        """
        This API is called by the a http GET request
        to get all Users in the Database
        :return: all users with their data in a json format
        """
        all_users = User.query.all()
        result = users_schema.dump(all_users)
        return jsonify(result.data)
        
class getUser(Resource):
    def get(self, id):
        """
        This API is called by the a http GET request
        to get one user specified by the ID
        :param id: ID of the User
        :return: the fields of the user in a json format
        """
        user = User.query.get(id)
        return user_schema.jsonify(user)

```
Bei getUsers() werden alle Datensätze die in der Datenbank orhanden sind ausgelesen und als JSON zurückgegeben. 
Bei getUser() wird genau der User ausgelesen der die bestimmte ID besitzt. Diese wird später z.B. über "user/1" aufgerufen.

Der Rest wie Put (Updatet die Fields eines User) und delete geht nach den genau den gleichen Schema.
Das nächste wichtige ist wie man die Methoden als Endpoint bzw. als URI in die Webapplikation einbaut.
```
#Adds an Subsite with the method that is called
#Basically the API
api.add_resource(UserAdd, '/user')
api.add_resource(getUser, '/user/<string:id>')
api.add_resource(getUsers, '/user/all')
api.add_resource(userDelete, '/user/<string:id>/delete')
api.add_resource(userUpdate, '/user/<string:id>/put')
api.add_resource(deleteAllUser, '/user/all/delete')
```
Dies ist selbsterklärend, zuerst die Methode dann die URI auf der es laufen soll. <string:id> bekommt man z.B. bei "user/1" 
Nun der ganze Sourcecode:
```
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_marshmallow import Marshmallow
from flask_restful import reqparse, abort, Api, Resource
from flask_cors import CORS
import os

app = Flask(__name__)
CORS(app)
api = Api(app)
basedir = os.path.abspath(os.path.dirname(__file__))
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///' + os.path.join(basedir, 'crud.sqlite')
db = SQLAlchemy(app)
ma = Marshmallow(app)

parser = reqparse.RequestParser()
parser.add_argument('user')

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


class UserSchema(ma.Schema):
    """
    Fields to expose
    """
    class Meta:
        # Fields to expose
        fields = ('username', 'email', 'picture', 'id')


user_schema = UserSchema()
users_schema = UserSchema(many=True)


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

class getUsers(Resource):
    def get(self):
        """
        This API is called by the a http GET request
        to get all Users in the Database
        :return: all users with their data in a json format
        """
        all_users = User.query.all()
        result = users_schema.dump(all_users)
        return jsonify(result.data)

class userDelete(Resource):
    def delete(self, id):
        """
        This API is called by the a http DELETE request
        to delete a user specified by the ID
        :param id: ID of the User
        :return: the deleted user data in a json format
        """
        user = User.query.get(id)
        db.session.delete(user)
        db.session.commit()

        return user_schema.jsonify(user)

class getUser(Resource):
    def get(self, id):
        """
        This API is called by the a http GET request
        to get one user specified by the ID
        :param id: ID of the User
        :return: the fields of the user in a json format
        """
        user = User.query.get(id)
        return user_schema.jsonify(user)

class userUpdate(Resource):
    def put(self, id):
        """
        This API is called by the a http PUT request
        to update the fields of an user
        :param id: ID of the User
        :return: the updated fields in a json format
        """
        user = User.query.get(id)
        username = request.json['username']
        email = request.json['email']
        picture = request.json['picture']

        user.email = email
        user.username = username
        user.picture = picture

        db.session.commit()
        return user_schema.jsonify(user)

class deleteAllUser(Resource):
    def delete(self):
        """
        This API is called by the a http DELETE request
        to delete every User in the DB
        :return: the updated fields in a json format
        """
        all_users = User.query.all()
        for i in all_users:
            db.session.delete(i)
            db.session.commit()
        return 201

#Adds an Subsite with the method that is called
#Basically the API
api.add_resource(UserAdd, '/user')
api.add_resource(getUser, '/user/<string:id>')
api.add_resource(getUsers, '/user/all')
api.add_resource(userDelete, '/user/<string:id>/delete')
api.add_resource(userUpdate, '/user/<string:id>/put')
api.add_resource(deleteAllUser, '/user/all/delete')
if __name__ == '__main__':
    app.run(debug=True)

```

Um die Flask Applikation zu starten muss man die flask_server.py via Console starten.
`python flask_server.py`
Nun kann man seine Methoden testen. Dazu wird API development Tool benötigt wie Insomnia. Man fügt die URL ein ("localhost:5000/user") und stellt, je nach ausgewählter Methode, das richtige HTTP Protokoll (POST, GET, DELETE, etc...) ein. Wenn man z.B. ein neuen User hinzufügen will muss man noch einen JSON Datensatz in das Package schreiben.

## Testen der Flask Applikation
Hier wird pytest benutzt um Flask zu testen. Dies passiert quasi wie jeder anderer Test.
```
import pytest
import os
from server.flask_server import app, db

userdata = {"username":"david", "email":"test1@stueden3t.tg3m.ac.at", "picture":"https://www.gooegle.com/pnglol"}
userdata2 = {"username":"david2", "email":"mvu2@stuedent.tgm.ac.at", "picture":"https://www.gooegle.com/png2"}
```
Man importet pytest, os und vom flask_server die App und die DB um darauf zugreifen zu können. Danach sind zwei Datensätze definiert die in die Datenbank hinzugefügt werden. 
```
@pytest.fixture
def client():
    """
    Something like StartUp
    Can specify objects that are needed in every test
    """
    app.config['TESTING'] = True
    db.create_all()
    client = app.test_client()
    yield client
    db.drop_all()
```
Pytest.fixture ist quasi nur das StartUp, man kann Objekte definieren die in jedem Test dann benutzt werden können.
Hier werden nur die config files eingestellt, die Datenbank erstellt bei yield auf die Testergebnisse gewartet und dann die B wieder gedroppt.
```
def test_addUser(client):
    """
    Test if the AddUser Method is working and checks if the data exists in the DB
    :param client:
    :return:
    """
    response = client.post("http://localhost:5000/user", json = userdata)
    answ = client.get("http://localhost:5000/user/1")
    assert (response.json['username'] == "david") and (response.json['email'] == "test1@stueden3t.tg3m.ac.at") and (response.json['picture'] == "https://www.gooegle.com/pnglol")

def test_getUsers(client):
    """
    Test the GetUsers Method
    Should get every User in the Database
    :param client:
    :return:
    """
    response = client.post("http://localhost:5000/user", json = userdata)
    response2 = client.post("http://localhost:5000/user", json = userdata2)
    answ = client.get("http://localhost:5000/user/all")
    assert (response.json['username'] == "david") and (response.json['email'] == "test1@stueden3t.tg3m.ac.at") and (response.json['picture'] == "https://www.gooegle.com/pnglol"
                        and response2.json['username'] == "david2") and (response2.json['email'] == "mvu2@stuedent.tgm.ac.at") and (response2.json['picture'] == "https://www.gooegle.com/png2")

def test_getUser(client):
    """
    Test the GetUser Method
    Should get one User with a specific ID
    :param client:
    :return:
    """
    response2 = client.post("http://localhost:5000/user", json = userdata2)
    answ = client.get("http://localhost:5000/user/1")
    assert (answ.json['username'] == "david2") and (answ.json['email'] == "mvu2@stuedent.tgm.ac.at") and (answ.json['picture'] == "https://www.gooegle.com/png2")

def test_updateUser(client):
    """
    Test the UpdateUser Method
    Should update a exisiting Users fields
    to a different set of data
    :param client:
    :return:
    """
    response = client.post("http://localhost:5000/user", json = userdata)
    update = client.put("http://localhost:5000/user/1/put", json = userdata2)
    getUser = client.get("http://localhost:5000/user/1")
    assert (getUser.json['username'] == "david2") and (getUser.json['email'] == "mvu2@stuedent.tgm.ac.at") and (getUser.json['picture'] == "https://www.gooegle.com/png2")

def test_deleteUser(client):
    """
    Test the DeleteUser Method
    Should delete an exisitng user with a specific id
    :param client:
    :return:
    """
    response = client.post("http://localhost:5000/user", json = userdata)
    response2 = client.post("http://localhost:5000/user", json = userdata2)
    deleteUser = client.delete("http://localhost:5000/user/1/delete")
    getUser = client.get("http://localhost:5000/user/1")
    assert len(getUser.json) == 0

def test_deleteAllUser(client):
    """
    Test the DeleteUsers Method
    Should delete all existing users in the Database
    :param client:
    :return:
    """
    response = client.post("http://localhost:5000/user", json = userdata)
    response2 = client.post("http://localhost:5000/user", json = userdata2)
    deleteAll = client.delete("http://localhost:5000/user/all/delete")
    getAll = client.get("http://localhost:5000/user/all")
    assert len(getAll.json) == 0
```
Um irgendwas zu testen ruft man nur die URI Schnittstellen auf und testet dich sachen. Die Datensätze die man zurückbekommt sind im JSON Format. Hier ist noch zu beachten dass man client.[HTTP_Methode] aufrufen muss.

Man geht hierzu zu seinem Ornder wo man das Installieren will und führt folgende Befehle aus:
```
$ npm install -g vue-cli
$ vue init webpack my-project
$ cd my-project
$ npm install
$ npm run
```
Zuerst wird vue cli installiert und dann im ordner initalisiert. Danach kommt ein npm install um alle benötigten Dependencies zu installieren. npm run startet den Webserver.

Einfaches Template fürs Verständnis:
```
template>
  <div>
    <p>{{ msg }}</p>
  </div>
</template>

<script>
export default {
  name: 'Ping',
  data() {
    return {
      msg: 'Hello!',
    };
  },
};
</script>
```
Das Template ist sozusagen nur das HTML Tag nur anders. Danach wird ist nur Standard HTML bis zum p-tag. da kommt {{ msg }}, this wird im folgenden script-tag erst definiert. Dies ist nur eine Variable die man dan später setzen wird.
Bei dem data(){...} setzt man nun seine Parameter. Hier wird msg als "Hello!" definiert, hier könnte man auch Arrays setzen oder ähnliches, wichtig hierbei ist immer ein ',' zu setzen wenn danach noch ein Datensatz folgt.


Um zu startet erstellt man in my-project/src/components eine neue "User.vue" Datein mit dem Template von https://github.com/testdrivenio/flask-vue-crud in client/src/components books.vue. Bei diesem muss man nur die Datensätze umändern.
```
<template>
   <div class="container">
    <div class="row">
      <div class="col-sm-10">
        <h1>Users</h1>
        <hr><br><br>
        <button type="button" id="addUserButton" class="btn btn-success btn-sm" v-b-modal.user-modal>Add User</button>
        <br><br>

        <!-- user table -->
        <table class="table table-hover" id="tableList">
          <thead>
            <tr>
              <th scope="col">Username</th>
              <th scope="col">Email</th>
              <th scope="col">Picture URL</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(user, index) in users" :key="index">
              <td>{{ user.username }}</td>
              <td>{{ user.email }}</td>
              <td>{{ user.picture }}</td>
              <td>
                <button type="button"
                        class="btn btn-warning btn-sm"
                        id="updateUserButton"
                        v-b-modal.user-update-modal
                        @click="editUser(user)">
                    Update
                </button>
                <button type="button"
                        class="btn btn-danger btn-sm"
                        id="deleteUserButton"
                        @click="onDeleteUser(user)">
                    Delete
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <!-- add User modal -->
    <b-modal ref="addUserModal"
             id="user-modal"
            title="Add a new user"
            hide-footer>
      <b-form @submit="onSubmit" @reset="onReset" class="w-100">
        <b-form-group id="form-username-group"
                      label="Username:"
                      label-for="form-username-input">
            <b-form-input id="form-username-input"
                          type="text"
                          v-model="addUserForm.username"
                          required
                          placeholder="Enter username">
            </b-form-input>
        </b-form-group>
        <b-form-group id="form-email-group"
                      label="Email:"
                      label-for="form-email-input">
          <b-form-input id="form-email-input"
                        type="email"
                        v-model="addUserForm.email"
                        required
                        placeholder="Enter email">
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-picture-group"
                      label="Picture (URL):"
                      label-for="form-picture-input">
          <b-form-input id="form-picture-input"
                        type="text"
                        v-model="addUserForm.picture"
                        required
                        placeholder="Enter picture URL">
          </b-form-input>
        </b-form-group>
        <b-button type="submit" id="submitNewUser" variant="primary">Submit</b-button>
        <b-button type="reset" id="resetNewUser" variant="danger">Reset</b-button>
      </b-form>
    </b-modal>

    <b-modal ref="editUserModal"
             id="user-update-modal"
             title="Update"
             hide-footer>
      <b-form @submit="onSubmitUpdate" @reset="onResetUpdate" class="w-100">
        <b-form-group id="form-username-edit-group"
                      label="Title:"
                      label-for="form-username-edit-input">
          <b-form-input id="form-username-edit-input"
                        type="text"
                        v-model="editForm.username"
                        required
                        placeholder="Enter username">
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-email-edit-group"
                      label="Email:"
                      label-for="form-email-edit-input">
          <b-form-input id="form-email-edit-input"
                        type="email"
                        v-model="editForm.email"
                        required
                        placeholder="Enter email">
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-picture-edit-group"
                      label="Picture (URL):"
                      label-for="form-picture-edit-input">
          <b-form-input id="form-picture-edit-input"
                        type="text"
                        v-model="editForm.picture"
                        required
                        placeholder="Enter picture URL">
          </b-form-input>
        </b-form-group>
        <b-button type="submit" id="submitEdit" variant="primary">Update</b-button>
        <b-button type="reset" id="resetEdit" variant="danger">Cancel</b-button>
      </b-form>
    </b-modal>
  </div>
</template>
```
Dies ist nur eine Bootstrap Seite. Wichtig hierbei ist das setzen von dem v-model. Dies ist nur ein custom input von vue, um diese dann später im script leichter setzten zu können. Bedeutet man hat einen Input für den usernamen mit v-model="addUserForm.username", nun kann man im Skript in der Data(){return{addUserForm: { username: ''}}}. Man kann nun den Value aus dem Input field ganz einfach über this.addUserForm.username bekommen. <b-form @submit="onSubmit" @reset="onReset" class="w-100"> @submit ruft nur die Methode onSubmit auf.
```
<script>
import axios from 'axios'
export default {
  name: 'User',
  data () {
    return {
      msg: null,
      showMessage: false,
      users: [],
      addUserForm: {
        username: '',
        email: '',
        picture: ''
      },
      editForm: {
        username: '',
        email: '',
        picture: ''
      }
    }
  },
  methods: {
    getUsers () {
      // Calls the API
      // Gets all Users existing in the DB
      // and is then updated in the actual vue html
      const path = 'http://localhost:5000/user/all'
      axios.get(path)
        .then((res) => {
          this.users = res.data
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error)
        })
    },
    addUser (payload) {
      // Calls the API
      // Adds an User to the DB
      // and then calls getUsers to update the HTML
      const path = 'http://localhost:5000/user'
      axios.post(path, payload)
        .then(() => {
          this.getUsers()
          this.message = 'User added!'
          this.showMessage = true
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error)
          this.getUsers()
        })
    },
    updateUser (payload, userID) {
      // Calls the API
      // Updates the fields of an User
      // and then calls getUsers to update the HTML
      const path = `http://localhost:5000/user/${userID}/put`
      axios.put(path, payload)
        .then(() => {
          this.getUsers()
          this.message = 'User updated!'
          this.showMessage = true
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error)
          this.getUsers()
        })
    },
    removeUser (userID) {
      // Calls the API
      // Removes an User when clicking on the delete button
      // and then calls getUsers to update the HTML
      const path = `http://localhost:5000/user/${userID}/delete`
      axios.delete(path)
        .then(() => {
          this.getUsers()
          this.message = 'User removed!'
          this.showMessage = true
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error)
          this.getUsers()
        })
    },
    initForm () {
      // Sets the value of the Forms
      // to empty Strings
      // when adding a new user
      this.addUserForm.username = ''
      this.addUserForm.email = ''
      this.addUserForm.picture = ''
      this.editForm.username = ''
      this.editForm.email = ''
      this.editForm.picture = ''
      this.editForm.id = ''
    },
    onSubmit (evt) {
      // When user pressed the submit button
      // when adding a new User
      // Calls the addUser Method with the Data as Parameter
      evt.preventDefault()
      this.$refs.addUserModal.hide()
      const payload = {
        username: this.addUserForm.username,
        email: this.addUserForm.email,
        picture: this.addUserForm.picture
      }
      this.addUser(payload)
      this.initForm()
    },
    onSubmitUpdate (evt) {
      // When user pressed the submit button
      // when updating the fields of a new User
      // Calls the updateUser Method with the Data as Parameter
      evt.preventDefault()
      this.$refs.editUserModal.hide()
      const payload = {
        username: this.editForm.username,
        email: this.editForm.email,
        picture: this.editForm.picture
      }
      this.updateUser(payload, this.editForm.id)
    },
    onReset (evt) {
      // When user pressed the reset button
      // when adding a new User
      // Clears the fields of the form
      evt.preventDefault()
      this.$refs.addUserModal.hide()
      this.initForm()
    },
    onResetUpdate (evt) {
      // When user pressed the reset button
      // when updating a new User
      // Clears the fields of the form
      evt.preventDefault()
      this.$refs.editUserModal.hide()
      this.initForm()
      this.getUsers()
    },
    onDeleteUser (user) {
      // When user pressed the remove button
      // Calls the removeUser Method with the ID as the parameter
      this.removeUser(user.id)
    },
    editUser (user) {
      this.editForm = user
      console.log(this.editForm)
    }
  },
  created () {
    // When the site is first created
    this.getUsers()
  }
}
</script>

<style scoped>

</style>
```
Created ist aufgerufen wenn die Seite erstellt wird, Payload sind nur die benötigten Datensätze die man von den input Feldern holt.
Der Rest wird über die Buttons aufgerufen.

Nun muss man in seiner my-project/src/router seine index.js updaten um seine User.vue zu referenzieren.
```
import Vue from 'vue'
import Router from 'vue-router'
import User from '@/components/User'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'User',
      component: User
    }
  ]
})
```
Zusätzlich fehlt noch der Bootstrap verweiß in der main.js
```
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import 'bootstrap/dist/css/bootstrap.css'
import BootstrapVue from 'bootstrap-vue'
import Vue from 'vue'
import App from './App'
import router from './router'

Vue.config.productionTip = false

Vue.use(BootstrapVue)

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})
```
# Quellen
1. https://medium.com/python-pandemonium/build-simple-restful-api-with-python-and-flask-part-2-724ebf04d12
2. [Flask ReST](https://flask-restful.readthedocs.io/en/latest/quickstart.html#full-example)
3. [Sqlite with Python](https://docs.python.org/3/library/sqlite3.html)
