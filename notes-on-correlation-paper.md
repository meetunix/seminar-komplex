# Konspekt zum paper:
# Correlation and consolidation of Distributed Logging Data in Enterprise Clouds


## Abstract

AUfgrund von aktuellen Entwicklungen wie CLoud-Computing und der steigenden Anzahl
von Anwendungen und Systemen ssteigt auch die Menge an Logdateien rapide an.

Die Auswertung kann nur mittels automatisierten Korrelations und Aggregations-Verfahren
geschehen, Die Fülle an Logdateien kann nicht dauerhaft gespeichert werden und nur
relevante Daten sollen ausgewertet werden.

IN diesem Paper wird ein Prototyp vorgestellt unter Verwendung eines NoSQL Storage-Backe-Ends.
Die Konsolidierung von Daten impliziert auch strategien zur minimierung von Langzeitspeicherung
und Korreltationstechniken um Anomalien (SECURITY!) zu erkennen.

Es wird die speziellen ANforderungen presentiert um mit dieses hochskalierbaren Logsystem
umzugehen, eben in einer hochverfügbaren Umgebnug mit dynamischer Infrastruktur.

## Introduction

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

### Logdaten Aggreagation und KOnsolidierung mit syslog

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
sind (ACID) - .

Es gibt eine VIelzahl an Implementierungen von NoSQL, alle mit speziellen Zielen:

Es lassen sich drei Typen von NoSQL-Implekentierungen identifizieren:

1. **Key-Value**:
* Speicherung von untrukturierten Daten
* Daten werden nicht interpretiert
* Hochperformant
* einfache API (insert, delete, lookup)
* Keine Suche in BLOBS
* SKalierung sehr einfach möglich

2. **extensible record stores/column-oriented datastores**
* Speichert verwandte Daten in ein Zeilen und Spalten - orientiertem System ab
* Skalierbarkeit wird erreicht, durch
** Durch das Teilen von Zeilen in *shards* durch ihren PK
** Durch das Teilen von Spalten in Spaltengruppen
* Dadurch kann das "schema" beliebig erweitert werden

3. **document-based data stores**
* document: Eine Menge an Objekten, wobei jedes unterschiedliche Anzahl an Attributen haben darf
* Neue Objecte dürfen jeder Zeit angelegt werden und sich in Anzahl und Typ ihrer
KEY-VAL paare unterscheiden.
* Durch Teilen von Objekten auch hier Skalierung möglich
* Volltextsuche

Durch zusätzliche Anwendung von Korrelierungstechniken könnnen Syslogdaten
bis zu 99% reduziert werden [23]. Es werden nur die Daten dauerhaft aufgehoben, die
relevant für auswertungen sind.

## IV Anforderungen für die Korrelation von Logdaten

In den folgenden Anschnitt wird eine Vorgehensweise beschrieben um die Evaluierung
der Logmedlung-Relevanz zu automatisieren und die Logdatenmenge zu Reduzieren.

Zudem wird eine Methode vorgestellt, um wichtige Eventtypen zu identifizieren um
eine Korrellation und KOnsoliedeirung durchzuführen.

### A. Correlating Ditributed Logs in Enterprose Clouds

Am Beispiel einer Brute-Force Attacke auf einen SSH-Server:
Unter Millionen von Versuchen sollen die herausgefunden werden, die zum erfolgreichen
Login führen.

Zwischen all dem Noise, soll sozusagen die Stecknadel im Heuhaufen gefinden werden, denn
daran ist der Sysadmin interessiert.

### B. Consolidating Logging Data from Distributed Services

Das zweite große Ziel ist die Nummer an Nachtrichten zu redizueren, welche
dauerhaft in Elasticsearch vorgehalten werden. Dies führt zu einer schnelleren
Verarbeitung und Anayse der Daten. Allerdings gehen dadurch Kontextinformationen
verloren, daher sind einfache Konsolidierungmechanismen nicht nützlich (?)

Das Ziel ist daher identische oder Serien von Events zu gruppieren und daraus
neue Lognacrichten zu generieren. Die neuen Syslog-Meldungen beinhalten eine
kompakte Repräsentation der Daten. Die initialen Daten können so verworfen werden.

Am Beispiel SSH-Brute-Force:

* Millionen von security-event Messages werden zu einer hochpriorisierten Nachricht
** Sysadmin weiß über das Angriff bescheid

Beispiel HTTP-Loadbalancer

* viele http-server liefern Logs an zentralen Log-Server aus
    * Einzelne Logs müssen Aggregiert werden -> eine neue Log-Nachricht
        * z.B. access count per Timeslot

**Es existieren correlation engines: Drools**

### C. Identifying logging events of interest

Nachrichten identifizieren die zu einem speziellen Event gehören

* Logdatenanalyse 
    * RDTool -> Holt-Winters Aberrant Behavior Detection [28].
        * Entdeckt abweichendes Verhalten 

BILD EINFÜGEN!! (DEMO??? Eher nicht!)


## V Implementation og Log Correlation and Consolidation in CLoud Environments

### A. Mode of Operation

