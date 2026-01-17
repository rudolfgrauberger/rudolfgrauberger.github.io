---
layout: post
comments: true
title:  "Microsoft Word - Bilder im Inhaltsverzeichnis?"
description: "In diesem Post beschreibe ich, wie man vorgeht, wenn Bilder in Inhaltsverzeichnis in Microsoft Word 2016 angezeigt werden."
date:   2018-06-16 20:56:00 +0100
category: Office
hideToc: true
tags: [microsoft, office, word, inhaltsverzeichnis, dokumentation, archiv]
---

In einer unserer letzten Projektdokumentation stellten wir in Word 2016 fest, dass Bilder im Inhaltsverzeichnis angezeigt wurden. Als erstes vermuteten wir, dass Word mit unserem Dokument oder der darin befindlichen Struktur nicht klarkommt.

Das sah ungefähr so aus:
<!--more-->

{{< archiv-note >}}

## Problem

![Word 2016 - Inhaltsverzeichnis](../word_inhaltsverzeichnis.PNG)

Die Frage ist, warum werden plötzlich Bilder im Inhaltsverzeichnis angezeigt? Das Bild steht - wie man im nachfolgenden Bild sehen kann - in dem Abschnitt zwischen Texten und trotzdem wird nur das Bild in dem Inhaltsverzeichnis angezeigt.

![Word 2016 - Bild zwischen Text](../word_inhaltsverzeichnis_abschnitt.PNG)

## Lösung

Der Grund für die "fehlerhafte" Darstellung des Bildes im Inhaltsverzeichnis ist, dass dem Bild eine **Überschrift**-Formatvorlage zugewiesen ist.

![Word 2016 - Bild mit Überschrift-Formatsvorlage](../word_inhaltsverzeichnis_ueberschrift.PNG)

Sobald wir das Bild markieren und die Formatvorlage auf **Standard** zurückstellen und dann das Inhaltsverzeichnis aktualisieren,

![Word 2016 - Verzeichnis aktualisieren](../word_inhaltsverzeichnis_aktualisieren.png)

verschwindet das Bild wieder aus dem Inhaltsverzeichnis.

![Word 2016 - Verzeichnis ohne Bild](../word_inhaltsverzeichnis_gefixt.PNG)

## Fazit

Was erstmal wie ein Fehler in Word aussah, stellte sich nachher als Bedienungsfehler von uns heraus.

Da Word sich standardmäßig nur an die Formatvorlagen für die Überschriften orientiert, werden alle Objekte auf die man Formatvorlagen anwenden kann (getestet habe ich Bilder, Piktogramme, SmartArt und Diagramme) auch in das Inhaltsverzeichnis aufgenommen.

Wenn man dieses Wissen gezielt einsetzt, kann man auch das Inhaltsverzeichnis etwas aufwerten (wenn man es will).
