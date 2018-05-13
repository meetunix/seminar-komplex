# Konspekt zu:
# Chapter 26 aus Network-Security-Essentials

Cyberspace = Voneinander abhängiges Netzwerk aus IT-Infrastruktur
* Internet
* embedded systems
* computer systems




## 26.2 Probing Campigns

Es wird ein neuer Ansatz von Internetweiten *fingerprinting* durch Sondierung (*probing*)
vorgestellt.

Dabei wird IP-Addressraum des *darknet* analysiert.

**Darknet**
Komische definition!!!!???

### 26.2.1 Inferring Probing Events

Es wird ein neuer Ansatz von Internetweiten *fingerprinting* durch Sondierung (*probing*)
vorgestellt. Die verfügbaren *scanning detections systems* habe eine zu große
Ungenauigkeit beim fingerprinting.

Die zugrunde liegende Überlegung der Vorschlagenen Methode äußert sich dadurch,
dass unabhängig von Quelle, Srategie und Ziel der Sondierung, die Aufklärung durch
schon bekannte scanverfahren generiert wird.

Wir beobachten das eine Nummer dieser SOndierungstechniken eine gleiche zeitliche
Korrelation und Ähnlichkeit besitzen, wenn man deren Traffic Korreliert.

Mit anderen Worten: Die Beobachtung zeigt, dass man die Scanning-Techniken basieren auf
der Traffic-Korrelation clustern kann.

Anschließed kann man zwischen probing und anderem darknet-Verkehr unterscheiden.

Man kann zudem den Scantraffic einer bestimmten Scan-Technik zuordnen.

Um exakt zu identifizieren, welche scan-technik verwendet wurde, wird die relative Nähe
über statistische Methoden im Vergleich zu den Scan-Techniken aus diesem Cluster 
ermittelt.

#### empirisches Resultat (1)

10 GiB of Darknet Data:
* 700 Sessions
* 612 Sessions können Probing zugeordnet werden
* Nach verwendung von sfportscan (snort-module): 590 Scans werden identifiziert
** 2% Falsch Positive Ergebnisse
** keine Falsch Negativen






