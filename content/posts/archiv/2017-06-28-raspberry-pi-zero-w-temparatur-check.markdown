---
layout: post
comments: true
title:  "Raspberry Pi Zero W Temperatur Test"
date:   2017-06-29 11:23:42 +0200
category: Entwicklung
hideToc: true
tags: [raspberry, temperatur, test, zero, pi, iot, archiv]
---

Kann ein Raspberry Pi Zero W überhitzen? In diesem Artikel beschreibe ich Tests die ich gemacht habe und meine Ergebnisse.

<!--more-->

{{< archiv-note >}}

## Einleitung

Wie einigen anderen auch macht es mir hin und wieder spaß mit dem Raspberry Pi rumzuspielen. Hin und wieder aber realisiere ich damit für mich aber auch für andere produktive Lösungen mit einem Raspberry Pi. Umgesetzt habe ich u. A. bereits eine individuelle WLAN-Steuerung umgesetzt und habe alte (USB-)Drucker Netzwerk (LAN/WLAN) fähig gemacht.

Dabei beschäftigt mich immer wieder die Frage ob das Raspberry Pi vielleicht im Dauereinsatz oder unter Volllast überhitzt und damit vielleich sogar einen Brand auslöst.

In diesem speziellen Fall habe ich ein Raspberry Pi Zero W eingerichtet, dass als Druckerserver einem alten Drucker neues Leben einhauchen sollte. Da ich dies bezüglich ruhiger Schlafen wollte, habe ich dazu einiges Recherchiert und auch einige Tests gemacht.

## Recherche

