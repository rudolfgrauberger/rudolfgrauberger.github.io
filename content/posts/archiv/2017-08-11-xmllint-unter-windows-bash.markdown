---
layout: post
comments: true
title:  "xmllint unter Windows mit Bash benutzen"
description: "Ein Linux-Programm (xmllint) mit dem Subsystem for Linux (ohne Virtual Machine) unter Windows benutzen."
date:   2017-08-11 10:18:42 +0100
category: Entwicklung
hideToc: true
tags: [xmllint, xml, windows, bash, linux, dev, tools, schema, validate, cmd, archiv]
---

Nach einer Erweiterung der XML-Konfigurations-Dateien in einem Open-Source Projekt, sind die Validierungs-Tests in der CI fehlgeschlagen. In diesem Projekt werden die XML-Dateien gegen eine rng-Datei ([RELAX NG-Schema](http://relaxng.org/)) validiert und ich habe das Schema noch nicht um meine Erweiterungen ergänzt.

Ich stellte fest, dass für die Validierung das Tool [**xmllint**](http://infohost.nmt.edu/tcc/help/xml/lint.html) (Bestandteil von [libxml2](http://xmlsoft.org/)) verwendet wird. Um meine Änderungen schnell selber prüfen zu können, wollte ich das Tool auf meiner Maschine ausführen und die XML-Dateien gegen die RNG-Dateien validieren.

<!--more-->

{{< archiv-note >}}

### Ausgangssituation

Kleiner Auszug aus der cppcheck-cfg.rng (mit meinen Änderungen):

```xml
<?xml version="1.0"?>
<grammar xmlns="http://relaxng.org/ns/structure/1.0" datatypeLibrary="http://www.w3.org/2001/XMLSchema-datatypes">
<start>
<element name="def">
  <optional>
    <attribute name="format">
      <value>2</value>
    </attribute>
  </optional>
  <zeroOrMore>
    <choice>

      <!-- ... -->

      <!-- Das sind meine Änderungen -->
      <element name="include">
        <attribute name="name"><ref name="DATA-EXTNAME"/></attribute>
      </element>

      <!-- ... -->

    </choice>
  </zeroOrMore>
</element>
</start>

<!-- ... -->

<define name="DATA-EXTNAME">
  <data type="string">
    <param name="pattern">[a-zA-Z_][a-zA-Z_0-9:,]*</param>
  </data>
</define>

<!-- ... -->

</grammar>
```

test.xml

```xml
<?xml version="1.0"?>
<def format="2">
   <!-- Posix Headers, Source: http://pubs.opengroup.org/onlinepubs/9699919799/idx/head.html -->
   <include name="aio" />
   <include name="arpa/inet" />
</def>
```

### Ausführen unter Windows

Normalerweise kennt man es unter Windows so, dass man einen Installer oder ein ggf. auch noch ein zip-Verzeichnis herunterlädt und dann eine .exe-Datei ausführen kann.

Bei dem Tool xmllint geht es leider nicht so einfach, obwohl die Entwickler unter [ftp://xmlsoft.org/libxml2/win32/](ftp://xmlsoft.org/libxml2/win32/) anscheinend für Windows kompilierte Dateien zur Verfügung stellen reicht es hier leider nicht aus, nur das *libxml2-2.7.8.win32.zip* herunterzuladen und auszupacken.

```batch
C:\Dev\libxml2-2.7.8.win32\bin>xmllint.exe --noout --relaxng D:\cppcheck-cfg.rng D:\test.xml
```

Beim Ausführen erhält man die schöne nachfolgende Meldung:

![Dependency Error](../xmllint_dependency_error.png)

Jetzt muss man aus dem Ftp-Verzeichnis auch noch die Abhängigkeits-Archive herunterladen, extrahieren und die benötigten Dlls in das *libxml2-2.7.8.win32\bin*-Verzeichnis kopieren und dann nochmals probieren. In diesem Fall ist es nur noch das *iconv-1.9.2.win32.zip* und der Aufwand geht noch. Bei noch mehr Abhängigkeiten wird es aber ein wenig nervig/lästig.

**PS:** ***Mit ist bewusst, dass diese Tools nicht für Endanwender gedacht sind und von daher dieser Aufwand (die Abhängigkeiten zusammenstellen) durchaus nicht zu viel ist. Es geht mir aber darum einen anderen möglichen Weg zu zeigen.***

Ab Windows 10 haben wir die Linux-Bash in Windows integriert bekommen. Da habe ich mich gefragt warum dann nicht über Linux die Validierung durchführen. Unter Linux werden die Abhängigkeiten von Paketen die über die Paketverwaltung verteilt werden, automatisch heruntergeladen und das Programm ist direkt verfügbar.

### Linux Programm unter Windows benutzen
Als erstes startet man die "Bash on Ubuntu on Windows", die man durch die Windows-Taste und die Eingabe von "Bash" in die Suchleiste aufgelistet bekommt. Oder alternativ öffnet man einfach die Eingabeaufforderung (STRG+R, cmd + ENTER) und gibt hier "bash" ein.

Sollte die Windows-Suche keine Ergebnisse bringen oder in der Eingabeaufforderung die nachfolgende Meldung ausgegen werden, dann ist das Linux-Subsystem noch nicht aktiviert ([Linux Bash unter Windows 10 aktivieren](https://www.deskmodder.de/blog/2016/04/07/linux-bash-unter-windows-10-aktivieren/)).

```batch
C:\>bash
Der Befehl "bash" ist entweder falsch geschrieben oder
konnte nicht gefunden werden.
```

Jetzt aber weiter mit der Bash.

**xmllint (bestandteil von libxml2-utils) installieren**

```shell
root@xxx:~# apt-get install libxml2-utils
```

Nach dem Abschluss des Befehls durch Drücken der Taste ENTER wartet man ein kurzen augenblich und schon ist das Paket installiert.

**Jetzt können wir ganz einfach mit der Bash in das gewünschte Verzeichnis navigieren:**

```shell
root@xxx:~# cd /mnt/d/
```

**Und dann ausführen:**

```shell
root@xxx:/mnt/d# xmllint --noout --relaxng cppcheck-cfg.rng test.xml
test.xml:5:  element include: Relax-NG validity error : Element include failed to validate attributes
test.xml fails to validate
```

Und schon funktioniert das Programm. :-)

### Fazit

Beides ist aufjeden Fall möglich und das ist auch gut so. Ich bin aber auf jeden Fall ein Freund der Bash unter Windows. So können sich die Entwickler weiterhin um die Programme auf den jeweiligen Plattformen kümmern, statt alles permanent zu portieren und für alle möglichen Plattformen anzupassen und ich muss auch unter Windows auf garnicht verzichten.