# Handoff: Fold-Animation „iPhone Duo“-Style für eigenen Launcher

> **v2.3:** Grundregel ergänzt: **Jede Animation ist eine stetige Funktion des Hinge-Winkels θ.** Keine Timer, keine Mindestdauern, keine Nachlauf-Animationen nach dem Schließen. Cover-Blur beim Öffnen läuft jetzt über 2°–40° mit wachsender Stärke (6.3). Cover-Entschärfung beim Schließen ist auf θ gemappt (5, 6.3), die im Video gemessene Zeitkomponente steht nur noch als Beobachtung.
>
> **v2.2:** Neuer Abschnitt 10 (Parallaxe / Bewegung der Apps, per Tracking gemessen) und Abschnitt 11 (weitere Details aus Video 1). Abschnitt 7 (Gyro) ist auf „niedrige Priorität, optional“ zurückgestuft.
>
> **Korrektur-Hinweis (v2.1):** Die vier `smoothstep`-Formeln in 6.3 hatten vertauschte Kanten (e0 > e1). Jetzt mit `1 - smoothstep(...)` und Prüfwerten. Verhalten laut Text war schon richtig.
>
> **Korrektur-Hinweis (v2):** Frühere Versionen dieses Dokuments behaupteten, Statusbar und Dock des
> **Cover-Screens** blieben beim Falten scharf. Das ist falsch. Die scharfen Elemente, die in den Videos neben
> dem verschwommenen Cover zu sehen sind, gehören zum **Innen-Screen** (rechtes Panel), das beim Klappen
> physisch neben dem Cover-Panel hervorschaut. **Auf dem Cover wird ausnahmslos alles unscharf.** Außerdem war
> die Richtung der Blur-Front falsch beschriftet; die Hinge-Kante des Covers ist die **rechte** Kante. Beides ist
> unten korrigiert. Falls du eine ältere Version umgesetzt hast: Abschnitte 2, 5, 6.2, 6.4 und 9 neu lesen.

> Analysegrundlage:
> **Video 1:** 15 s, 1432×1080, ~39,6 fps, Gerät in der Hand, Kamera frontal. 2× Auf-, 1× Zuklappen.
> **Video 2:** 21 s (MKBHD-Short), 1080×1708, ~29,6 fps, Person steht und filmt schräg von oben. 3× Auf-/Zuklappen.
> Beide sind Concept-Renderings mit unterschiedlichem Wallpaper/Layout, aber identischer Mechanik.

---

## 0. Aufgabe für dich (Ziel-Session)

Baue im Launcher eine **winkelgesteuerte** Übergangsanimation zwischen Cover-Screen und Innen-Screen nach,
die exakt dem unten beschriebenen Verhalten folgt. Treiber ist der **Hinge-Winkel θ** (0° = zu, 180° = flach)
aus dem Hinge-Sensor (Android: `WindowInfoTracker` / `FoldingFeature` + `Sensor.TYPE_HINGE_ANGLE`).
Zusätzlich fließt der **Gyroskop-/Gravity-Vektor** als Perspektiv-Modulator ein (Abschnitt 7).

Markierung: **[beobachtet]** = direkt im Video sichtbar. **[abgeleitet]** = Parametrisierung, die zu den Frames
passt. **[Extrapolation]** = nicht im Video, vom Auftraggeber gewünscht.

---

## 1. Geometrie und Layout-Grundprinzip [beobachtet]

### 1.1 Welches Panel ist was
- Book-Style-Foldable, Hinge vertikal. Beim Öffnen schwingt das **linke Panel** auf, das **rechte Panel** bleibt in der Hand.
- Der **Cover-Screen sitzt auf der Rückseite des linken (schwingenden) Panels**. Im geschlossenen Zustand ist der
  Hinge-Rücken (Spine) am **rechten** Rand des Covers sichtbar. Das Kameraloch des Covers sitzt oben rechts,
  also an der Hinge-Seite.
- Folge für alle Richtungsangaben unten: **Cover: Hinge-Kante = rechts, Außenkante = links.**
  **Innen: Hinge = Mitte, linkes Panel = das schwingende.**
- Während des Klappens (θ zwischen ~10° und ~170°) sind **beide Screens gleichzeitig sichtbar**: das Cover auf
  dem schwingenden Panel und daneben der rechte Teil des Innen-Screens. Das rechte Innen-Panel wird nie
  geblurrt (Abschnitt 3). Deshalb sieht man in jedem Übergangsframe ein scharfes „20:33“/WLAN und ein scharfes
  Dock **neben** dem verschwommenen Cover. Diese Elemente sind **nicht** Teil des Covers.

