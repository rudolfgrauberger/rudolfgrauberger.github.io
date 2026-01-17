---
layout: post
comments: true
title:  "Entwickeln mit Visual Studio für Raspberry Pi mit Raspbian"
description: "Visual Studio 2017 für/auf Raspbian/Raspberry Pi inkl. Debuggen, IntelliSense-Unterstützung und Standardeingabe und Standardausgabe einrichten."
date:   2018-04-27 18:27:00 +0100
category: Entwicklung
hideToc: false
tags: [visualstudio, debuggen, build, raspberry, remote, raspberrypi, raspbian, c, cpp, c++, cmake, debug, archiv]
---

Für ein Projekt während des Studiums musste ich ein Projekt für/auf dem Raspberry Pi realisieren. Die Variante, mit vim oder nano auf dem Raspberry direkt zu entwickeln und dann mit gcc von Hand zu kompilieren, die uns von unseren Dozenten/Betreuern empfohlen wurde, war wegen der bereits gewohnten und lieb gewonnenen Komfortabilität von IDE's für mich keine Option.

In diesem Post beschreibe ich, was und wie man alles einrichten muss, damit man wie gewohnt in Visual Studio entwickeln, und das Debuggen und Deployen auf dem Raspberry Pi ausgeführt wird.

{{< archiv-note >}}

{{< notice info >}}
Mit der Version 15.7 von Visual Studio hat Microsoft, IntelliSense für Linux verbessert. Beim Verbinden mit dem Gerät, werden automatisch alle benötigten Includes von dem Zielsystem heruntergeladen und von IntelliSense genutzt.Dieser Beitrag sowie das Beispiel-Projekt wurden am 21.05.2018 daran angepasst.
{{< /notice >}}

<!--more-->

## 1. Einrichtung

Wie bei fasst allen Sachen, ist eine gewisse Einrichtung/Vorbereitung notwendig. In diesem Fall sind sogar Schritte auf beiden Seiten durchzuführen.

### 1.1 Visual Studio 2017

Als Erstes geht es darum, Visual Studio 2017, wenn noch nicht bereits bei der Installation geschehen über den Visual Studio Installer um das Toolset "Linux Entwicklung mit C++" zu erweitern.

![Visual Studio Installer](../visual_studio_installer_linux.png)

Dazu sind die Auswahlboxen in den markierten Bereichen zu setzen und der Visual Studio Installer mit Ändern zu bestätigen.

### 1.2 Raspberry Pi

Auf dem Raspberry Pi müssen **openssh-server, g++, gdb, zip** und **gdbserver** installiert sein. [2][3]

Mit der nachfolgenden Anweisung werden, wenn noch nicht vorhanden, diese heruntergeladen und installiert.

```bash
sudo apt-get install openssh-server g++ gdb zip gdbserver
```

## 2. Neues Projekt/Beispielprojekt

