## Correlation and consolidation of Distributed Logging Data in Enterprise Clouds


### Abstract

AUfgrund von aktuellen Entwicklungen wie CLoud-Computing und der steigenden Anzahl
von Anwendungen und Systemen ssteigt auch die Menge an Logdateien rapide an.

Die Auswertung kann nur mittels automatisierten Korrelations und Aggregations-Verfahren
geschehen, Die Fülle an Logdateien kann nicht dauerhaft gespeichert werden und nur
relevante Daten sollen ausgewertet werden.

IN diesem Paper wird ein Prototyp vorgestellt unter Verwendung eines NoSQL Storage-Backe-Ends.
Die Konsolidierung von Daten impliziert auch strategien zur minimierung von Langzeitspeicherung
und KOrreltationstechniken um Anomalien (SECURITY!) zu erkennen.

Es wird die speziellen ANforderungen presentiert um mit dieses hochskalierbaren Logsystem
umzugehen, eben in einer hochverfügbaren Umgebnug mit dynamischer Infrastruktur.

# Introduction

* schnelle entwicklung von Virtualisierungsumgebiungen
* speziell Cloud-COmputing: schnelle Bereitstellung von SaaS, Iaas ... 
* Anfallende Logdaten werden zum Mnitoring der Systeme benötigt
** Monitoring: (Status des Systems, Dinestgüte (servicequality)) 
* Jede neue Quelle (System, Service, INfrastruktur) generiert neue Logs
** alte Logs können verworfen werden, Cloud 

... (nachmal lesen, sehr kompakt gefasst)

## II State-of-the-art (evolution of logging methods in distributet envirinments)

unter der nutzung von standardtechniken zur Kosnolidierung und Aggregation von Loggdateien
mithilfe von Standardtools (like syslog).

Derzeit sind viele Monitoringmechanismen verbereitet:

AWS and Rackspace: Mnoitoring mehrerer Metriken (CPU LOAD, Network-Load ..) auch mittels
API, aber keine Möglichkeiten zur Aggregation und Korrelation von Daten die direkt vom 
OS und den ANwendungen stammen.

Auch nutzen alle Cloud-Provider unterschiedliche APIS und Monitoringtechniken:
DIe IETF arbeitet allerdings an einer Lösung: [3]
Grundlage von [3] ist **Syslog**, was auch die de-facto standard in unixoiden Umgebungen ist.
Zusätzlich ist kann eine strukturierte Speichjerung in NoSQL Datenbanken erfolgen.

## Logdaten Aggreagation und KOnsolidierung mit syslog

* de-facto standart
* keine Standardisierung des Protokolls
* [RFC 3164] BSD-Syslog Protokoll: Standardiesierung des Log-Formats
* verteilte Struktur
* Trennung von Anwendungen, System und Netzwerkinfrastruktur möglich

### Aufbau Syslog nach RFC 3164

Jedes Paket startet mit **PRI** (Priority): Gewichtung und Facility gehen daraus hervor
(als eine Dezimalnummer zusammen kodiert)
<DD>
Es werden 8 Gewichtungen spezifiziert: 
24 facility-Werte werden sppezifiziert: Diese werden den Daemons des Systems vorgenommen.

Danach folgt der Teil **HEADER**: TIMESTAMP HOSTNAME (getrennt durch Leerzeichen)
TIMESTAMP: Mmm dd hh:mm:ss (lokale Zeit bei Generierung der Nachricht)
HOSTNAME: hostname, IPv4 oder IPv6

Danach folgt der **MSG** Teil:
TAG: Programm welches die Nachricht generiert hat. (kann variieren zb PRG/PID)
CONTENT: Die tatsächliche LOG-Nachricht

* Speicherung in Logdateien für verteilte Systeme -> zentrale Ablage von Logs
* BSD-Style Syslog verwendet UDP
** Für zentralisierte Infrastruktur ist eine Sendung mittels TCP/TLS besser

Verbesserungen bringt RFC **RFC 5424**:
* **PRI** ist genauso kodiert, aber nun Teil des **HEADER**. Zudem zusätzlich:
* **PID**
* **MESSAGE-ID**
* **TIMESTAMP** nach RFC 3339
* Und das wichtigste: **Strukturierte Daten**

**EXAMPLE**

    <165>1 2003-10-11T22:14:15.003Z mymachine.example.com
           evntslog - ID47 [exampleSDID@32473 iut="3" eventSource=
           "Application" eventID="1011"] BOMAn application
           event log entry...

Software mit RFC 5425 unterstützung: *syslog-ng* und *rsyslog*

**Das Problem ist allerdings:**
* Die extreme Menge und Größe der anfallenden Log-Daten
* Einführung neuer Strukturen (vgl. RFC 5424)

--> Anmerkung: HIer noch auf alternativen eingehen:
* zentrale Logserver mit Monitoring
** Menge an Daten kann unnutzbar werden

* relationale Datenbanken (SQL-Only)
** Struktur der Daten muss zur Erstellung der Datenbank bekannt sein

Das einzige System was diesen Anforderungen gerecht wird: NoSQL
* Speicherung von enormen Mengen an strukturierten Daten (KEY->VAL)
* Strukturierung der Daten kann variieren

**NoSQL:**
Im Gegensatz zu SQL Datenbanken, wird bei NoSQL-Datenbanken kein Schema oder ein
minimales Schema angelegt, das erlaubt die Änderunge der Structur zu jeder Zeit.
Ergo, perfekt um Log-Daten für eine gewisse Zeit zu speichern.
Die Daten sind wohlformatiert abgelegt und die KEY->VAL Paare unterscheiden
sich lediglich nach LOG-Quelle.
Zudem lassen sich NoSQL-Lösungen über hunderte von Systemen redundant skalieren.
Da bei NoSQL allerdings die performance und Ausfallsicherheit im Vordergrund steht,
ist nicht sichergestellt, das bei jeder NoSQL-Instanz die aktuellsten Daten vorhanden
sind.

Es gibt eine VIelzahl an Implementierungen von NoSQL, alle mit speziellen Zielen:

Es lassen sich drei Typen von NoSQL-IMplekentierungen identifizieren:

1. **Key-Value**:
** Speicherung von untrukturierten Daten
** Daten werden nicht interpretiert
** Hochperformant
** einfache API (insert, delete, lookup)
** Keine Suche in BLOBS
2. **extensible record stores/column-oriented datastores**
** Speichert verwandte Daten in ein Zeilen und Spalten - Orientiertem System ab











 