### 1.2 Layout
- Cover-Screen: Wetter-Widget (oben links) + Uhr-Widget (oben rechts, neben Kameraloch) + 4×4 Icon-Grid +
  vertikales Dock am rechten Rand + Statusbar (Uhrzeit, WLAN) unter dem Kameraloch. Video 2: Sleep-/Kalender-Widget
  oben, 4×4 Grid, Dock rechts, Such-Button unten rechts. Gleiche Struktur.
- Innen-Screen (Landscape, ~2:1): zeigt **exakt dieselbe Cover-Komposition pixelidentisch auf dem rechten Panel**
  (Widgets, Grid, Dock, Statusbar oben rechts). Nichts skaliert, verschiebt oder re-layoutet sich.
- Linkes Innen-Panel: zusätzliche Widget-Spalte (Video 1: Batterie, Kalender, Notizen. Video 2: Kalender,
  Erinnerungen, Batterie 97 %, Karten).
- Die **linke Icon-Spalte** des Grids liegt **auf der Knick-Linie**. Sie gehört logisch zum rechten Panel und
  bleibt scharf; die Blur-Maske beginnt links daneben (Abschnitt 3, 9.6).
- Folgerung: Cover-Layout = rechte Hälfte des Innen-Layouts. **Keine** Slide-/Scale-/Morph-Animation. Die gesamte
  Animation ist: **Blur + Helligkeit + Maskensweep**, pro Screen getrennt.

---

## 2. AUFKLAPPEN – Cover-Screen [beobachtet]

Zeitachse Video 1: Blur beginnt ~Frame 140 (θ ≈ 3–5°), voll ~Frame 158 → **~0,45 s**, θ dann ≈ 20–25°.
Video 2: Blur beginnt bei θ ≈ 2°, voll nach **~0,25 s** (θ ≈ 15°). Die Cover-Animation ist fertig, bevor
das Gerät sichtbar offen ist.

**Es gibt keine Ausnahmen: Statusbar, Dock, Widgets, Icons, Labels, Page-Dots und Wallpaper werden alle unscharf.**

Was passiert, Ecke für Ecke (Cover, Hinge rechts):

| Bereich Cover | Verhalten |
|---|---|
| **Oben rechts (Uhr-Widget, Kameraloch, Statusbar darunter)** | Wird **als erstes** unscharf, am stärksten. Zifferblatt → weißes Rechteck mit weichem Rand. Uhrzeit/WLAN der Statusbar verschwimmen mit, bis nichts mehr lesbar ist. |
| **Rechts (Dock)** | Blurrt mit der Uhr, früh. Dock-Icons werden zu Farbflecken. |
| **Oben links (Wetter-Widget)** | Wird **als letztes** unscharf. Bei Frame 152 ist „Leipzig 12°“ noch lesbar, während die Uhr komplett weg ist; ~6 Frames später ist auch das weg. |
| **Links (Icon-Spalte WhatsApp/Fotos/Telefon)** | Bleibt am längsten scharf, gleiche Front wie Wetter-Widget. |
| **Unten (Page-Dots, Wallpaper)** | Wallpaper wird mitgeblurrt, Page-Dots verschwinden im Blur. |

→ **Der Blur ist ein horizontaler Sweep: Die Blur-Front startet an der Hinge-Kante (rechts) und läuft zur
Außenkante (links).** Front-Breite (Übergangszone) ≈ 30–40 % der Cover-Breite, sehr weich. [abgeleitet]

Weitere Details:
- Keine Skalierung, kein Fade der Inhalte, Position bleibt. Nur Gaussian-Blur, Radius wächst bis ~20–28 dp
  (bei Cover-Breite ~400 dp). [abgeleitet]
- Abdunklung: In Video 2 wird das Cover beim Öffnen von der **Außenkante (links)** her dunkler. Das Panel kippt
  dort von der Kamera weg; es ist sehr wahrscheinlich gerendertes Off-Axis-Dimming/Beleuchtung und **kein
  UI-Effekt**. Siehe 9.2, optional.
- Nach Erreichen des Voll-Blurs bleibt das Cover so stehen, bis es abgeschaltet wird.

---

## 3. AUFKLAPPEN – Innen-Screen [beobachtet]

Zeitachse: Innen-Screen sichtbar ab θ ≈ 60°, linkes Panel scharf bei θ = 180°. Der größte Teil der
Entschärfung passiert in den **letzten ~25°** (θ 155°→180°, ~0,25 s).

| θ (ca.) | Rechtes Panel | Linkes Panel |
|---|---|---|
| 60–90° | **Sofort komplett scharf**, Cover-Komposition 1:1, inkl. Statusbar und Dock. (Das ist das scharfe Material, das neben dem Cover sichtbar ist.) | Fast schwarz/dunkelgrau. Nahe der Hinge-Kante ein schwacher heller, verwaschener Streifen; die linke Icon-Spalte „leuchtet“ als Farbflecken aus dem Knick. |
| 90–150° | unverändert | Blur-Radius maximal (~40–60 dp; Widgets nur große weiche Farbfelder). Heller Streifen wandert von der Hinge-Kante nach außen (9.5). Wallpaper mitgeblurrt. **Farbflecken bluten über Widget-Grenzen bis zum Panel-Rand**: Blur wird auf dem gesamten Panel-Layer gerechnet, nicht pro Widget geclippt. |
| 150–170° | unverändert | Außenkante jetzt hell. Blur-Radius nimmt deutlich ab, Formen erkennbar, nicht lesbar. |
| 175–180° | unverändert | Blur → 0, Abdunklung → 0. Ende. |