1. Zentraler Syslog: rsyslog
** empfängt und normalisiert syslog messages
** Normalisierung mittels **liblognorm**

#### liblognorm

**Ziel:** 
Gleiche Events die verschieden codiert aus verschiedenen Quellen stammen
identifizieren und "Verbinden" und
Identifizierung unterschiedlicher Nachrichten mit der gleichen Bedeutung zur
Korrelierung, dazu werden eindeutige Tags der original Syslog-Meldung hinzugefügt.

A. Normalisierung: die tags: SSHFAILURE, SSHFAILURE werden speziellen Events zugeordnet.

**Beispiel einer Regelstruktur:**

        rule=SSHSUCCESS : Accepted password for %user:
        word% from %ip:ipv4% port %port : number% %protocol:word%
        
        rule=SSHFAILURE : Failed password for %user:
        word% from %ip:ipv4% port %port : number% %protocol:word%
        
        rule=SSHFAILURE : Failed password for invalid user %user:
        word% from %ip:ipv4% port %port : number% %protocol:word%

B. Serialisierung mithilfe von JSON (JavaScript Object Notation) und Weiterleitung zum rsyslog output-Modul und dort zur Korrelations-Software.

        {
            ”data”: {
                ”protocol”:”ssh2”,
                ”port” : ”54548”,
                ”ip”: ”10.0.23.4”,
                ”user”: ”root”
            } ,
            ”time”:”2014−01−29T16:06:00.000”,
            ”host”:”test.example.com”,
            ”facility”:”auth”,
            ”severity”:”info”,
            ”program” :”sshd ”,
            ”message”:”Failed password for root from
            10.0.23.4 port 54548 ssh2”,
            ”tags” : [”SSHFAILURE”]
        }

Mithilfe dieser Daten kann die Korrelations-Software [1] die Meldungen analysieren und an
Elasticsearch weiterleiten.
Nochmal: Die Korrelations-Software basiert auf der *Complex Event Processing (CEP) Engine Drools Fusion*

Drools hält alle Events die zu einer Regel gehören in einem MEM-Cache, andere Nachrichten
werden verworfen. Alle Nachrichten die zu einer Regel gehören werden korreliert, mit
einem FLAG versehen (das die erfolgreiche Korrelierung anzeigt) und einem Verweis auf
die Originalnachticht und in Elasticsearch gespeichert.

In Elasticsearch kann dann periodisch nach erfolgreichen Korrelationen gesucht
werden.

2. **Drools Fusion** als Korrelation und Konsolidierung

Mithilfe von *Drools Fusion* kann man zeitliche Schlussfolgerungen ziehen, 
dazu werden Regeln erstellt, die spezielle Events, durch eine zeitliche Folge von
Syslog-Nachrichten, beschreiben.

Erinnerung-Ziel: Den einen erfolgreichen Login auf einem System entdecken, zwischen
millionen gescheiterten Login-Versuchen.

A. Entdeckung einer Brute-Force-Attacke

* Um eine Attacke zu entdecken, muss zuvor eine Brute-Force-Attacke identifiziert werden
    * Drools-Regel *SSH brute-force attempt* mit Bezug auf Syslog-tag *SSHFAILURE*
    * basierend auf *username* und *host* bildunge mehrer *in-memory-queues*
        * bei mehr als 10 Versuchen innerhalb einer Minute -> Brute-Force Angriff

* Drools-Regel (1) fasst alle Diese Meldungen zu einer neuen Syslog-Meldung zusammen
    * facility=*security* und severity=*warning*, Message=*SSH brute-force attack*
    * neues Syslog-tag=**BRUTEFORCE**

        rule ”SSH brute−force attempt”
        no−loop
        when
            Message (   $host:host,
                        $user:data[”user”])
            $atts: CopyOnWriteArrayList(size >= 10)
                from collect(
                    Message(    tags contains”SSHFAILURE”,
                                host == $host,
                                data[”user” ] == $user)
                    over window : time (1m))
        then
            Message last = (Message) $atts.get($atts.size( )−1) ;

            for (Object f: $atts) {
                retract ( f ) ;
        }

            insert (messageFactory(last)
                . setTime(last.getTime( ))
                . setSeverity(Message.Severity.WARNING)
                . setFacility(Message.Facility.SECURITY)
                . setMessage(”SHH brute−force attack” +
                    ”for @{data.user} from @{data.ip}”)
                . addTag (”BRUTEFORCE”)
                . message( )) ;
        end

* Drools-Regel (2) *Successful SSH brute-force attack*
** trifft zu, wenn innerhalb einer Brute-Force-Attacke eine Login gelingt
** bezieht sich auf Syslog-tag *SSHSUCCESS* innerhalb von 10s nach den tags: *SSHFAILURE* und *BRUTEFORCE* + selber *username*.
*** severity=*emergency* und tSyslog-tag: *INCIDENT*

        rule ”Successful SSH brute−force attack”
        no−loop
        when
            $ att: Message ( tags contains ”SSHFAILURE” ,
                                tags contains ”BRUTEFORCE”,
                                $host: host ,
                                $user: data [”user”])
            $ suc: Message ( host == $host,
                                data [”user” ] == $user,
                                tags contains ”SSHSUCCESS” ,
                                this finishes[10 s] $att)
        then
            $att.addTag(”INCIDENT”);
            $att.setSeverity(Severity.EMERGENCY);
            $att.setMessage($att.getMessage( ) + ”[bruteforce]”) ;
        update ($att) ;
        end

        [ABBILDUNG: Mögliche Anzeige S.8(46)]

