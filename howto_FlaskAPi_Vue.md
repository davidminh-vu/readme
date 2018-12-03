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

#Ausführung von Teil 1
Nimmt die Quelle 1 als Vorlage.
Hier wird eine REST API geschrieben. Diese soll dann im Teil 2 auf ein Webinterface abgebildet werden.

Man muss zuerst alle Module/Dependencies installieren. 
`
$ pip install flask_sqlalchemy
$ pip install flask_marshmallow
$ pip install marshmallow-sqlalchemy
`

# Quellen
1. https://medium.com/python-pandemonium/build-simple-restful-api-with-python-and-flask-part-2-724ebf04d12
2. [Flask ReST](https://flask-restful.readthedocs.io/en/latest/quickstart.html#full-example)
3. [Sqlite with Python](https://docs.python.org/3/library/sqlite3.html)