Ecken des Innen-Screens:
- **Oben rechts (Statusbar), rechts (Dock)**: immer scharf, Teil des nie geblurrten rechten Panels.
- **Mitte (Knick-Linie)**: weiche Maskenkante ~5–8 % der Gesamtbreite. Bei θ ≈ 80° liegt sie noch knapp rechts
  des Knicks (die hinge-nächste Spalte ist leicht weich), ab θ ≈ 110° knapp links davon (Spalte scharf). Kein
  Hard-Cut, keine Linie, Wallpaper nahtlos. [beobachtet, beide Videos]
- **Oben links / unten links (Panel-Außenecken)**: am dunkelsten, werden zuletzt hell.
- **Unten (Wallpaper linkes Panel)**: mitgeblurrt.

Kein Element bewegt sich. Keine Scale. Kein Cross-Fade zweier Layouts.

---

## 4. ZUKLAPPEN – Innen-Screen [beobachtet]

Blur auf dem linken Panel startet **sofort**, sobald θ unter ~178° fällt. Bei θ ≈ 150° ist er bereits stark;
bei ≈ 90° maximal + dunkel. Die Dunkelfront läuft von der Außenkante zur Hinge-Kante zurück. Das rechte Panel
bleibt bis zum Verschwinden aus dem Blickfeld scharf und unverändert.

**Zuklappen = exakte Umkehrung von Abschnitt 3, winkelgetrieben**, derselbe Kurvenverlauf.

Detail: In den ersten Frames (θ 178°→165°) blurren zuerst die großen hellen Flächen sichtbar (gelb/weiß),
Zahlen bleiben kurz lesbar. Das ist nur die Wahrnehmung eines wachsenden Radius, kein eigener Sweep.

---

## 5. ZUKLAPPEN – Cover-Screen [beobachtet]

Zeitachse Video 1: Cover sichtbar ~Frame 282 (θ ≈ 20°), geschlossen ~Frame 286, komplett scharf ~Frame 306–310.
→ **~0,55–0,6 s Entschärfung, größtenteils NACH dem vollständigen Schließen (zeitbasiert, nicht winkelbasiert).**
Video 2: gleiches Timing.

Phasen:
1. **θ ≈ 20°→0°: Cover sehr dunkel.** Nur schwache, stark verwaschene Farbschemen. Helligkeit ≈ 10–20 %.
   In Video 2 ist die Außenkante (links) dunkler als die Hinge-Seite (siehe 9.2).
2. **Nach Schließen, 0–200 ms: Helligkeit rampt auf 100 %** (ease-out). Blur noch maximal.
3. **Nach Schließen, ~100–600 ms: Entschärfungs-Sweep von der Außenkante (links) zur Hinge-Kante (rechts).**
   Wetter-Widget, WhatsApp, Fotos, Telefon werden zuerst scharf; Uhr-Widget, Statusbar, Dock und rechte
   Icon-Spalten zuletzt. Exakte Spiegelung des Aufklapp-Sweeps.
4. **Statusbar (Uhrzeit/WLAN unter dem Kameraloch) wird mit dem Sweep scharf, also unter den letzten Elementen.**
   Vorher ist sie unlesbar verwaschen. Das scharfe „20:33“ rechts daneben in den Videos ist das Innen-Panel.

Ecken des Covers beim Zuklappen:
- **Oben links (Wetter)**: erste scharfe Ecke.
- **Unten links (Telefon/Mail)**: kurz danach.
- **Oben rechts (Uhr + Statusbar)**: zuletzt.
- **Unten rechts (letzte Grid-Spalte + Dock-Ende)**: allerletzte scharfe Elemente.

---

## 6. Implementierungs-Spezifikation (für den Launcher) [abgeleitet]

### 6.0 Grundregel (Vorgabe des Auftraggebers)
**Alles hängt durchgehend vom geöffneten Winkel θ ab.** Blur-Radius, Maskenfront, Helligkeit, Lichtfront und
Maskenkante sind reine Funktionen von θ. Es gibt **keine** Timer, **keine** Mindest- oder Höchstdauern,
**keine** Animationen, die nach Erreichen von 0° oder 180° weiterlaufen, und **keinen** Zustand, der sich
merkt, ob gerade geöffnet oder geschlossen wird. Reißt der Nutzer das Gerät in 0,2 s auf, läuft der Verlauf in
0,2 s; hält er es bei 25° an, bleibt der Verlauf bei 25° stehen. Dreht er zurück, läuft er rückwärts.
Die einzige zeitliche Komponente ist die Sensor-Glättung von θ (Low-Pass, ~30–50 ms), damit Jitter nicht
sichtbar flackert. Damit der Verlauf trotzdem nicht als Sprung wirkt, muss er über **genug Grad** verteilt
sein (siehe Cover-Werte unten), nicht über Zeit gestreckt werden.

