---
layout: post
comments: true
title:  "SQL: Vorherige und nachfolgende Datenbank-Einträge selektieren"
description: "Manchmal braucht man X-Einträge vor und/oder nach einem Datenbank-Eintrag. Wie das mit SQL geht, wird in diesem Beitag gezeigt."
date:   2020-09-17 11:00:00 +0200
category: Entwicklung
hideToc: true
tags: [sql, select, statement, praxis, postgresql, dev, archiv]
---

In der Praxis kommt es hin und wieder vor, dass man zu einem Eintrag X vorherige und/oder nachfolgende Einträge benötigt.

Beispiele dafür könnten folgende User-Stories sein:

- In einer Tabelle sind für die Lebzeit eines Gebäudes alle geplanten Wartungen eingetragen. Als Gebäudeverwalter möchte ich im Dashboard zu dem Gebäude immer die nächsten 3 fälligen Wartungen sehen.
- In einer Tabelle sind alle bisher durchgeführten Reparaturen eines Autos erfasst. Als Nutzer möchte ich in einem Dashboard/Report immer die letzten 5 Wartungen sehen.
- Eine Tabelle beinhaltet alle veröffentlichte PC-Spiele. Als Nutzer möchte ich zu jedem Spiel, das Spiel, die vorherige und die 2 nachher veröffentlichten Spiele angezeigt bekommen.

Sehr häufig habe ich bisher die Aufgabe aufgeteilt. Ich habe per `SQL` die Daten so weit wie notwendig eingegrenzt und sortiert und in einer Programmiersprache, dann die Logik für X vorherige/nachfolgende Einträge gebaut.

Diese Aufgabe lässt sich auch komplett im `SQL` lösen und genau darum geht es auch in diesem Beitrag.

<!--more-->

{{< archiv-note >}}

In diesem Beitrag zeige ich die Lösungen für alle User-Stories anhand der Daten der dritten User-Story.

Nehmen wir an, dass wir die folgende Tabelle mit den Datensätzen haben:

```sql
CREATE TABLE game (
    id                 int8        not null,
    name               varchar(255) not null,
    published_date     timestamp,
    primary key (id)
);

CREATE SEQUENCE game_seq START 1 INCREMENT 1 OWNED BY game.id;

-- Aus der Liste https://de.wikipedia.org/wiki/Liste_der_erfolgreichsten_Computerspiele, Stand: 17.09.2020
INSERT INTO game (id, name, published_date)
VALUES
    (nextval('game_seq'), 'Tetris', TO_TIMESTAMP('06.06.1984', 'DD.MM.YYYY')),
    (nextval('game_seq'), 'Minecraft', TO_TIMESTAMP('18.11.2011', 'DD.MM.YYYY')),
    (nextval('game_seq'), 'Grand Theft Auto V', TO_TIMESTAMP('17.09.2013', 'DD.MM.YYYY')),
    (nextval('game_seq'), 'Wii Sports', TO_TIMESTAMP('19.11.2006', 'DD.MM.YYYY')),
    (nextval('game_seq'), 'PlayerUnknown’s Battlegrounds', TO_TIMESTAMP('20.12.2017', 'DD.MM.YYYY')),
    (nextval('game_seq'), 'Overwatch', TO_TIMESTAMP('24.05.2016', 'DD.MM.YYYY'));
```

Die Tabelle beinhaltet folgende Veröffentlichungen, sortiert aufsteigend nach Veröffentlichungsdatum.

```sql
SELECT * FROM game g ORDER BY g.published_date;
```