Da es Ziel dieser Anleitung sein soll, möglichst einfach und schnell mit Visual Studio auf dem Raspberry PI zu debuggen, habe ich auf GitHub ein Beispiel-Projekt [https://github.com/rudolfgrauberger/vs2017-raspberry-pi](https://github.com/rudolfgrauberger/vs2017-raspberry-pi) erstellt. Das bitte wie im Repository beschrieben clonen und dann in diese Anleitung zurückkehren.

### 2.1 Der erste Start

**Visual Studio 2017** starten und hier im Menü auf *Datei > Öffnen > CMake...* klicken. 

![Datei > Öffnen > CMake... ](../visual_studio_open_cmake.PNG)

In dem **CMake-Projekte öffnen (durch CMakeLists.txt oder CMakeCache.txt)**-Dialog dann bitte in das neue Verzeichnis wechseln und hier in das Unterverzeichnis ***example*** und hier die Datei **CMakeLists.txt** auswählen.

![CMakeLists.txt auswählen](../visual_studio_open_cmake_2.png)

Sobald (also etwas warten) in der Auswahlbox "x86-Debug" angezeigt wird, **Linux-Debug** auswählen.
![Visual Studio 2017 Linux-Debug](../visual_studio_linux-debug.png)

Es öffnet sich direkt das nachfolgende Fenster mit dem man die Verbindungsinformationen des Raspberry-PI eintragen muss.

![Mit Remotesystem verbinden](../visual_studio_add_ssh_remote.png)

Anstelle des Hostnamen kann auch die IP-Adresse eingetragen werden, alle anderen Informationen sollten selbsterklärend sein. Ihr solltet natürlich im selben Netz (egal ob WLAN oder LAN) mit dem Raspberry PI sein, damit die Verbindung hergestellt werden kann.

Sobald die Verbindung hergestellt wurde, werden die Header-Dateien vom Zielsystem für das IntelliSense in Visual Studio heruntergeladen und vorbereitet.

![Update Header for IntelliSense](../visual_studio_update_header.png)

Anschließend sieht man folgendes unten in der Ausgabe.

![CMake Remote Initialisierung](../visual_studio_cmake_remote_init.png)

In diesem Schritt wird das Projekt direkt auf dem Raspberry PI vorbereitet.

In der aktuellen Version von Visual Studio kann man leider noch nicht direkt über **F5** das Projekt starten, es fehlt nur noch ein Schritt (den ich in mit den zwei nachfolgenden Screenshots zeige).

Auf den kleinen Pfeil der Auswahlbox **Startelement auswählen** klicken.
![Startelement auswählen](../visual_studio_startelement.png)

Und hier dann **raspberry-example** auswählen

![raspberry-example auswählen](../visual_studio_startelement_raspberry_pi.png)

Und jetzt **F5** drücken oder im Menü **Debuggen > Starten** auswählen. Bei der nachfolgenden Ausgabe sieht man genau, was gemacht wird und das es auch tatsächlich funktioniert.

![F5 Ausgabe](../visual_studio_f5.png)

{{< notice info >}}
Nachtrag vom 21.05.2018
{{< /notice>}}

Damit man die Ausgaben auf der Konsole mit `printf()` sieht und damit man Tastatur-Eingaben z. B. mittelts `scanf()` vornehmen kann, muss man einmal im Menü **Debuggen > Linux Console** auswählen.

![Menü Debuggen > Linux Console](../visual_studio_linux_console.png)

Es öffnet sich die Linux Console, die - wie jedes Fenster in Visual Studio - an beliebiger Stelle angedockt werden kann.

![Linux Console Window](../visual_studio_linux_console_window.png)

Wenn man jetzt das Projekt über **Debuggen > Starten** oder **F5** ausführt, dann werden in der Linux Console die Ausgaben z. B. über `printf()` darauf ausgegeben und die Eingaben kann man direkt in das Fenster eingeben z. B. bei `scanf()`-Anweisungen.

![Linux Console Output](../visual_studio_linux_console_output.png)

Das war jetzt schon ziemlich cool, aber wenn das alles wäre, dann hätte ich nicht so viel Zeit in diesen Beitrag gesteckt.

Das besonders angenehme beim Arbeiten mit einer IDE wie Visual Studio ist, dass man IntelliSense-Unterstützung hat und man Debuggen kann. Das Visual Studio 2017 unter Windows beides (auch bei Linux exklusiven Funktionen) anbietet macht das Entwickeln für den Raspberry Pi erst so richtig angenehm.

### 2.2 Debuggen

Dafür gehen wir erstmal in dem Projektmappen-Explorer und klappen den Ordner **src** auf und öffnen die **main.c** durch einen Doppelklick.

![Projektmappen-Explorer](../visual_studio_project_explorer.png)

In der geöffneten Datei dann z.B. in der Zeile 8 einen Haltepunkt setzen, dann hält Visual Studio die Ausführung an der Stelle an. Bitte genau an die Stelle (roter Kreis auf dem Bild) klicken, damit ein Haltepunkt gesetzt wird.

![Haltepunkt setzen](../visual_studio_set_break_point.png)

Über den Menüpunkt **Debuggen > Starten** oder **F5** noch mal das Projekt starten. Nun hält Visual Studio auch an der nächstmöglichen zur gewünschten Stelle an. In diesem Fall ist es die Zeile 9.

![An Haltepunkt angehalten](../visual_studio_stopped_on_breakpoint.png)

Und jetzt kann man mit der Taste F10 (Prozedurschritt = geht in der aktuellen Datei Zeile für Zeile) wie gewohnt Debuggen. Zum Debuggen gehört auch, dass man sich Variablen-Werte anzeigen lassen kann und auch das funktioniert beim Zusammenspiel von Visual Studio und Raspberry Pi super.[1]

![Variablen überwachen](../visual_studio_debugging_watch.png)

### 2.3 IntelliSense

Ein weiteres sehr bequemes und nützliches Feature beim Entwickeln mit einer IDE ist die Unterstützung beim Schreiben des Codes. Dabei schlägt Visual Studio beim Tippen je nach Kontext Includes, Funktionen und Variablen vor, die man z.B. mit der Tabulator-Taste übernehmen kann.

Mit meinem Beispiel-Projekt unterstützt Visual Studio beim Include von Linux-Headern.

![IntelliSense Linux Header](../visual_studio_instellisense_linux_include.png)

Bei den Funktionsnamen
![IntelliSense Linux Header](../visual_studio_instellisense_linux_function.png)

Und bei den Parametern
![IntelliSense Linux Header](../visual_studio_instellisense_linux_parameter.png)

Damit viel Spaß und Erfolg beim Entwickeln für den Raspberry-Pi mit Visual Studio. Lasst gerne Feedback da und meldet euch, sollte etwas nicht oder nicht mehr stimmen.

___

## Hinweise und Quellen

### Hinweise

[1] Das stimmt technisch nicht ganz, denn es liegt an dem guten Zusammenspiel zwischen Visual Studio und **gdb** der auf dem Raspberry installiert wurde. Das würde aber an der Stelle ein weiteres Detail in den Beitrag bringen und ich wollte es an der Stelle möglichst einfach halten.

### Quellen

[2] Install and configure the Linux Workload. [https://docs.microsoft.com/en-us/cpp/linux/download-install-and-setup-the-linux-development-workload](https://docs.microsoft.com/en-us/cpp/linux/download-install-and-setup-the-linux-development-workload) (27.04.2018).

[3] IntelliSense for Remote Linux Headers. [https://blogs.msdn.microsoft.com/vcblog/2018/04/09/intellisense-for-remote-linux-headers/](https://blogs.msdn.microsoft.com/vcblog/2018/04/09/intellisense-for-remote-linux-headers/) (21.05.2018)

Marc Goodner - MSFT: Visual C++ for Linux Development. [https://blogs.msdn.microsoft.com/vcblog/2016/03/30/visual-c-for-linux-development/](https://blogs.msdn.microsoft.com/vcblog/2016/03/30/visual-c-for-linux-development/) (27.04.2018)

Configure a Linux CMake project. [https://docs.microsoft.com/en-us/cpp/linux/cmake-linux-project#building-a-supported-cmake-release-from-source](https://docs.microsoft.com/en-us/cpp/linux/cmake-linux-project#building-a-supported-cmake-release-from-source) (27.04.2018)