### 6.1 Eingangsgrößen
```
θ        Hinge-Winkel 0..180 (geglättet, Low-Pass 60–90 Hz, kein Jitter)
dθ/dt    Winkelgeschwindigkeit (Vorzeichen = öffnen/schließen)
g        Gravity-Vektor (gx, gy, gz) aus Gyroskop/Accelerometer-Fusion
t_closed Zeit seit θ < 3° (für die zeitbasierte Cover-Entschärfung)
```

### 6.2 Layer-Aufbau pro Screen
```
Cover-Screen:
  [Wallpaper + Widgets + Grid + Dock + Statusbar + Page-Dots] → EIN Render-Layer „coverContent“
  Es gibt KEINE ausgenommene Ebene. Effekt: Blur(radius) mit horizontaler Masken-Rampe + Brightness.

Innen-Screen:
  [Rechtes Panel (= Cover-Komposition inkl. Statusbar + Dock)] → Layer „rightPanel“, NIE geblurrt
  [Linkes Panel (eigene Widgets + Wallpaper-Hälfte)]           → Layer „leftPanel“
  Effekt auf leftPanel: Blur(radius) + Dim-Gradient + weiche Maskenkante am Hinge
```
Blur = Gaussian / RenderEffect.createBlurEffect (API 31+), Fallback RenderScript / vorgerechnete Mips. Blur
immer auf dem **ganzen Panel-Layer** rechnen, erst danach mit Screen-Rundung clippen.

### 6.3 Kurven

Koordinaten: `x` = 0 an der Hinge-Kante, 1 an der Außenkante, jeweils des betroffenen Screens/Panels.
`smoothstep(e0, e1, x)` immer mit e0 < e1 (GLSL-Semantik); Richtungsumkehr nur über `1 - smoothstep(...)`.
Prüfwerte: Cover-Aufklappen q=0 → Blur überall 0, q=1 → überall max. Cover-Zuklappen s=0 → überall max, s=1 → 0.
Innen lightFront=0 → Hinge-Kante hell, Rest dunkel; lightFront=1 → überall hell.

**Innen-Screen, linkes Panel (winkelgetrieben, symmetrisch auf/zu):**
```
p = clamp((θ - 120) / 60, 0, 1)        // 0 bei ≤120°, 1 bei 180°
e = p^3                                 // stark ease-in: fast alles in den letzten 25°
blurRadius = BLUR_MAX_INNER * (1 - e)   // BLUR_MAX_INNER ≈ 48 dp

lightFront = clamp((θ - 60) / 100, 0, 1)                           // 0 bei 60°, 1 bei 160°
f = lightFront * 1.3
dimAt(x)   = 0.6 * smoothstep(f - 0.25, f + 0.05, x)   // 0 (hell) an der Hinge-Kante, 0.6 (dunkel) jenseits der Front; bei 160° überall 0
           + 0.1 * (1 - e)                                           // globaler Rest
```
Maskenkante am Knick (`iconW` = Icon-Rasterbreite):
```
maskEdgeX = hingeX + iconW * (0.5 - clamp((θ - 60) / 50, 0, 1))   // wandert von rechts nach links über den Knick
blurWeight(x_px) = 1 - smoothstep(maskEdgeX - 0.07 * screenW, maskEdgeX, x_px)   // 1 links der Kante (Blur), 0 rechts davon
```

**Cover-Screen beim Aufklappen (winkelgetrieben + Sweep, Hinge rechts):**
```
q = clamp((θ - 2) / 38, 0, 1)                          // 2°..40° → 0..1
frontX = q * 1.35                                      // Blur-Front läuft von x=0 (Hinge) nach x>1 (außen)
strength = 0.3 + 0.7 * q                               // direkt hinter der Front anfangs nur 30 % Radius, am Ende 100 %
blurAt(x) = BLUR_MAX_COVER * strength * (1 - smoothstep(frontX - 0.35, frontX, x))   // bei q=0 überall 0
// Begründung: In Video 1 ist das Cover bei ~30–35° voll unscharf (Frame 158), Video 2 bei ~15–17°. Der
// schnelle Wert aus Video 2 wirkt beim Nachbau als Sprung, weil die ersten 17° in einem Augenblick vorbei
// sind. 40° mit wachsender Stärke deckt beide Videos ab und bleibt eine reine Funktion von θ.
// Falls es zu träge wirkt: 30° statt 40°, sonst nichts ändern. Keine Zeit-Untergrenze einbauen.
```
BLUR_MAX_COVER ≈ 24 dp. Gilt für den gesamten Layer inkl. Statusbar und Dock. Ab θ ≥ 40° bleibt der
Endzustand, bis der Screen ausgeht.

