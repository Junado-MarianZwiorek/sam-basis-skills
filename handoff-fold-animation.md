# Handoff v3: Fold-Animation „iPhone Duo“ für den eigenen Launcher

> **v3 ersetzt alle früheren Fassungen vollständig.** v2.x hatte die Lage des Knicks am Cover falsch
> (Knick rechts angenommen). Neu gemessen: **Der Knick des Covers liegt links, die Kamera-Loch-Seite rechts
> ist die freie Außenkante.** Daraus folgen andere Blur-Richtungen und vor allem der Parallaxe-Effekt
> (Abschnitt 3), den v2 übersehen hat. Ältere Versionen sind in der Git-Historie.

Quellen: **Video 1** (Felixba, 1432×1080, ~40 fps, frontal gefilmt, dunkles Wallpaper), **Video 2** (MKBHD-Short,
schräg von oben, helles Wallpaper). Markierung: **[gemessen]** = per Tracking/Raster ausgewertet,
**[beobachtet]** = sichtbar, nicht vermessen, **[abgeleitet]** = Parametrisierung.

---

## 0. Grundregeln

1. **Alles ist eine stetige Funktion des Klappwinkels θ** (0° zu, 180° flach). Keine Timer, keine Mindestdauern,
   kein Nachlauf. Gerät anhalten → Bild steht. Zurückdrehen → läuft rückwärts. Einzige Zeitkomponente: Low-Pass
   auf θ (30–50 ms) gegen Sensorrauschen.
2. **Alle Richtungen sind relativ zum Knick** angegeben. Koordinate auf dem Cover: `g` = 0 an der Knickkante,
   1 an der freien Außenkante.

---

## 1. Geometrie [gemessen, Video 1 Frames 137–178, Video 2 Frames 52–100]

- Zugeklappt liegt die **linke Hälfte** umgeklappt auf der rechten. Ihre Rückseite ist der Cover-Screen.
- **Knick = linke Kante des Covers.** Beim Öffnen bleibt sie stehen (Video 1: x ≈ 635 → 605 px, nur Handdrift).
- **Freie Außenkante = rechte Kante des Covers, mit dem Kameraloch oben rechts.** Beim Öffnen hebt sie sich zum
  Betrachter und schwenkt nach links über den Knick (Video 1: x ≈ 1068 → 863 px bis Frame 164, Kante bei ~170
  hochkant, danach liegt das Panel links).
- **Rechts neben der Außenkante wird das Innen-Panel frei.** Es liegt genau unter dem Cover und zeigt dieselbe
  Komposition (Widgets, Raster, Dock rechts, Statusleiste oben rechts) pixelgleich. Statusleiste und Dock des
  Innen-Panels erscheinen also scharf direkt rechts neben der Außenkante des Covers.
- Video 2 zeigt dieselbe Geometrie (linke Cover-Kante steht, Kamera-Loch-Kante schwenkt, Innen-Panel rechts daneben).

---

## 2. Cover-Screen: Blur und Helligkeit

### 2.1 Öffnen [beobachtet, beide Videos]
- **Blur beginnt an der Außenkante (rechts) und läuft zum Knick (links).** Uhr-Widget, Dock und rechte Spalten
  werden zuerst unscharf, Wetter-Widget und linke Spalte (WhatsApp/Fotos/Telefon) zuletzt.
- Es gibt **keine Ausnahmen**: Statusleiste, Dock, Badges, Page-Dots, Wallpaper blurren alle mit.
- Video 1: Blur startet im ersten Frame mit sichtbarem Spalt, voll unscharf bei ~30–35°. Video 2: schneller.
- Helligkeit: Video 1 (dunkles Wallpaper) wird zur Außenkante hin dunkler. Video 2 (helles Wallpaper) wird zur
  Außenkante hin heller und ausgewaschen. Beides folgt dem **Wallpaper, das durch die Parallaxe (Abschnitt 3) an
  der Außenkante übrig bleibt**, wenn die Icons dort hinausgleiten. Kein eigener Weiß-Overlay belegbar.

### 2.2 Zuklappen [beobachtet, Video 1 Frames 276–320]
- **Schärfe beginnt am Knick (links) und läuft zur Außenkante (rechts).** Wetter-Widget und WhatsApp werden
  zuerst scharf, Uhr und Dock zuletzt.