Ausführen mit [SQL Fiddle](http://sqlfiddle.com/#!17/442d6/17)

Ergebnis

| id |                          name |       published_date |
|----|-------------------------------|----------------------|
|  1 |                        Tetris | 1984-06-06T00:00:00Z |
|  4 |                    Wii Sports | 2006-11-19T00:00:00Z |
|  2 |                     Minecraft | 2011-11-18T00:00:00Z |
|  3 |            Grand Theft Auto V | 2013-09-17T00:00:00Z |
|  6 |                     Overwatch | 2016-05-24T00:00:00Z |
|  5 | PlayerUnknown’s Battlegrounds | 2017-12-20T00:00:00Z |

Damit geht es jetzt in die User-Stories.

### User-Story 1: Die nächsten 3 Einträge

Wir möchten die 3 nachfolgenden Spiel-Veröffentlichungen nach `Wii Sports` haben:

```sql
with games as (
    select row_number() over (order by gs.published_date) as row, gs.* from game gs
),
my_game as (
    select m.row from games m where m.name = 'Wii Sports'
)
select * from games g where g.row > (select row from my_game) and g.row <= (select row +3 from my_game);
```

Ausführen mit [SQL Fiddle](http://sqlfiddle.com/#!17/442d6/6/0)

Ergebnis

| row | id |               name |       published_date |
|-----|----|--------------------|----------------------|
|   3 |  2 |          Minecraft | 2011-11-18T00:00:00Z |
|   4 |  3 | Grand Theft Auto V | 2013-09-17T00:00:00Z |
|   5 |  6 |          Overwatch | 2016-05-24T00:00:00Z |

### User-Story 2: Die 5 vorherigen Einträge

Wir möchten die 5 Spiel-Veröffentlichungen vor `PlayerUnknown’s Battlegrounds` haben:

```sql
with games as (
    select row_number() over (order by gs.published_date) as row, gs.* from game gs
),
my_game as (
    select m.row from games m where m.name = 'PlayerUnknown’s Battlegrounds'
)
select * from games g where g.row >= (select row-5 from my_game) and g.row < (select row from my_game);
 ```

Ausführen mit [SQL Fiddle](http://sqlfiddle.com/#!17/442d6/8/0)

Ergebnis

| row | id |               name |       published_date |
|-----|----|--------------------|----------------------|
|   1 |  1 |             Tetris | 1984-06-06T00:00:00Z |
|   2 |  4 |         Wii Sports | 2006-11-19T00:00:00Z |
|   3 |  2 |          Minecraft | 2011-11-18T00:00:00Z |
|   4 |  3 | Grand Theft Auto V | 2013-09-17T00:00:00Z |
|   5 |  6 |          Overwatch | 2016-05-24T00:00:00Z |

### User-Story 3: Das Spiel mit der vorherigen und 2 nachfolgenden Veröffentlichungen

Wir möchten das Spiel `Minecraft` mit der vorherigen und 2 nachfolgenden Spiel-Veröffentlichungen haben:

```sql
with games as (
    select row_number() over (order by gs.published_date) as row, gs.* from game gs
),
my_game as (
    select m.row from games m where m.name = 'Minecraft'
)
select * from games g where g.row >= (select row-1 from my_game) and g.row <= (select row+2 from my_game);
```

Ausführen mit [SQL-Fiddle](http://sqlfiddle.com/#!17/442d6/15/0)

Ergebnis

| row | id |               name |       published_date |
|-----|----|--------------------|----------------------|
|   2 |  4 |         Wii Sports | 2006-11-19T00:00:00Z |
|   3 |  2 |          Minecraft | 2011-11-18T00:00:00Z |
|   4 |  3 | Grand Theft Auto V | 2013-09-17T00:00:00Z |
|   5 |  6 |          Overwatch | 2016-05-24T00:00:00Z |

### Fazit

Wie man sehr gut sehen kann, kann man diese Anforderungen auch im reinen `SQL` lösen.

Es kann durchaus sein, dass man die Statements noch optimieren kann und ich habe auch nicht getestet, ob diese Lösung effizienter ist als die Mischform von `SQL` und einer Schleife in einer Programmiersprache.

Mir ging es in erster Linie darum, zu zeigen, dass man mit `SQL` nicht nur filtern und sortieren kann, sondern wenn man es geschickt kombiniert auch mit Folgen von Einträgen arbeiten kann.

---
Quellen

Keine