**Cover-Screen beim Zuklappen (winkelgetrieben, auf die letzten Grad gemappt):**
```
// Cover wird bei θ ≤ 40° eingeschaltet (symmetrisch zum Öffnen), zunächst dunkel und voll unscharf.
brightness = 0.15 + 0.85 * smoothstep(30, 8, θ)        // dunkel bei ≥30°, voll hell ab 8°
s = smoothstep(25, 0, θ)                               // Schärfe-Fortschritt 0 bei 25°, 1 bei 0°
frontX = 1.35 - s * 1.35                                // Schärfe-Front läuft von außen (x=1) zur Hinge-Kante (x=0)
blurAt(x) = BLUR_MAX_COVER * (1 - smoothstep(frontX - 0.35, frontX, x))   // bei s=0 überall unscharf, bei s=1 überall scharf
// smoothstep(a, b, θ) mit a > b ist hier als 1 - smoothstep(b, a, θ) zu lesen (fallende Rampe).
```
Weil das Cover-Öffnen über 2°–40° und das Cover-Schließen über 25°–0° läuft, sind Hin- und Rückweg nicht
dieselbe Kurve. Das ist gewollt und entspricht den Videos (Öffnen: Blur-Front Hinge→außen; Schließen:
Schärfe-Front außen→Hinge). Beim Richtungswechsel mitten im Verlauf gilt immer die Formel der aktuellen
Bewegungsrichtung, mit θ als einziger Eingabe; weil beide bei θ=0 scharf und bei θ≥40° voll unscharf sind,
gibt es an den Enden keinen Sprung. Dazwischen ist ein kleiner Sprung beim Umkehren akzeptiert.

**Abweichung zum Video [beobachtet]:** In beiden Videos ist das Gerät bei Beginn der Entschärfung bereits
geschlossen und die Entschärfung läuft danach noch ~0,5 s weiter (Abschnitt 5). Das wäre zeitgetrieben. Der
Auftraggeber will durchgehend winkelgetrieben; deshalb ist der Verlauf oben auf die letzten 25° gelegt. Wer
es exakt wie im Video will, muss die Entschärfung über eine Zeitrampe nach θ<1° laufen lassen; das ist
**nicht** die Vorgabe.

### 6.4 Was NICHT passiert (bewusst weglassen)
- Keine Timer, keine Mindestdauern, keine Nachlauf-Animationen (6.0).
- Kein Scale/Translate/Parallax der Icons, Widgets, des Docks oder des Wallpapers (per Tracking bestätigt, Abschnitt 10).
- Kein Cross-Fade zwischen zwei unterschiedlichen Layouts.
- Keine Hinge-Linie, kein Schatten entlang der Knicklinie, keine Lichtkante.
- **Keine Ausnahmen vom Blur auf dem Cover.** Statusbar und Dock des Covers blurren mit.
- Kein Blur auf dem rechten Innen-Panel (dort bleiben Statusbar und Dock scharf, weil das ganze Panel scharf bleibt).
- Kein Umschalten der Wallpaper-Position: Innen-Wallpaper = Cover-Wallpaper, rechts deckungsgleich, links Fortsetzung.

---

## 7. Gyroskop / Perspektive [Extrapolation – NIEDRIGE PRIORITÄT, optional, zuletzt bauen]

In beiden Videos liegen alle Blur-Fronten und Dunkel-Verläufe parallel zur Hinge-Achse und richten sich nicht
nach der Kamera; auch in Video 2 (schräg von oben) ändert sich daran nichts. Es gibt also **keinen** im Material
sichtbaren Gyro-Effekt. Für den Launcher soll die Animation trotzdem auf die Lage im Raum reagieren.
Vorschlag, der nur die *Maske* räumlich interpretiert und das Video-Verhalten als Default behält:

1. **Physik-Modell:** Das geblurrte Panel ist eine Milchglas-Scheibe, die physisch auf dem Panel liegt.
   `hingeDir` = Projektion der Hinge-Achse (aus `FoldingFeature.bounds` + `g`) auf die Screen-Ebene. Masken-
   und Lichtfronten liegen senkrecht zu `hingeDir`. Bei normaler Haltung ist das exakt das Video-Verhalten.
   Bei gekipptem Gerät (Roll ±30°) kippt die Front um denselben Winkel mit, geclampt auf ±12°.
2. **Perspektiv-Skew der Blur-Front (Cover):** Die Front ist ein Trapez: die laut `g` + θ weiter vom Betrachter
   entfernte Seite bekommt eine bis zu 20 % breitere Übergangszone.
