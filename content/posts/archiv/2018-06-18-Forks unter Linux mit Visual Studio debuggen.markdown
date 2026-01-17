---
layout: post
comments: true
title:  "Forks unter Linux mit Visual Studio debuggen"
description: "In diesem Beitrag geht es darum, wie man den Kind-Prozess nach dem Fork mit Visual Studio debuggen kann."
date:   2018-06-19 18:00:00 +0100
category: Entwicklung
hideToc: true
tags: [visualstudio, debuggen, debugger, windows, linux, raspberry, remote, raspberrypi, raspbian, c, cpp, c++, debug, gdb, fork, child, process, parent, archiv]
---
Beim Entwickeln auf dem Raspberry Pi/Raspbian mit Visual Studio (wie das geht, beschreibe ich in dem Beitrag [Entwickeln mit Visual Studio für Raspberry Pi mit Raspbian](/posts/archiv/2018-04-18-entwickeln-mit-visual-studio-für-raspberry-pi-mit-raspbian)) bin ich darauf gestoßen, dass man beim Debuggen nach dem `fork()`-Aufruf nur den Elternteil Schritt für Schritt durchgehen kann.

<!--more-->

{{< archiv-note >}}

## Problem

An dem kleinen nachfolgenden Beispiel lässt sich das bereits schön zeigen:

```cpp
#include <stdio.h>
#include <unistd.h>

int main(int argc, char* argv[]) {

   int pid = fork();

   if (pid > 0) { // parent process
      printf("PID: %d: Ich bin der Elternprozess", getpid()); // 1
      getchar();
   }
   else if (pid == 0) { // child process
      printf("PID: %d: Ich bin der Kindprozess", getpid()); // 2

      // Irgendwelche andere Aufgaben die in dem Kindprozess ausgeführt werden.
   }

   return 0;
}
```

Wenn ich in den Zeilen die ich mit // 1 und // 2 markiert habe jeweils einen Haltepunkt setzte und dann den Debugger starte, dann hält der Debugger bei // 1 aber leider nicht auch bei // 2.

Allerdings wird auch der Text "X: Ich bin der Kindprozess" ausgegeben, denn der Kindprozess läuft ja ab dem Moment des Forks als eigenständiger Prozess.

## Lösung / Workaround

Ich habe eine Möglichkeit gefunden, wie man mit Visual Studio entweder den Kindprozess oder Elternprozess debuggen kann.

Um den Kindprozess zu debuggen, muss in der launch.vs.json in dem Projekt-Root-Verzeichnis .vs folgendes als Element in dem "setupCommands"-Array hinzugefügt werden.

```json
{
   "text": "-gdb-set follow-fork-mode child",
   "ignoreFailures": true
}
```

Die launch.vs.json-Datei kann man auch direkt über Visual Studio öffnen.

![Visual Studio - launch.vs.json öffnen](../visual_studio_open_launch.vs.json.png)

Wenn man das "child" durch "parent" ersetzt, dann folgt der Debugger wieder dem Elternprozess.

Sodass die komplette Datei für das Debuggen des Kindprozesses nach dem Hinzufügen des Elements wie folgt aussieht:

**launch.vs.json**

```json
{
  "version": "0.2.1",
  "defaults": {},
  "configurations": [
    {
      "type": "cppdbg",
      "name": "raspberry-example",
      "project": "CMakeLists.txt",
      "projectTarget": "raspberry-example",
      "cwd": "${debugInfo.remoteWorkspaceRoot}",
      "program": "${debugInfo.fullTargetPath}",
      "MIMode": "gdb",
      "externalConsole": true,
      "remoteMachineName": "${debugInfo.remoteMachineName}",
      "pipeTransport": {
        "pipeProgram": "${debugInfo.shellexecPath}",
        "pipeArgs": [
          "/s",
          "${debugInfo.remoteMachineId}",
          "/p",
          "${debugInfo.parentProcessId}",
          "/c",
          "${debuggerCommand}",
          "--tty=${debugInfo.tty}"
        ],
        "debuggerPath": "/usr/bin/gdb"
      },
      "setupCommands": [
        {
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        },
        {
          "text": "-gdb-set follow-fork-mode child",
          "ignoreFailures": true
        }
      ],
      "visualizerFile": "${debugInfo.linuxNatvisPath}",
      "showDisplayString": true
    }
  ]
}
```

In diesem Fall wird aber der jeweils andere Prozess direkt nach dem Fork weiter ausgeführt. Wenn man jetzt z. B. nur den Kindprozess debuggen möchte, ohne dass z. B. die Standardeingabe/-ausgabe von dem anderen Prozess zugemüllt wird, dann muss man gdb mitteilen, dass er sich nicht von dem jeweils anderen Prozess trennen soll und das macht man mit dem nachfolgenden Element/Befehl:

```json
{
   "text": "-gdb-set detach-on-fork off",
   "ignoreFailures": true
}
```

Mit diesem Block wird der jeweils andere Prozess pausiert und das ermöglicht es etwas komfortabler den gewünschten Prozess zu debuggen. Kombiniert sieht die Datei komplett jetzt so aus:

**launch.vs.json**
```json
{
  "version": "0.2.1",
  "defaults": {},
  "configurations": [
    {
      "type": "cppdbg",
      "name": "raspberry-example",
      "project": "CMakeLists.txt",
      "projectTarget": "raspberry-example",
      "cwd": "${debugInfo.remoteWorkspaceRoot}",
      "program": "${debugInfo.fullTargetPath}",
      "MIMode": "gdb",
      "externalConsole": true,
      "remoteMachineName": "${debugInfo.remoteMachineName}",
      "pipeTransport": {
        "pipeProgram": "${debugInfo.shellexecPath}",
        "pipeArgs": [
          "/s",
          "${debugInfo.remoteMachineId}",
          "/p",
          "${debugInfo.parentProcessId}",
          "/c",
          "${debuggerCommand}",
          "--tty=${debugInfo.tty}"
        ],
        "debuggerPath": "/usr/bin/gdb"
      },
      "setupCommands": [
        {
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        },
        {
          "text": "-gdb-set follow-fork-mode child",
          "ignoreFailures":  true
        },
        {
          "text": "-gdb-set detach-on-fork off",
          "ignoreFailures": true
        }
      ],
      "visualizerFile": "${debugInfo.linuxNatvisPath}",
      "showDisplayString": true
    }
  ]
}
```

## Fazit

Da es sich beim Forken ja um eine Art der Nebenläufigkeit handelt, und das immer hart zu debuggen ist, ist insbesondere der Befehl **detach-on-fork off** sehr nützlich. Da meistens in dem Kindprozess ein nicht unwesentlicher Teil der Anwendungslogik liegt, bin ich sehr froh, dass es die Möglichkeit gibt, diesen zu debuggen.

Das man, das (noch) nicht einfach in Visual Studio einstellen kann, zeigt das Microsoft mit der Unterstützung für Linux in Visual Studio noch nicht fertig ist. Solange man aber mit jedem Update Verbesserungen sieht, bin ich zuversichtlich, dass es noch komfortabler wird (wie man es eigentlich von Visual Studio gewohnt ist!).

___

### Quellen

How can I switch between different processes fork() ed in gdb? [https://stackoverflow.com/a/6223890/6624218](https://stackoverflow.com/a/6223890/6624218) (19.06.2018).