Meine Recherche ergab, dass das Raspberry Pi bzw. die CPU bei einer Temparatur von ca. 80° auf 600 MHz runter regelt. Auch Tests wie z. B. hier (https://www.youtube.com/watch?v=e6okZKRwnTQ) zeigen, dass die Temparatur gedeckelt ist.

## Tests

Leider habe ich aber zu dem Raspberry Pi Zero keine solche Informationen gefunden und habe beschlossen selbst Tests durchgeführt.

### Rahmenbedingungen

Ich habe mehrere Tests unter folgenden Rahmenbedinungen durchgeführt.

| Name | Wert |
|---------------|---------------|
| Raumtemperatur | ca. 20° |
| Model | [Raspberry Pi Zero W V1.1](https://www.amazon.de/dp/B06XCYGP27/ref=cm_sw_r_cp_dp_T2_aEdvzbE1JXTDW) |
| Betriebssystem | Raspbian Lite (June 2017) |
| MicroSD | [SanDisk Ultra microSDHC 16GB bis zu 80 MB/Sek, Class 10](https://www.amazon.de/dp/B013UDL5V6/ref=cm_sw_r_cp_dp_T2_Wjdvzb9XW65T7) |
| Netzteil | [Wicked Chili 3100mA / 15.5W Netzteil für Raspberry Pi 3 B / Pi Zero / Pi 2 B / Pi Model B+ (B Plus) / Banana Pi / Model B / Modell A (microUSB Stecker, 100-240V, 3.100mA, 110 cm)](https://www.amazon.de/dp/B01N13021D/ref=cm_sw_r_cp_dp_T2_aldvzb4WTNWQT) |

Installierte zusätzliche Software: Samba, cups, sysbench

Test (Shell-Skript: test.sh):

```shell
#!/bin/bash
echo "Start: $(date --rfc-3339=seconds)"
vcgencmd measure_temp
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=4 run >/dev/null 2>&1
vcgencmd measure_temp
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=4 run >/dev/null 2>&1
vcgencmd measure_temp
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=4 run >/dev/null 2>&1
vcgencmd measure_temp
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=4 run >/dev/null 2>&1
vcgencmd measure_temp
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=4 run >/dev/null 2>&1
vcgencmd measure_temp
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=4 run >/dev/null 2>&1
vcgencmd measure_temp
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=4 run >/dev/null 2>&1
vcgencmd measure_temp
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=4 run >/dev/null 2>&1
vcgencmd measure_temp
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=4 run >/dev/null 2>&1
vcgencmd measure_temp
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=4 run >/dev/null 2>&1
vcgencmd measure_tempecho
echo "Ende: $(date --rfc-3339=seconds)"
```

Shell-Skript-Datei ausführbar machen

```shell
pi@printerserver:~ $ chmod +x test.sh
```

In folgenden Fällen habe ich diesen Test ausgeführt:

1. Ohne Gehäuse, Ohne Kühlkörper
2. Mit [Gehäuse](https://www.amazon.de/gp/product/B01FHDXNNU/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1), Ohne Kühlkörper
3. Ohne Gehäuse, Mit [Kühlkörper](https://www.amazon.de/dp/B06X3X948R/ref=cm_sw_r_cp_dp_T2_6Y.uzbRTEZ03Y)
4. Mit [Gehäuse](https://www.amazon.de/gp/product/B01FHDXNNU/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1), Mit [Kühlkörper](https://www.amazon.de/gp/product/B01FHDXNNU/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1)

Zwischen den Tests habe ich Pausen von jeweils mind. 1 Stunden eingelegt, in dem das Raspberry Pi Zero W vom Netzteil getrennt war.

### Test 1

**Konfiguration:** Ohne Gehäuse, Ohne Kühlkörper

Skript ausführen:

```shell
pi@printerserver:~ $ ./test.sh
```

Der Test erzeugt folgende Ausgabe:

```log
Start: 2017-06-29 13:20:43+02:00
temp=34.7'C
temp=47.6'C
temp=48.7'C
temp=49.8'C
temp=49.8'C
temp=49.8'C
temp=44.4'C
temp=46.5'C
temp=48.2'C
temp=48.7'C
temp=48.2'C
Ende: 2017-06-29 15:53:01+02:00
```

### Test 2

**Konfiguration:** Mit Gehäuse, Ohne Kühlkörper

Skript ausführen:

```shell
pi@printerserver:~ $ ./test.sh
```

Der Test erzeugt folgende Ausgabe:

```log
Start: 2017-06-29 16:58:50+02:00
temp=35.2'C
temp=54.1'C
temp=55.1'C
temp=55.1'C
temp=55.1'C
temp=56.2'C
temp=56.2'C
temp=56.2'C
temp=55.7'C
temp=55.1'C
temp=55.1'C
Ende: 2017-06-29 19:30:30+02:00
```

### Test 3

**Konfiguration:** Ohne Gehäuse, Mit Kühlkörper

Skript ausführen:

```shell
pi@printerserver:~ $ ./test.sh
```

Der Test erzeugt folgende Ausgabe:

```log
Start: 2017-06-29 23:54:06+02:00
temp=34.2'C
temp=47.1'C
temp=47.1'C
temp=47.1'C
temp=47.6'C
temp=47.1'C
temp=47.1'C
temp=47.6'C
temp=47.6'C
temp=47.1'C
temp=47.6'C
Ende: 2017-06-30 02:25:43+02:00
```

### Test 4

**Konfiguration:** Mit Gehäuse, Mit Kühlkörper

Skript ausführen:

```shell
pi@printerserver:~ $ ./test.sh
```

Der Test erzeugt folgende Ausgabe:

```log
Start: 2017-06-30 20:51:07+02:00
temp=35.2'C
temp=53.0'C
temp=54.1'C
temp=54.1'C
temp=54.6'C
temp=54.1'C
temp=54.1'C
temp=54.1'C
temp=54.1'C
temp=54.6'C
temp=54.6'C
Ende: 2017-06-30 23:22:49+02:00
```

## Fazit

So wie ich diesen Ergebnissen entnehmen kann, scheint die maximale Temparatur 48.7'C (laut internen Sensor) zu sein. Ich nehme an, dass beim Raspberry Pi Zero eine frühere Grenze eingebaut wurde, bei dem die Leistung runtergeregelt wird.

Zwar habe ich damit noch keine Langzeittests, aber es ist schon sehr beruhigent, zu sehen, dass auch nach mehreren Stunden unter Volllast die eingebaute Drosselung anscheinend ihren Job erledigt und das Raspberry Pi Zero nicht überhitzt.