3. **Lichtfront nach Gravity (linkes Innen-Panel):** `lightFront` aus 6.3 bekommt den Faktor 0.7–1.3, je nachdem
   ob das Panel laut `g` nach oben (schneller hell) oder nach unten (langsamer hell) zeigt. Low-Pass 0,3 s.
4. **Wallpaper-Parallax nur auf dem geblurrten Panel:** solange `blurRadius > 0`, darf der verschwommene
   Wallpaper-Layer bis 6 dp gegen `g` driften (Low-Pass 0,3 s). Scharfe Elemente nie. Bei `blurRadius == 0`
   wieder 0 dp, mit 150 ms ease.
5. **Reduce-Motion:** Punkte 1–4 abschaltbar; dann exakt das 2D-Verhalten aus Abschnitt 2–5.

Alle Gyro-Effekte sind additiv und klein; die Kernanimation darf sich durch Haltungsänderung nie um mehr als ~15 % verändern.

---

## 8. Abnahme-Checkliste (gegen die Videos prüfen)

- [ ] Aufklappen Cover: Uhr-Widget und Statusbar (Hinge-Seite, rechts) blurren vor dem Wetter-Widget (außen, links).
- [ ] Aufklappen Cover: **nichts** bleibt scharf, auch nicht Statusbar oder Dock.
- [ ] Aufklappen Cover: komplett unscharf bei θ ≈ 40°.
- [ ] Innen bei θ ≈ 90°: rechtes Panel 1:1 Cover-Layout, scharf inkl. Statusbar/Dock; linkes Panel dunkel + maximal unscharf.
- [ ] Innen: Maskenkante wandert zwischen 60° und 110° von knapp rechts nach knapp links des Knicks; linke Icon-Spalte danach scharf.
- [ ] Innen: heller Streifen wandert von der Hinge-Kante nach außen (60°→160°), Außen-Ecken zuletzt hell.
- [ ] Innen: Widget-Farben bluten im Blur über Widget-Grenzen bis zum Panel-Rand.
- [ ] Innen: Entschärfung fast komplett in den letzten 25° vor 180°.
- [ ] Zuklappen: linkes Panel blurrt ab dem ersten Grad unter 180°.
- [ ] Zuklappen Cover: kommt bei ≤40° dunkel, wird bis 8° hell, Schärfe-Sweep von außen (links) zur Hinge-Kante (rechts) über 25°→0°.
- [ ] Gerät bei beliebigem Winkel anhalten → Bild friert exakt ein, nichts läuft weiter. Zurückdrehen → läuft rückwärts.
- [ ] Aufklappen Cover: Verlauf über 2°–40°, Stärke wächst mit; bei ruckartigem Öffnen kein Sprung, nur schnell.
- [ ] Zuklappen Cover: Wetter-Widget scharf vor Uhr-Widget; Statusbar und Dock unter den letzten scharfen Elementen.
- [ ] Kein Element bewegt oder skaliert sich in irgendeiner Phase.
- [ ] Unterbrechen (halb auf, wieder zu) erzeugt keine Sprünge.
- [ ] Gyro-Effekte an/aus unterscheidbar, aber nie dominant; Reduce-Motion → reines 2D.

---

## 9. Ergänzungen aus Video 2 (MKBHD-Short, schräg von oben gefilmt)

Grundmechanik identisch zu Abschnitt 1–5. Die Punkte unten sind zusätzlich sichtbar bzw. präzisieren.

### 9.1 Zurückgenommen: „Dock bleibt scharf“
Eine frühere Version behauptete, das Cover-Dock bleibe scharf. Das scharfe Dock neben dem Cover ist das Dock
des **rechten Innen-Panels**, das während des Klappens neben dem Cover-Panel sichtbar ist. Das Cover-Dock
blurrt mit allem anderen (Abschnitt 2). Kein Flag nötig.

### 9.2 Dunkel-Verlauf am Cover von der Außenkante [beobachtet, Video 2; Einordnung abgeleitet]
- Beim Öffnen wird das Cover von der **Außenkante (links)** her grau-dunkel; bei θ ≈ 30° reicht der Verlauf bis
  ~70 % der Breite. Beim Schließen taucht das Cover mit diesem Verlauf auf und er verschwindet in ~200 ms nach
  dem Schließen.
- Die Außenkante ist genau die Seite, die beim Klappen von der Kamera wegkippt. Sehr wahrscheinlich ist das
  gerendertes Off-Axis-Dimming / Beleuchtung des 3D-Modells, **kein UI-Effekt**. Video 1 zeigt es nicht
  eindeutig.
- Empfehlung: **nicht als UI nachbauen.** Falls gewünscht, als optionalen Gyro-Effekt (Abschnitt 7) mit
  `edgeDim = 0.3 * clamp(θ / 35, 0, 1)` von der Außenkante, default aus.

