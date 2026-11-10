# Gestenlerner: Datenmodell und Lernlogik

Prototyp: `index.html` im Browser öffnen (am besten am Handy oder mit Touch-Emulation). Keine Installation, keine Abhängigkeiten.

## 1. Datenmodell

```ts
type Word = {
  id: string;
  lang: "la" | "en" | "fr" | "es";
  target: string;          // Zielsprache, mit Makrons: "perīculum"
  native: string;          // Muttersprache: "die Gefahr"
  chunks: string[];        // Wortteile für die Chunk-Übung: ["perī","cu","lum"]
  gesture: Point[];        // Polylinie, Koordinaten 0..100: [[60,10],[35,50],...]
  image?: string;          // Illustration unter der Geste (optional)
  audio?: string;          // Aussprache (optional)
};

type List = { id: string; title: string; wordIds: string[]; source: "scan" | "typed" | "book" };

type Progress = {          // pro Nutzer und Wort
  wordId: string;
  stage: 0 | 1 | 2 | 3 | 4;   // neu → lernend → sicher → fest → gemeistert
  dueAt: number;               // nächster Wiederholungszeitpunkt (ms)
  correct: number; wrong: number;
  lastSeenAt: number;
};

type Plan = { listIds: string[]; targetDate: number; dailyGoal: number };  // max. 5 Listen aktiv
```

## 2. Übungstypen

| Typ | Richtung | Eingabe | Im Prototyp |
|---|---|---|---|
| `gesture` | Einführung | Pfad nachfahren | `gest()` |
| `mc` | Ziel → Muttersprache | 1 aus 4 antippen, Ablenker aus derselben Runde | `mc()` |
| `pairs` | beide | Paare verbinden | `pairs()` |
| `spell-chunks` | Mutter → Ziel | Wortteile in Reihenfolge | `spell(m=1)` |
| `spell-letters` | Mutter → Ziel | Buchstaben-Kacheln (eindeutige Buchstaben, Makrons eigene Kacheln) | `spell(m=0)` |
| `grid-trace` | Mutter → Ziel | 5×5-Raster, orthogonaler Pfad, Zufallsbuchstaben als Ablenker | `grid()` |

Der Tipp-Button ist bei `spell-*` und `grid-trace` aktiv.

## 3. Empfohlene Sequenz pro Wort (Hypothese)

`gesture` → `mc` / `pairs` (Erkennen) → `spell-chunks` → `spell-letters` → `grid-trace` (Produzieren)

Welche Übung für welche Stufe gilt, ist in der Vorlage nicht belegt. Vorschlag:

| Stufe | Übungen |
|---|---|
| 0 neu | gesture, mc |
| 1 lernend | mc, pairs, spell-chunks |
| 2 sicher | spell-letters |
| 3 fest | grid-trace |
| 4 gemeistert | nur noch selten im Test |

## 4. Wiederholungsplan (Spaced Repetition)

Eigenes Verfahren, weil das Original nicht öffentlich ist (Leitner-Schema):

```
richtig  → stage = min(stage+1, 4);  dueAt = jetzt + [0, 1, 3, 7, 21][stage] Tage
falsch   → stage = max(stage-1, 0);  dueAt = jetzt (noch in dieser Sitzung)
```

Tagesziel rückwärts vom Testtermin: `dailyGoal = ceil(offeneWörter / verbleibendeTage)`. Schwache Wörter (viele Fehler) kommen vorrangig dran.

## 5. Modi

Smart (Tagespaket nach Plan), Sprint (ca. 5 Min), Marathon (alle fälligen), Test (ohne Tipps, bewertet), Karten (Aufdecken, selbst bewerten).

## 6. Bekannte Lücken des Prototyps

- Gestenprüfung prüft nur die Abdeckung des Pfads (92 %), nicht die Zeichenrichtung.
- Keine Persistenz, kein Audio, keine Bilder, kein Fehler-Feedback mit Lösung.
- Die Gesten sind handgezeichnete Platzhalter. Für ein echtes Produkt braucht jedes Wort eine passende Geste samt Illustration.
- Wortlisten und Gesten sind fest im Code. Das Datenmodell oben zeigt das Zielformat.
