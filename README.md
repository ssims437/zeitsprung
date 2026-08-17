# Zeitsprung

Zeitzonen und Sommerzeit **gemessen statt nachgeschlagen**. Die Umstellungen werden
sekundengenau gesucht — mit dem Zeitzonenwissen, das jeder Browser ohnehin mitbringt.
Keine Datenbank, keine Bibliothek, keine gepflegte Regelliste.

### → [Öffnen](https://ssims437.github.io/zeitsprung/)

Eine einzelne HTML-Datei. Kein Build, keine Abhängigkeit, nichts verlässt den Browser.

---

## Was drin ist

| | |
|---|---|
| **Umstellungen** | jeder Sprung eines Jahres, sekundengenau, mit Ortszeit davor und danach |
| **Jahreskalender** | jeder Tag eingefärbt nach seiner Länge — 23, 24 oder 25 Stunden |
| **Gibt es diese Uhrzeit?** | eine Ortszeit eingeben und sehen, ob sie **fehlt**, **eindeutig** ist oder **zweimal** existiert |
| **Selbstprüfung** | 252 Prüfungen über zehn Zonen und vier Jahre |

## Der Trick

`Intl.DateTimeFormat` kennt die Zeitzonendatenbank. Liest man **dieselbe Zeitmarke** einmal
als Wanduhrzeit einer Zone und rechnet sie zurück nach UTC, ist die Differenz der Versatz:

```js
const w = new Intl.DateTimeFormat("en-US", { timeZone: zone, /* … */ }).formatToParts(t);
const versatz = (Date.UTC(w.jahr, w.monat - 1, w.tag, w.stunde, w.minute, w.sekunde) - t) / 60000;
```

Dann in Tagesschritten durchs Jahr gehen, und wo sich der Versatz ändert, den Zeitpunkt
**binär einschachteln** bis auf die Sekunde. Das findet auch Zonen mit drei Sprüngen in
einem Jahr — und Sprünge, die nichts mit Sommerzeit zu tun haben.

## Was dabei herauskommt

**Wien 2026** — der erwartete Fall:

| Umstellung (UTC) | Ortszeit davor | Ortszeit danach | Folge |
|---|---|---|---|
| 2026-03-29 01:00:00Z | 01:59:59 | 03:00:00 | 60 Minuten fehlen |
| 2026-10-25 01:00:00Z | 02:59:59 | 02:00:00 | 60 Minuten doppelt |

**Lord Howe Island** stellt um **30 Minuten** um, nicht um eine Stunde:
`01:59:59 → 01:30:00`. Wer mit „plus oder minus eine Stunde" rechnet, liegt hier daneben.

**Samoa 2011** ist der Fall, den man gesehen haben muss: **drei** Umstellungen, und am
30. Dezember springt die Zone von −10:00 auf **+14:00**. Es fehlen **1440 Minuten** — der
30. Dezember 2011 hat auf Samoa nie stattgefunden. Das Jahr hat dort **8736 statt 8760
Stunden**, also 364 Tage.

**São Paulo 2018** stellte genau um **Mitternacht** um: `23:59:59 → 23:00:00` im Februar,
`23:59:59 → 01:00:00` im November. In der zweiten Nacht existierte der Tagesbeginn nicht —
wer Tage über „00:00 Ortszeit" definiert, verliert dort einen ganzen Tag.

**Chatham** liegt bei **+12:45**, Nepal bei **+5:45**, St. John's bei **−3:30**. Versätze
sind Vielfache von 15 Minuten, nicht von Stunden.

## Beweis statt Behauptung

```
252 Prüfungen, kein einziger Fehler · 0,8 s
```

| Was | Umfang | Kriterium |
|---|---|---|
| Umstellungen sekundengenau | 46 Funde in 10 Zonen | eine Sekunde davor gilt noch der alte Versatz, in der Sekunde selbst der neue |
| Tage ergeben das Jahr | 40 Zonenjahre | Summe aller Tageslängen = Jahreslänge, auf die Millisekunde |
| Ortszeit rückwärts auffindbar | 46 Umstellungen | jede reale Ortszeit wird wiedergefunden |
| Versatz in Viertelstunden | 120 Proben | immer ein Vielfaches von 15 Minuten |

## Was mich das gekostet hat

Die Selbstprüfung war beim ersten Lauf **rot — und sie hatte recht**. Zwei echte Fehler,
beide in derselben Funktion: dem Rückweg von einer Ortszeit zur Zeitmarke.

**Das Suchfenster war zu klein.** Ich habe ±7 Stunden um die Zeitmarke abgesucht. Es gibt
Zonen bei **+14:00** (Kiritimati, Samoa seit 2011) und **−12:00**. Deren Ortszeiten lagen
außerhalb und galten damit als nicht existent. 16 Fehler.

**Das Suchraster war zu grob.** Danach lief es in Halbstundenschritten — und übersah
damit **jede Zone mit 45-Minuten-Versatz**. Chatham (+12:45) und Nepal (+5:45) meldeten
für jede reale Uhrzeit „gibt es nicht". Das ist die tückischere Variante: Der Fehler
betrifft nur Zonen, an die man beim Bauen nicht denkt, und sieht überall sonst richtig aus.
8 Fehler.

Beides fand nicht das Auge, sondern die Prüfung. Ohne sie hätte das Blatt für Wien
tadellos ausgesehen.

**Und die Zonenliste stimmt nicht mit sich selbst überein.** `Intl.supportedValuesOf`
liefert in der getesteten Fassung `Asia/Katmandu` — die alte Schreibweise. Dieselbe Engine
akzeptiert aber auch `Asia/Kathmandu`. Wer eine Zone aus einer anderen Quelle setzt, bekommt
einen leeren Auswahlwert und eine `RangeError`. Zonen werden deshalb geprüft und bei Bedarf
als Alias nachgetragen.

## Warum das praktisch zählt

Eine Ortszeit ohne Zone ist keine Zeitangabe. Zweimal im Jahr gibt es eine Uhrzeit nicht
und eine andere doppelt: Termine verschieben sich, Arbeitszeiten werden doppelt gezählt,
Logzeilen erscheinen in falscher Reihenfolge, Abrechnungen stimmen um eine Stunde nicht.
Wer in UTC speichert und erst zur Anzeige umrechnet, hat keines dieser Probleme.

## Benutzen

Datei öffnen genügt — über `file://` funktioniert alles. Wer lieber einen Server mag:

```bash
python -m http.server 8000
```

Die Auswahl kennt alle Zonen des Browsers (rund 420); die Beispielliste führt direkt zu den
lehrreichen Fällen.

## Lizenz

[MIT](LICENSE) — nimm es, zerleg es, bau was Besseres.

Alle fünfzehn Blätter, nach Feld geordnet: **[ssims437.github.io](https://ssims437.github.io/)**