### 9.3 Cover: Blur-Onset schneller [beobachtet, Video 2]
Blur setzt bei θ ≈ 2° ein, komplett nach ~0,25 s (θ ≈ 15°). Video 1: ~0,45 s bis ≈ 25°. In 6.3 ist die
schnellere Variante eingetragen; die Front-Richtung (Hinge → außen) bleibt.

### 9.4 Zurückgenommen: „Statusbar blendet spät ein“
Die Statusbar des Covers ist einfach Teil des geblurrten Layers und wird mit dem Sweep an der Hinge-Seite
als eines der letzten Elemente scharf (Abschnitt 5, Punkt 4). Kein eigener Fade, kein Flag.

### 9.5 Innen, linkes Panel: Lichtfront von der Hinge-Kante [beobachtet, Video 2]
Zwischen θ ≈ 60° und 160° ist das linke Panel nicht gleichmäßig dunkel: nahe der Hinge-Kante ein heller,
verwaschener Streifen, der mit wachsendem θ nach außen wandert; die linke Screen-Kante bleibt am längsten
dunkelgrau, zuletzt nur die Ecken oben links / unten links. Beim Zuklappen läuft die Dunkelfront von außen
zurück zur Hinge-Kante. Formel in 6.3 (`lightFront`, `dimAt`). Front-Breite ≈ 30 % Panel-Breite, sehr weich.

### 9.6 Innen: Maskenkante am Knick wandert mit θ [beobachtet, beide Videos]
Bei θ ≈ 80° ist die hinge-nächste Spalte des rechten Panels leicht weich; bei θ ≈ 110° scharf. Die Maskenkante
startet ~eine halbe Icon-Breite rechts vom Knick und wandert bis θ ≈ 110° auf ~eine halbe Icon-Breite links
davon. Formel in 6.3 (`maskEdgeX`).

### 9.7 Kein Element bewegt sich, mit Vorbehalt [beobachtet, Video 2]
Beim Entschärfen des linken Panels (θ 165°→180°) wirken die weißen Widget-Flächen minimal größer als ihr
scharfer Endzustand (≤ 5 %). Das kann reine Blur-Ausbreitung sein. **Keine Skalierung bauen**; optional Flag
`leftPanelSettleScale = 1.03 → 1.0` über die letzten 15°, default aus.

### 9.8 Was Video 2 für die Gyro-Perspektive liefert
Nichts Zusätzliches: Auch bei schräger Kameraperspektive bleiben alle Fronten hinge-parallel. Die
Gyro-Kopplung in Abschnitt 7 ist vollständig Extrapolation.

---

## 10. Parallaxe / Bewegung der Apps beim Öffnen und Schließen [gemessen, Video 1 + 2]

Der Eindruck, dass die Apps beim Klappen „wandern“, ist real, aber er entsteht **nicht durch eine Verschiebung in der UI**.
Gemessen wurde per Template-Tracking mit Gehäuse-Referenz (Bezel-Ecken + Hinge-Naht, Similarity-Fit), Residuen in px:

| Phase (Video 1, Frames) | Icons / Widgets / Dock / Statusbar relativ zum Gehäuse | Wallpaper relativ zum Gehäuse |
|---|---|---|
| Cover geschlossen, vor dem Blur (104–137) | ±3 px (Hand-Perspektive) | ±3 px |
| Innen rechtes Panel, Öffnen 60°→180° (172–199) | **0–2 px** | **0–2 px** |
| Innen rechtes Panel, flach (199–220) | 0–1 px | 0–1 px |
| Innen rechtes Panel, Schließen 180°→150° (226–262) | gleicher Verlauf wie Wallpaper, kein Unterschied | wie Icons |
| Innen linkes Panel, flach (198–226) | Widgets 0–5 px | Wallpaper ≤ 20 px, Messung unsicher (weiche Wolken), visuell nicht erkennbar |
| Cover nach dem Schließen (305–355) | Verschiebung vollständig durch Drehung der Hand erklärt (6°) | – |

**Ergebnis: Kein Element bewegt oder skaliert sich relativ zum Panel.** Icons, Widgets, Dock, Statusbar und
Wallpaper sind in jeder Phase pixelfest am Gehäuse. Keine iOS-artige Bewegungsparallaxe zwischen Wallpaper und
Icons ist nachweisbar. Video 2 ist damit konsistent (Layout bei 100° identisch mit Layout bei 180°).

