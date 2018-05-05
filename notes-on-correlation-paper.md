#Quelle- Notizen zum Seminar SoSe 2018

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
    ** Für zentralisierte INfrastruktur ist eine Sendung mittels TCP/TLS besser

Verbesserungen bringt RFC **RFC 3164**





 