### 2.3 Formeln [abgeleitet]
```
// g = 0 Knick, 1 Außenkante. BLUR_MAX ≈ 28 dp bei ~400 dp Cover-Breite (σ ≈ 3,5 % der Breite).
// Öffnen:
q        = clamp((θ - 2) / 38, 0, 1)
strength = 0.3 + 0.7 * q
front    = 1 - 1.35 * q                         // Front läuft von g=1 (außen) nach g<0 (über den Knick hinaus)
blur(g)  = BLUR_MAX * strength * smoothstep(front - 0.35, front, g)   // unscharf außen von der Front
// Zuklappen (Schärfe-Front vom Knick nach außen über 25° → 0°):
s        = 1 - smoothstep(0, 25, θ)             // 0 bei ≥25°, 1 bei 0°
front    = -0.35 + 1.35 * s
blur(g)  = BLUR_MAX * smoothstep(front, front + 0.35, g)              // scharf innen von der Front
dim      = 0.15 * smoothstep(3, 40, θ)         // leichte globale Abdunklung, keine Weiß-Überlagerung
```
Prüfwerte: Öffnen θ=2° → überall scharf, θ=40° → überall voll unscharf, bei θ=15° rechts unscharf, links scharf.
Zuklappen θ=25° → überall unscharf, θ=0° → überall scharf, bei θ=10° links scharf, rechts unscharf.

---

## 3. Parallaxe des Cover-Inhalts [gemessen, Video 1] – der Kern des Effekts

### 3.1 Befund
Gemessen wurde die Lage der Icon-Spalten **relativ zum Cover-Rahmen** (Knickkante und Außenkante pro Frame
abgelesen, Spalten-Schwerpunkte per Raster). Bei starrem Bild müssten die Spalten ihren Anteil `g = f` behalten.
Normale Perspektive (Außenkante kommt näher zur Kamera) würde sie sogar leicht **zum Knick** schieben.

Gemessen (Anteil g von der Knickkante, Video 1, Öffnen):

| Frame | Cover-Breite rel. | WhatsApp (f 0.11) | Formel 1 (f 0.29) | Kamera (f 0.48) | Karten (f 0.66) | Dock (f 0.87) |
|---|---|---|---|---|---|---|
| 137 zu | 1.00 | 0.11 | 0.29 | 0.48 | 0.66 | 0.87 |
| 152 | 0.93 | 0.13 | 0.31 | 0.49 | 0.68 | 0.88 |
| 158 | 0.86 | 0.12 | 0.31 | 0.51 | 0.70 | nicht mehr sichtbar |
| 160 | 0.80 | 0.13 | 0.32 | 0.53 | 0.72 | – |
| 162 | 0.72 | 0.13 | 0.34 | 0.57 | – | – |
| 164 | 0.61 | 0.13 | 0.37 | – | – | – |

**Die Spalten wandern relativ zum Rahmen zur Außenkante, umso stärker, je weiter sie vom Knick weg sind und je
weiter das Cover gedreht ist.** Die Spalte am Knick bleibt stehen. Die äußerste Spalte (Dock) gleitet unter der
Außenkante hinaus und ist ab ~Frame 158 vom Cover verschwunden. Beim Zuklappen (Frames 285–288) zeigen alle
Spalten dieselbe Richtung, kleinere Winkel, geringere Messsicherheit.

### 3.2 Deutung
Der Inhalt bleibt **teilweise im Raum stehen**, während das Glas wegdreht. Rechnerisch bewegt sich das Bild
etwa **40–50 % weniger als das Panel** (α ≈ 0,4 gemessen ohne Perspektivkorrektur; mit Korrektur eher 0,45–0,5).
Es wirkt, als läge das Bild auf der Ebene des Innen-Displays darunter und würde auf das kippende Cover projiziert.
Weil gleichzeitig rechts neben der Außenkante das echte Innen-Panel mit identischem Layout frei wird, schließt
der Cover-Inhalt fast nahtlos an das Innen-Display an. **Das ist der „Projektions“- bzw. Parallaxe-Eindruck.**

### 3.3 Formel [abgeleitet]
```
// Für jedes Pixel des Covers mit Anteil g (0 Knick, 1 Außenkante) wird der Inhalt an f statt an g gesampelt:
α    = 0.45
θc   = min(θ, 75°)
s(θ) = 1 + α * (1 / cos(θc) - 1)       // 1 bei 0°, 1.45 bei 60°, ~2.3 bei 75°
f    = g / s(θ)                         // Pivot = Knickkante; Inhalt rutscht zur Außenkante und wird dort abgeschnitten
// vertikal: unverändert (nicht vermessen)
```
Prüfwerte: θ=0 → f=g. θ=60°, g=1 → f=0.69, d. h. das äußere Drittel (Dock, rechte Spalte) ist hinter der
Außenkante verschwunden, die Knickspalte steht. Reihenfolge: zuerst Parallaxe-Sampling, dann Blur (2.3), dann dim.

