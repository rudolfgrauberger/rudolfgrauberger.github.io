---
layout: post
comments: true
title:  "Docker für Windows: Fehler beim Starten des Jenkins Blueocean-Containers"
description: "Docker für Windows-Fehler beim Mappen von gewissen Portnummern. In diesem Beispiel beim Starten des Jenkins Blueocean-Containers nach der offiziellen Installationsanleitung."
date:   2019-04-07 23:02:00 +0100
category: Entwicklung
hideToc: true
tags: [docker, windows, dev, docker for windows, port, hyper-v, jenkins, archiv]
---

Beim Durcharbeiten der [Jenkins Installationsanleitung](https://jenkins.io/doc/book/installing/) erhielt ich unter Windows (Version: 1809 Build 17763.404) die nachfolgende Fehlermeldung:

```log
Error response from daemon: 
driver failed programming external connectivity on endpoint gracious_cartwright (fdab7a0981070a7acbe7ee7e7868adaf10953fdc0a81c46ed114357bee65eaca): 
Error starting userland proxy: Bind for 0.0.0.0:50000: unexpected error Permission denied.
```

Die Anleitungen, bei denen ein Neustart des Systems oder von Docker empfohlen wird, habe ich alle ausprobiert und die haben nichts gebracht.[1]

In diesem Beitrag gibt es die Lösung und einige Hintergrundinformationen.

<!--more-->

{{< archiv-note >}}

# Problem

Die Fehlermeldung wurde bei dem nachfolgenden Befehl ausgegeben (kann Copy&Paste benutzt werden)

```shell
docker run --name jenkins-blueocean `
-u root `
--rm -d `
-p 8080:8080 `
-p 50000:50000 `
-v jenkins-data:/var/jenkins_home `
-v /var/run/docker.sock:/var/run/docker.sock `
jenkinsci/blueocean
```

# Lösung

Anderen Port verwenden oder Port 50000 reservieren.

#### 1. Einen anderen Port benutzen

Diese Lösung betrachte ich maximal als Workaround.

Wenn man mit JNLP-basierten Jenkins Agenten über den Standardport arbeiten muss, dann hilft einem das garnicht weiter! Außerdem weiß ich damit natürlich auch nicht welchen Port man ohne Weiteres benutzen darf... (für Infos dazu siehe [Hintergrundinformationen](#hintergrundinformationen))

#### 2. Port 50000 reservieren

Dafür muss man mit den folgenden Befehlen (Powershell `Als Administrator` starten)...

1. Hyper-V deaktivieren, da sonst keine Änderungen an den Einstellungen möglich sind (erfordert einige Neustarts)

   ```shell
   dism.exe /Online /Disable-Feature:Microsoft-Hyper-V
   ```

2. Port 50000 zu der Liste der Portausschlüsse hinzufügen (damit Hyper-V sich den Port nicht schnappt!)

   ```shell
   netsh int ipv4 add excludedportrange protocol=tcp startport=50000 numberofports=1
   ```

3. Hyper-V aktivieren (Neustart erforderlich)

   ```shell
   dism.exe /Online /Enable-Feature:Microsoft-Hyper-V /All
   ```

4. Docker mit Port 50000 benutzen

   ```shell
   docker run --name jenkins-blueocean `
   -u root `
   --rm -d `
   -p 8080:8080 `
   -p 50000:50000 `
   -v jenkins-data:/var/jenkins_home `
   -v /var/run/docker.sock:/var/run/docker.sock `
   jenkinsci/blueocean
   ```

Die Lösung habe ich nicht selber gefunden, sondern aus einem Issue von Docker Desktop for Windows.[4]

# Hintergrundinformationen

Der Port 50000 gehört ja offensichtlich weder zu den reservierten (0-1023), noch zu den registrierten (1024-49151) sondern zu den dynamischen Ports (49152-65535) und die kann/darf man ohne Probleme nutzen.[2]

Da dieser Port ja nicht genutzt werden kann, muss der Port bereits von einer Anwendung genutzt werden, dachte ich zumindest. Das prüfte ich mit dem Befehl:

```shell
netstat -p TCP -a
```

Hier wurden zwar einige Ports aufgeführt, der Port 50000 war aber nicht dabei. Also muss es ein anderes Problem bzw. einen anderen Grund geben!

Nach einiger Recherche habe ich einen Beitrag von Microsoft gefunden.[3] Nachdem gibt es Möglichkeiten Bereiche von Ports irgendwie zu reservieren, die dann aus der automatischen Vergabe von Ports ausgeschlossen sind.

Mit den Begriffen `excludedportrange` und `docker` bin ich dann mithilfe einer Suchmaschine auf den Beitrag mit der Lösung gestoßen.[4]

Das Problem liegt hier an dem Port 50000, der sich bei mir von Windows für die IPv4-Schnittstelle in einer Ausschlussliste befindet und von Hyper-V genutzt wird. Doch woher weiß ich, welche Ports sich in der Ausschlussliste befinden?

Das erfährt man ganz einfach durch den Befehl

```shell
netsh interface ipv4 show excludedportrange protocol=tcp
```

Das Ergebnis sieht bei mir so aus

```log
Portausschlussbereiche für das Protokoll "tcp"

Startport      Endport
----------    --------
     28380       28380
     28385       28385
     49711       49810
     49811       49910
     49911       50010
     50111       50210
     50211       50310
     51099       51198

* - Verwaltete Portausschlüsse.
```

Über die Portausschlussbereiche wird sichergestellt, dass wenn eine Anwendung einen zufälligen bzw. keinen bestimmten Port (auch [ephemeral Port](http://netzikon.net/lexikon/e/ephemeral-ports.html) genannt) benötigt, diese auf keinen Fall aus diesen Bereichen gezogen wird.[3]

Ich habe nach dem Deaktivieren von Hyper-V den Befehl noch mal ausgeführt und siehe da, die Ausgabe sieht bei mir um einiges kürzer aus:

```log
Portausschlussbereiche für das Protokoll "tcp"

Startport      Endport
----------    --------
     28380       28380
     28385       28385

* - Verwaltete Portausschlüsse.
```

Das sind die reservierten Ports bzw. Bereiche, die von Anwendungen nur explizit angefordert werden können. Das heißt, dass die anderen Ports (einschließlich 50000) von Hyper-V eingetragen werden.

Dass Docker diesen Port, obwohl er explizit angefordert wird, nicht benutzen kann, liegt daran, dass Hyper-V diese Ports für sich beansprucht und diese bereits reserviert/blockiert.

Aus diesem Grund reservieren wir den Port 50000 und aktivieren erst danach wieder Hyper-V. Da Hyper-V die Portausschlüsse beachtet, reserviert und blockiert er den Port 50000 nicht mehr und der steht für uns zur Verfügung.

Bei der erneuten Abfrage der reservierten Bereiche, sieht man nun das die 50000 nun auch als eigener Eintrag da aufgeführt ist.

```log
Portausschlussbereiche für das Protokoll "tcp"

Startport      Endport
----------    --------
     28380       28380
     28385       28385
     49708       49807
     49808       49907
     50000       50000     *
     50001       50100
     50101       50200
     50201       50300
     50301       50400

* - Verwaltete Portausschlüsse.
```

Wenn wir jetzt den ursprünglichen Befehl zum Starten des Containers ausführen, dann sehen wir, dass das Binden des Ports 50000 vom Container zu Port 50000 auf dem Host jetzt funktioniert.

---
Quellen

[1] Docker on Windows 10 “driver failed programming external connectivity on endpoint”, [https://stackoverflow.com/questions/44414130/docker-on-windows-10-driver-failed-programming-external-connectivity-on-endpoin](https://stackoverflow.com/questions/44414130/docker-on-windows-10-driver-failed-programming-external-connectivity-on-endpoin), 07.04.2019

[2] Internet Assigned Numbers Authority (IANA) Procedures for the Management of the Service Name and Transport Protocol Port Number Registry, [https://tools.ietf.org/html/rfc6335#section-6](https://tools.ietf.org/html/rfc6335#section-6), 07.04.2019

[3] Microsoft, You cannot exclude ports by using the ReservedPorts registry key in Windows Server 2008 or in Windows Server 2008 R2, [https://support.microsoft.com/de-de/help/2665809/you-cannot-exclude-ports-by-using-the-reservedports-registry-key-in-wi](https://support.microsoft.com/de-de/help/2665809/you-cannot-exclude-ports-by-using-the-reservedports-registry-key-in-wi), 07.04.2019

[4] Emad Nashed (enashed), [https://github.com/docker/for-win/issues/3171#issuecomment-459205576](https://github.com/docker/for-win/issues/3171#issuecomment-459205576), 07.04.2019