Die Daten können jetzt persistent gespeichert werden und/oder es kann ein Event an
an **Icinga/Nagios** gesendet werden.


3. Abspeichern der Daten mittel elasticsearch

## VI Log-Correlation in an Cloud-Environment

### A. Scalability in an OpensTack-Environment

* limitierender Faktor ist Drools in-memory-engine

**mögliche Lösungen:**
* mehrere Drools-Instanzen die mittels *ditributed memory* gelinked sind
** Keine OpenSource-Lösung verfügbar
** für große Enterprise-Clouds empfehlenswert


* Jede Applikation sendet ihre Logs zu einer dedizierten DROOL-VM
** Korrelierbare Applikationen senden ebenfalls zu einer dedizierten DROOL-VM

Die Korrelation erst über Elasticsearch-Abfragen zu realisieren scheint nicht sinnvoll
sein sein, da bei LOG-Bursts die Auswertung zu lange dauert.

        [BILD Seite 9 (47) (FIG 6)]

### C. Correlation with information from external management systems

* Externe Quellen sind ebenso möglich in Drools-Regeln einzupflegen
** erhöht die Rechen- und Speicherlast

**Beispiele**

* Events von einem Monitoringsystem (Icinga, OpenNMS)
* Daten von aktiven Netzwerkkomponenten (Router, Switche, Firewalls) über z.B.: *SNMP*
* Umweltdaten: Temp, Humidity, CO, CO2 Konzentration


## VII Performance Evaluation

### A. Test-bed setup
**Testumgebung**

Rackspace-VM (rsyslog + liblognorm):
* 4 i7 mit 2.8 GHz
* 8 GiB SPeicher
* SSD oder SATA

Mittels *loggen [43]* werden Nachrichten auf einem entfernten Host mit einer
1GBit/s Anbindung generiert und an obiges System weitergeleitet. Es wurde mit
verschiednen Storage-backends experimentiert:

        [TABELLE SEITE 11 (49)]

### A. Evaluation of the test-bed peak performance

* *Gerhards [44]*: rsyslog kann bis zu 250K Nachrichten pro Sekunde verarbeiten

**Ergebnis Storage-Backend Benchmark PEAK (ohne log-Normalisierung)**

* Ergebnisse direkt nach */dev/null*

        [FIG. 7]

### B. Evaluation of the correlation prototype performance

*loggen* -> rsyslog(+liblognorm) -> Storage-Backend

        [FIG. 7]

* Das Schreiben in eine Datei ist am schnellsten (kein Overhead)

* MySQL: TCP-Overhead

* Elasticsearch: REST-Interface hat HTTP-Overhead (daher noch langsamer als MySQL)


**jCorrelat(1)**

Durchsatz ohne Korrelierung von Events -> JCorrelat speichert Daten (nach normalisierung)
auch in Elasticsearch ab, allerdings direkt via Java-API, nicht über REST-Interface.


**jCorrelat(2)**

Durchsatz **mit** Korrelierung von Events. Genauso schnell wie das direkte Schreiben
nach Elasticsearch, aber mit Korrelation.

Weiter mögliche Geschwindigkeitssteigerung ist mittels rsyslog-Regeln möcglich:

So könnte zum Beispiel eine rsyslog-Regel erstellt werden, die lediglich aus speziellen
*facilities* die Logdaten zur Normalisierung an *liblognorm* weiterleitet. In diesem 
Beispiel aus der *facility* * auth*.

## VIII. Conclusion and future work

### Conclusion

* Prototyp für **automatische** Korreltation und Konsolidierung aus vercshiedenen Quellen
* Prototyp ist gut skalierbar (naiv und über *distributed memory*)
* Effiziente Verdichtung durch gruppierung
* Starke Steigerung der Auswertegeschwindigkeit von Logdaten
** Insbesondere wenn mehrere Systeme gleiche Logmeldung schicken
* Existierende Monitoringsysteme können mittels Plugins erweitert werden
** Senden von *traps*, Benachrichtigungen, Visualisierungen

* Speicher ist der limitierende Faktor
** Je mehr Speicher zur Verfügung und damit Vergrößerung der Drools-in-mem-pipeline
*** genauere Ergebnisse
 


### Future

* Korrelatione mit weiteren Daten
** detaillierter: exakte Interfaces, DPI
** Geo-Daten

* Korrelation mittels Prototyp kann uach auf anderer Datenbasis erfolgen (z.B. Elastic)
** historische Daten

* Vorgestellte Regeln können generalisiert werden um andere Verwendungszwecke abzudecken

* OpenStack Integration

* Integration in *ELK*-Stack




















 