### 10.1 Woher der Parallaxe-Eindruck kommt [beobachtet]
1. **Verdeckung durch das schwingende Panel.** Zwischen ~60° und ~130° steht das linke Panel aus Kamerasicht
   **vor** dem Knick und verdeckt den hinge-nächsten Streifen des rechten Panels. Video 2, θ ≈ 100°: Sleep-Widget
   und die zwei linken Icon-Spalten (FaceTime/Calendar, Mail/Notes, Siri/TV, Shortcuts/App Store) sind komplett
   hinter dem linken Panel verschwunden, sichtbar sind nur Calendar-Widget, Spalten 3–4 und das Dock. Video 1,
   θ ≈ 90°: Wetter-Widget („…zig 12°“) und WhatsApp-Spalte sind am Knick angeschnitten. Beim Weiteröffnen
   „schieben“ sich diese Spalten scheinbar aus dem Knick heraus. Das ist reine Geometrie und passiert auf einem
   echten Foldable von selbst.
2. **Wandernde Blur-Maskenkante (9.6).** Die hinge-nächste Spalte wird zwischen 80° und 110° von „leicht weich“
   zu scharf. Zusammen mit (1) wirkt es, als würde die Spalte beim Öffnen „hervortreten“.
3. **Perspektivische Stauchung.** Das rechte Panel kippt während des Klappens leicht mit (in Video 1 ~2–3°
   Bildrotation + Foreshortening). Der Icon-Abstand zum Knick wirkt dadurch variabel, ist es aber nicht.
4. **Blur-Ausbreitung.** Helle Flächen (Uhr-Widget, Notizen-gelb) wirken im Blur größer und leicht versetzt,
   weil ihr Helligkeitsschwerpunkt nach außen wandert. Keine Skalierung in der UI (siehe 9.7).

### 10.2 Konsequenz für den Launcher
- **Nichts bauen, was Elemente verschiebt oder skaliert.** Layout pro Panel fest verankern (rechtes Panel =
  Cover-Komposition, linkes Panel = Zusatz-Widgets), unabhängig von θ.
- Die wandernde Maskenkante aus 9.6 ist der einzige Teil des „Parallaxe-Looks“, den der Launcher selbst erzeugt.
  Sie muss drin sein, sonst wirkt der Übergang am Knick tot.
- Verdeckung und Perspektive liefert die Hardware.
- **Optional, nur wenn der Auftraggeber den Eindruck künstlich verstärken will** (Flag `hingeParallax`, default
  aus): Inhalt des rechten Panels bei θ ≈ 90° um bis zu 10 dp **zum Knick hin** verschoben, linear zurück auf 0
  bei θ ≥ 150°; Wallpaper dabei nur die Hälfte. Das reproduziert den Verdeckungseindruck, wenn die Hardware
  ihn nicht zeigt. Nicht aus dem Video belegt.

---

## 11. Weitere Details aus Video 1 [beobachtet]

- **Abdunklung am Cover beim Öffnen sitzt an der Hinge-Seite (rechts).** Frames 140–158: das rechte Drittel
  des Covers (Dock, rechte Icon-Spalten, Uhr) wird mit wachsendem θ grau-dunkel, das linke Drittel (Wetter,
  WhatsApp) bleibt hell. In Video 2 beim Schließen liegt der dunkle Verlauf an der Außenkante. Beides passt zu
  **gerendertem Schatten/Beleuchtung des 3D-Modells** (der jeweils andere Panelkörper wirft Schatten), nicht zu
  einem UI-Effekt. Bleibt bei „nicht nachbauen“ (9.2).
- **Blur-Beginn liegt exakt auf dem ersten Hinge-Frame.** Frames 104–137 ist das Gerät geschlossen und wird
  nur gehalten, Cover bleibt scharf. Frame 140 (erste sichtbare Spaltöffnung, θ ≈ 3–5°) ist der erste Frame mit
  Blur. Kein Vorlauf, keine Verzögerung.
- **Page-Dots** (zwei Punkte unten mittig auf dem Cover) blurren mit und verschwinden. Keine Sonderbehandlung.
- **Badges** (rote Zähler 1/4/325) blurren mit, werden zu roten Flecken.
- **Linke Innen-Widgets erscheinen an ihrer Endposition.** Batterie-Widget bei θ ≈ 165° (blurred) und bei 180°
  (scharf) am selben Ort (≤ 8 px inkl. Perspektive). Kein Slide-in, kein Scale-in.
- **Kein Overshoot.** Blur-Abbau am Innen-Screen (184–192) und Cover-Entschärfung (286–310) sind monoton.
  Keine Feder, kein kurzes Nachblurren.
- **Statusbar ist in beiden Modi identisch** (Uhrzeit, Batterie, WLAN oben rechts neben/unter dem Kameraloch).
  Kein Layoutwechsel der Statusbar zwischen Cover und Innen.
- **Hinge-Naht oben am Gehäuse** ist physisch sichtbar; auf dem flachen Innen-Screen gibt es keine sichtbare
  Knicklinie in der UI und keine UI-Behandlung dafür.
- **Dock-Pill** (halbtransparentes abgerundetes Rechteck) ist auf Cover und rechtem Panel identisch, keine
  Formänderung beim Klappen.
