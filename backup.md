Author: David Minh Vu, 5CHIT, 5.12.2018
# Backup
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
