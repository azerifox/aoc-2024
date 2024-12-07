# Advent of Code 2024 - Unix Edition

## Kick-off
Advent of Code 2024 hat begonnen und damit wieder der Ruf nach Weiterentwicklung der eigenen Fähigkeiten. Eric Wastl liefert jedes Jahr eine großartige Plattform, um mit festlicher Motivation und Spaß der eigenen (E)Xpertise etwas Gutes zu tun. Advent of Code ist, was man aus ihm macht. Wer sich mit anderen ein Punkterennen um die am schnellsten gefundene Lösung liefern möchte, findet im privaten (oder globalen) Leaderboard eine spaßige Herausforderung mit einer Prise Wettbewerbsnervenkitzel. Für mich eine nette Nebenquest die gerne mit abfallen darf, aber nicht das ganze Potential von AoC. Es eignet sich z.B. hervorragend zum Lernen einer Programmiersprache, die man dem eigenen Repertoire hinzufügen will, weit abseits von HelloWorld!-Minimalbeispielen oder der berüchtigten Tutorial-Hölle. Ein dedizierter Fokus auf Laufzeiteffiziente Lösungen kann auch ein persönliches Ziel sein. Dabei rückt vielleicht die ein oder andere bessere Methode in den Vordergrund, die ggf. unbedacht ineffiziente Gewohnheiten aus dem gefestigten Repertoire ablösen. Die Möglichkeiten sind vielfältig und die Plattform ein fantastischer Lernkatalysator.

Ich habe zahlreiche Ideen, was ich aus dem diesjährigen Advent of Code für mich machen könnte. Leider werde ich dieses Jahr durch Familie und andere kurzfristige Dezember-Prioritäten nur sehr wenig Zeit für Advent of Code haben. Das finde ich schon jetzt sehr schade, denn Nachholen ist einfach nicht dasselbe. Wenigstens den Auftakt von AoC lasse ich mir aber nicht nehmen und habe meine diesjährige Waffe gewählt: **shell + unix utilities**.