### 3.4 Abgrenzung zu „Extra 8“
Extra 8 sampelt mit `u = x·D/(cos a·D − (x − cx)·sin a)` (x vom Knick). Für die meisten x gilt `u > x`, der Inhalt
wandert also **zum Knick**. Gemessen ist die Gegenrichtung: Inhalt wandert **zur Außenkante**. Außerdem beginnt
der Effekt im Video ab dem ersten Grad, nicht erst ab 25°. Schattenbänder oben/unten sind im Video nicht zu sehen.
Vorzeichen bitte prüfen; wenn Extra 8 x von der Außenkante misst, stimmt die Richtung.

---

## 4. Innen-Screen [gemessen/beobachtet, unverändert gegenüber v2]

- **Rechtes Panel** (= Cover-Komposition): nie geblurrt, nie verschoben. Gemessen 0–2 px Abweichung zum Gehäuse
  in allen Phasen. Seine linken Spalten werden zwischen ~60° und ~130° vom hochstehenden linken Panel verdeckt
  und tauchen beim Weiteröffnen auf. Das ist Geometrie, keine Animation.
- **Linkes Panel** (Zusatz-Widgets): stark unscharf und dunkel, wird erst in den letzten ~25° vor 180° scharf.
  Video 2: Helligkeit kommt als weiche Front vom Knick nach außen (60°→160°). Widgets erscheinen an ihrer
  Endposition, kein Slide-in, kein Scale-in.
- Maskenkante am Knick wandert zwischen 60° und 110° von knapp rechts nach knapp links der Knicklinie, danach ist
  die Knickspalte des rechten Panels scharf.
```
p    = clamp((θ - 120) / 60, 0, 1);  e = p^3
blurL(x) = BLUR_MAX_INNER * (1 - e)                 // BLUR_MAX_INNER ≈ 48 dp
lf   = 1.3 * clamp((θ - 60) / 100, 0, 1)
dimL(x)  = 0.6 * smoothstep(lf - 0.25, lf + 0.05, x) + 0.1 * (1 - e)   // x = 0 Knick, 1 Außenkante links
maskEdge = hingeX + iconW * (0.5 - clamp((θ - 60) / 50, 0, 1))
```
Zuklappen innen: dieselben Formeln (reine Funktion von θ).

---

## 5. Was nicht passiert
- Keine Skalierung, kein Slide-in, kein Layoutwechsel zwischen Cover und Innen.
- Keine Bewegung auf dem Innen-Screen. Die einzige Inhaltsbewegung ist die Parallaxe auf dem Cover (Abschnitt 3).
- Keine Schattenbänder, keine Knicklinie, kein Weiß-Overlay.
- Keine Timer.

---

## 6. Abnahme-Checkliste
- [ ] Öffnen: Cover-Knickkante steht, Außenkante (Kameraloch-Seite) schwenkt zum Knick.
- [ ] Öffnen: Blur beginnt an der Außenkante, Knickspalte bleibt am längsten scharf, bei 40° alles unscharf.
- [ ] Öffnen: Icons gleiten relativ zum Cover-Rahmen zur Außenkante. Knickspalte steht, Dock verschwindet unter
      der Außenkante. Bei 60° ist das äußere Drittel weg.
- [ ] Die Inhalte nahe der Außenkante schließen optisch an das daneben frei werdende Innen-Panel an.
- [ ] Zuklappen: Schärfe beginnt am Knick, Außenkante zuletzt, Parallaxe läuft rückwärts.
- [ ] Innen rechts: nie unscharf, nie verschoben. Innen links: wird erst in den letzten 25° scharf.
- [ ] Anhalten bei beliebigem Winkel friert alles ein.

---

## 7. Messgrenzen
- Kanten und Spalten wurden manuell im 10-px-Raster abgelesen, Genauigkeit ±5 px, das sind ±0,01–0,02 im Anteil g.
  Der Parallaxe-Trend (bis +0,09 bei 160–164) liegt deutlich darüber.
- Die Außenkante wurde an der Mitte der hellen Metallkante gemessen. Deren sichtbare Breite wächst beim Drehen
  nach außen. Das unterschätzt g eher, der echte Effekt ist also eher größer.
- Perspektive ist nicht herausgerechnet. Sie wirkt in die Gegenrichtung, also ist α ≥ 0,4.
- Vertikale Parallaxe wurde nicht vermessen.
- Zuklappen: gleiche Richtung, weniger Frames, geringere Sicherheit.
