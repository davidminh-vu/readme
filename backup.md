Author: David Minh Vu, 5CHIT, 5.12.2018
# Backup
# Schritt 1
Backups sind wichtig, Dateiverlust kann aufgrund Hardwaredefekts zustande kommen. Es ist deshalb sinnvolle Backup Strategien anzuwenden. In diesem Dokument werden folgende Strategien besprochen:
1. Inkrementelles Backup
2. Differentielles Backup
3. Vollbackup

## Inkrementelles Backup
Zuerst wird eine Vollsicher des vollen Datenbestandes gemacht. Die darauffolgenden Backups bauen auf den vorherigen Backup auf und ergänzen nur neu Daten zu einem neuen Band hinzu. Bedeutet wenn man nicht alle Bänder besitzt, ist die Wiederherstellung der gesicherten Daten nicht möglich.

### Vorteile
Dies ist ein einfaches Verfahren und benötigt nicht viel Speicherbedarf, da die inkrementellen Backups meist kleiner sind.
Man kann die Daten zu jedem Backupzeitpunkt wiederherstellen.

### Nachteile
Es sind das Vollkabup und alle darauf folgenden Backupbänder benötigt um eine Wiederherstellung möglich zu machen.

## Differentielles Backup
Prinzipell ist das differentielle Backup gleich dem Inkrementellen. Es wird eine Vollsicherung gemacht und danach werden alle Veränderungen zum letzten Vollbackup gesichert. Daten die einmal verändert werden, müssen bei jedem differentiellen Backup neu gesichert werden. Demnach ist zur Wiederherstellung des Datenbestandes das Vollbackup und das gewünchste differentielle Backup notwendig.

### Vorteile
Mehr Speicherbedarf als beim inkrementellen Backup, aber weniger als beim Vollbackup
Nur Vollbackup und die differentielle Sicherung zum gewünschten Zeitpunkt notwendig.

### Nachteile
Dateien, die einmal verändert werden, müssen bei jedem differentiellen Backup neu gesichert werden. Dadurch hat man ein erhöhtes Datenaufkommen.

## Vollbackup
Dies sichert immer bei jeder Iteration den kompletten Datenbestand. Es wird deshalb nur das entsprechende Vollbackupmedium benötigt.

### Vorteile
Nur ein Band zur Wiederherstellung notwenidg
Einfache Wiederherstellung von Daten

### Nachteile
Hoher Speicherbedarf
Um  mehrere Versionen zu haben, müssen mehrere Sicherungsbänder aufbewahrt werden.

## Optimale Backup-Methode
Meist werden aus allen 3 Backup-Methoden eine Kombination gemacht. Dies kommt auf den Anwendungsfall an, aber ein Beispiel wäre wöchentlich ein Vollbackup zu erstellen und jeden Tag ein inkrementelles Backup um das Datenaufkommen niedrig zu halten.

# Backup and Restore in PostgreSQL
# Schritt 2
## SQL Dump | Vollbackup
In PostgreSQL kann man auch die Backupstrategien einsetzen. PostgreSQL erlaubt es alle Befehle die benutzt wurden um die Datenbank zu erstellen in eine textfile zu schreiben und diese kann später dann benutzt werden um die Datenbank zu dem Zeitpunkt der erstellung des textfile herzustellen. Dies ist das Vollbackup und verbraucht am meisten Speichern von all den Backup-Methoden.

Der Befehl `pg_dumb dbname > out_textfile` wird in PostgreSQl benutzt um das Verfahren einzuleiten. Hier wird die DB "dbname" in eine File "out_textfile" geschrieben. Wichtig hierbei ist, dass nur die Tabelen in das Textfile geschrieben werden zu denen der derzeitige User "Read" Berechtigungen besitzt. Deshalb erfolgt dies meist mit einem Superuser. 

Um dieses Textfile wieder in eine Datenbank umzuwandeln muss man den folgenden Befehl ausführen: `psql dbname < out_textfile`. Hier wird das Textfile in eine Datenbank "dbname" eingelesen. Wichtig hierbei ist, dass die Datenbank bereits davor exisitieren muss, da der Befehl die Datenbank nicht erstellt sonder nur in die Datenbank einliest. 
Auch ist es wichtig, dass bevor man den Befehl ausführt alle Administratoren/User die davor in der Datenbank irgendwelche Berechtigungen besitzt haben, auch in der Datenbankumgebung exisiteren, ansonsten werden die Objecte nicht mit den orignialen Ownerships/permission erstellt.

## File System Level Backup
Eine Alternative zu SQL Dump ist es nur die Daten, die PostgreSQL benutzt um alles abzuspeicher, zu kopieren. Doch diese Methode ist umpraktisch da der Server heruntergefahren sein muss damit man die Datenbank kopieren kann. Und falls die Datenbank über mehrere Dateisysteme verteilt ist, ist es schwer alle Backups zu bekommen.

## Continuous Archiving
Das PostgreSQL-System schreibt zu allen zeiten eine sogennantes "Write ahead Log (WAL)" in dem alle Veränderungen bei der Datenbanken mitgeschrieben werden. Im falle eines Absturz kann die Datenbank wiederhergestellt werden. Somit kann auch eine Backupstrategie daraus erstellt werden. Man kann das File System Level Backup mit dem Backup der WAL-FIles kombinieren. Dies ist weitaus komplexer aber bietet Vorteile.

- Die Datenbank muss nicht vollkommen Konsitent sein, die Fehler die während des File System Level Backup entstehen werden durch das WAL Wile behoben.
- Man kann mehrere WAL-Files unendlichlang miteinander anzureihen. Dies ermöglicht es fortlaufend backups zu machen die dann aneinander gereiht werden können. Bei großen Datenbaken ist dies sehr wichtig, da nicht soviel Speicher verwendet wird als beim Vollbackup.
- Zudem kann auch nur bestimmte WAL-Files einanderreihen und zu einem bestimmten Zeitpunkt der Backups wieder herstellen ohne immer alle ausführen zu müssen.

Zu beachten ist, dass pg_dumb in diesem Fall nicht genügend Daten besitzt ist um so eine Kombination mit WAL-files zu unterstützen.
Auch ist ein Nachteil dass man nur die komplette Datenbank kopieren kann und nicht nur ein Teil der Datenbank.

# Beispiel