Inspiriert durch Gary Bernhardts fantastischen Vortrag ["The Unix Chainsaw"](https://www.youtube.com/watch?v=sCZJblyT_XM) springen dann wenigstens ein paar Fingerübungen bei raus, die meine Flexibilität im Terminal etwas erhöhen. Zumindest für ein paar der leichten, kleinen Aufgaben. Übrigens, aus dem Vortrag stammt mein Lieblings-Pragmatismus-Zitat:

> _Half-assed is OK when you only need half of an ass_

In diesem Sinne, hier der Auftakt: [Tag 1 von Advent of Code](https://adventofcode.com/2024/day/1).

### Tag 1

Aufgabe verstanden. Linke und rechte Spalte jeweils in aufsteigender Reihenfolge numerisch sortieren und den Betrag der Differenz pro Zeile aufaddieren. Ein typischer erster Tag, noch schön einfach.

Und wie zaubern wir uns das Ergebnis im Terminal auf stdout? Ganz nach der [Unix Philosophie](https://en.wikipedia.org/wiki/Unix_philosophy) durch die richtige Kombination von mehreren kleinen Programmen, die eine spezialisierte Aufgabe besonders gut können. Und aus Teamarbeit einiger Xperten wird dann etwas Gutes. Kennen wir ja.

Vor allem aber: hübsch der Reihe nach. Wie bei einem guten Rezept folgt eine Zutat der anderen und am Ende freuen wir uns über das Ergebnis.

Erstmal den (Beispiel-)Input unter `aoc24-01.txt` abspeichern und aufs Terminal damit.

```console
azerifox@Defiant aoc-2024 % cat aoc24-01.txt
3   4
4   3
2   5
1   3
3   9
3   3
```

Um die Listen einzeln zu sortieren und dann wieder nebeneinanderlegen zu können, brauchen wir sie auch getrennt.

```console
azerifox@Defiant aoc-2024 % cut -d ' ' -f1 < aoc24-01.txt
3
4
2
1
3
3
```

`-f[n]` für die n. Spalte. Für Spalte zwei also `-f2`?

```console
azerifox@Defiant aoc-2024 % cut -d ' ' -f2 < aoc24-01.txt






```

Der Delimiter `' '` führt leider bei mehreren Leerzeichen zwischen den Listen zu leeren Spalten. Es ist auch nur ein Zeichen als Delimiter erlaubt. Wir könnten jetzt mit `-f4` arbeiten, aber dann juckts in irgendeiner Synapse so unangenehm. Wir sind ja keine Unmenschen. Also z.B. mit `tr` viele Leerzeichen auf eins trimmen.

```console
azerifox@Defiant aoc-2024 % cat aoc24-01.txt | tr -s ' '
3 4
4 3
2 5
1 3
3 9
3 3
```

Dann bekommen wir unsere Spalte ja doch über `-f2`.

```console
azerifox@Defiant aoc-2024 % cat aoc24-01.txt | tr -s ' ' | cut -d ' ' -f2
4
3
5
3
9
3
```

Prima! Dann können wir die auch gleich mal numerisch sortieren, um zu sehen, wie das geht.

```console
azerifox@Defiant aoc-2024 % cat aoc24-01.txt | tr -s ' ' | cut -d ' ' -f2 | sort -n
3
3
3
4
5
9
```

Und wie bekommen wir die zwei Listen sortiert wieder nebeneinander? Zwischenspeichern 'güldet' nicht. Mit `paste` lassen sich Zeilen von zwei Dateien kombinieren. Da wir keine zwischengespeicherten Dateien für die Listen haben, die als Argumente erwartet werden würden, greifen wir noch auf [Process Substitution](https://en.wikipedia.org/wiki/Process_substitution) zurück. Aus der Vogelperspektive sowas wie `paste <(sortierte Liste links) <(sortierte Liste rechts)`. Na dann puzzeln wir mal zusammen.

```console
azerifox@Defiant aoc-2024 % paste <(cat aoc24-01.txt | tr -s ' ' | cut -d ' ' -f1 | sort -n) <(cat aoc24-01.txt | tr -s ' ' | cut -d ' ' -f2 | sort -n)
1       3
2       3
3       3
3       4
3       5
4       9
```

Im Ganzen gesehen wird mir das aber nun ein wenig zu unübersichtlich. Die anfänglichen Überlegungen zum trimmen mehrerer Leerzeichen nochmal überdacht, geht es auch einfacher. `awk` trennt per default nach Leerzeichen, egal wie viele sich da tummeln. Die zweite Spalte kann man also auch einfach damit aufs terminal 'drucken'.

```console
azerifox@Defiant aoc-2024 % cat aoc24-01.txt | awk '{ print $2 }'
4
3
5
3
9
3
```

Wenn man das ein bisschen komprimiert und statt zu pipen einfach den Input über `awk`s Dateiargument übergibt, dann sieht das schon übersichtlicher aus.

```console
azerifox@Defiant aoc-2024 % paste <(awk '{print $1}' aoc24-01.txt | sort -n) <(awk '{print $2}' aoc24-01.txt | sort -n)
1       3
2       3
3       3
3       4
3       5
4       9
```

Wo wir schon bei `awk` sind, lässt sich damit auch wunderbar die positive Differenz errechnen, nämlich z.B. über

```awk
awk '{ print ($1 > $2) ? $1 - $2 : $2 - $1 }'
```

Vergleiche, elementare Rechenoperationen und den [Ternary Operator](https://en.wikipedia.org/wiki/Ternary_conditional_operator) sind wir ja aus anderen Sprachen gewohnt. Stöpseln wir das noch hinten dran, haben wir unsere Differenzen.

```console
azerifox@Defiant aoc-2024 % paste <(awk '{print $1}' aoc24-01.txt | sort -n) <(awk '{print $2}' aoc24-01.txt | sort -n) | awk '{ print ($1 > $2) ? $1 - $2 : $2 - $1 }'
2
1
0
1
2
5
```

Jetzt noch aufsummieren. Eine neue pipe gar nicht nötig, statt im letzten `awk` Befehl zu `print`en, summieren wir lieber. Aus

```awk
awk '{ print ($1 > $2) ? $1 - $2 : $2 - $1 }'
```

wird

```awk
awk '{ sum += ($1 > $2) ? $1 - $2 : $2 - $1 }'
```

Aktuell wird so aber noch nichts angezeigt. Die Summe wird zwar berechnet, aber ohne `print` nichts mehr ausgegeben. Wir wollen die Summe auch nur ganz am Ende einmal ausgeben und nicht jedes Zwischenergebnis. Also noch einen `END` Block definieren:

```awk
awk '{ sum += ($1 > $2) ? $1 - $2 : $2 - $1 } END { print sum }'
```

Und Tadaa!

```console
azerifox@Defiant aoc-2024 % paste <(awk '{print $1}' aoc24-01.txt | sort -n) <(awk '{print $2}' aoc24-01.txt | sort -n) | awk '{ sum += ($1 > $2) ? $1 - $2 : $2 - $1 } END { print sum }'
11
```

Das war eine gute Übung!

Sollte jemand neue Motivation und vielleicht sogar Inspiration gefunden haben, aus Advent of Code etwas persönliches für sich rauszuholen, dann habe ich mein Ziel erfüllt. Wir freuen uns über neue Gesichter im #AdventOfCode channel.

Zweiter Teil (und folgende) nur als Lösung:

awk mit ausgelagertem awk-Programm, weil inline zu komplex: `awk -f part2.awk < aoc24-01.txt`

part2.awk:
```awk
{
    # Track frequency per input from right list
    freq[$2]++
    # Keep left list as array
    left[NR] = $1
}
END {
    sum = 0
    for(i = 1; i <= NR; i++) {
        num = left[i]
        if(num in freq) {
            sum += num * freq[num]
        }
    }
    print sum
}
```

### Tag 2:

```shell
cat 02.txt | ./day02.sh | grep -E  "^(([1-3]\s?)+|(-[1-3]\s?)+)$" | wc -l
```

day02.sh:
```bash
#!/bin/bash

while read -r line; do
    prev=""
    diffs=""
    for num in $line; do
        if [ ! -z $prev ]; then
            diffs+="$(($num - $prev)) "
        fi
        prev=$num
    done
    echo "$diffs"
done
```
