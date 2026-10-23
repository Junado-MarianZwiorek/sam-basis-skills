# Handoff: Fold-Animation „iPhone Duo“-Style für eigenen Launcher

> Analysegrundlage: 15 s Screen-Recording (1432×1080, ~39,6 fps, 592 Frames) eines iPhone-Duo-Videos.
> Enthält 2× Aufklappen (Frames ~135–195 und ~400–445) und 1× Zuklappen (Frames ~228–310).
> Das Gerät wird in der Hand gehalten, Hinge vertikal (Book-Style-Foldable). Cover-Screen sitzt
> auf der Rückseite des **rechten** Panels; das **linke** Panel schwingt auf.

---

## 0. Aufgabe für dich (Ziel-Session)

Baue im Launcher eine **winkelgesteuerte** Übergangsanimation zwischen Cover-Screen und Innen-Screen nach,
die exakt dem unten beschriebenen Verhalten folgt. Treiber ist der **Hinge-Winkel θ** (0° = zu, 180° = flach)
aus dem Hinge-Sensor (Android: `WindowInfoTracker` / `FoldingFeature` + `Sensor.TYPE_HINGE_ANGLE`).
Zusätzlich fließt der **Gyroskop-/Gravity-Vektor** als Perspektiv-Modulator ein (Abschnitt 6).

Alles, was mit **[beobachtet]** markiert ist, ist im Video direkt sichtbar. Alles mit **[abgeleitet]** ist
Parametrisierung/Interpretation, die zu den Frames passt. **[Extrapolation]** = nicht im Video, vom Auftraggeber gewünscht.

---

## 1. Layout-Grundprinzip (Cover ↔ Innen) [beobachtet]

- Der Cover-Screen zeigt: Wetter-Widget (oben links) + Uhr-Widget (oben rechts) + 4×4 Icon-Grid + vertikales Dock am **rechten** Rand + Statusbar oben rechts (Uhrzeit, WLAN) neben der Kamera-Aussparung.
- Der Innen-Screen (Landscape, ~2:1 breiter) zeigt **exakt dieselbe Cover-Komposition pixelidentisch auf dem rechten Panel** (Wetter, Uhr, 4×4 Grid, rechtes Dock, Statusbar oben rechts). Nichts davon skaliert, verschiebt oder re-layoutet sich.
- Das **linke Panel** bekommt zusätzlich eine Widget-Spalte: Batterie-Widget (4 Ringe) oben, Kalender-Widget (weiß/lila) mittig, Notizen-Widget (gelb/weiß) unten.
- Die **linke Icon-Spalte** des Grids (WhatsApp / Fotos / Telefon / Mail) liegt **genau auf der Knick-Linie**. Beim Falten bleibt sie **scharf** (gehört logisch zum rechten Panel). Die Blur-Maske beginnt also ein Stück *links* der Knick-Linie, nicht exakt darauf.
- Folgerung für den Launcher: Cover-Layout = rechte Hälfte des Innen-Layouts. Es gibt **keine** Slide-/Scale-/Morph-Animation von Elementen. Die gesamte Animation ist: **Blur + Helligkeit + Maskensweep**, pro Panel getrennt.

---

## 2. AUFKLAPPEN – Cover-Screen [beobachtet]

Zeitachse (1. Aufklappen): Blur beginnt ~Frame 140 (3,5 s), voll geblurrt ~Frame 158 (4,0 s) → **~0,45 s**, dabei ist θ erst ca. 5°–25°. Die Cover-Animation ist praktisch fertig, bevor das Gerät sichtbar offen ist.

Was passiert, Ecke für Ecke:

| Bereich Cover | Verhalten |
|---|---|
| **Oben rechts (Statusbar: 20:33, WLAN, Kamera-Loch)** | Bleibt **durchgehend scharf**. Statusbar ist vom Blur ausgenommen. |
| **Oben rechts (Uhr-Widget)** | Wird **als erstes** unscharf, am stärksten. Weißes Zifferblatt verschwimmt zu weißem Rechteck mit weichem Rand, Zeiger werden zu grauem „7“-Schemen. |
| **Oben links (Wetter-Widget)** | Wird **als letztes** unscharf. Im Frame 152 ist „Leipzig 12°“ noch lesbar, während die Uhr schon komplett weg ist. ~6 Frames später ist auch das Wetter-Widget weg. |
| **Links (Icon-Spalte WhatsApp/Fotos/Telefon)** | Bleibt am längsten scharf (gleiche Front wie Wetter-Widget). |
| **Rechts (Dock + rechte Icon-Spalten)** | Blurren mit der Uhr, also früh. |
| **Unten (Page-Dots, Wallpaper-Wolken)** | Wallpaper wird **mitgeblurrt** (Wolken verschwimmen), Page-Dots verschwinden im Blur. |

→ **Der Blur ist ein horizontaler Sweep: Die Blur-Front läuft vom rechten (Außen-)Rand zum linken (Hinge-)Rand.** [beobachtet]
Front-Breite (Übergangszone scharf→unscharf) ≈ 30–40 % der Cover-Breite, sehr weich. [abgeleitet]

Weitere Details:
- Keine Skalierung, kein Fade der Inhalte, Position bleibt. Nur Gaussian-Blur, Radius wächst bis ~ 20–28 dp (bei Cover-Breite ~ 400 dp; Icons werden zu weichen Farbflecken, Labels komplett unlesbar, Widgets zu Farbflächen). [abgeleitet]
- Leichte Abdunklung des Cover-Contents im voll geblurrten Zustand (≈ 10–15 %). [abgeleitet, auf Video schwer von Blickwinkel zu trennen]
- Nach Erreichen des Voll-Blurs bleibt der Cover-Screen so stehen, bis er abgeschaltet wird (Gerät dreht sich aus dem Blick). Kein weiteres Verhalten.

---

## 3. AUFKLAPPEN – Innen-Screen [beobachtet]

Zeitachse: Innen-Screen sichtbar ab ~Frame 164 (θ ≈ 60–80°), linkes Panel wird scharf bis ~Frame 190 (θ = 180°). Der größte Teil der Entschärfung passiert in den **letzten ~25°** (θ 155°→180°, ~0,25 s).

Zustände nach Hinge-Winkel:

| θ (ca.) | Rechtes Panel | Linkes Panel |
|---|---|---|
| 60–90° | **Sofort komplett scharf**, Cover-Komposition 1:1. Statusbar oben rechts scharf. Dock rechts scharf. | Fast schwarz/dunkelblau, nur schwache verschwommene Schemen (die linke Icon-Spalte „leuchtet“ als Farbflecken aus dem Knick heraus). Starke Abdunklung (≈ 50–60 %). |
| 120–150° | unverändert | Blur-Radius maximal (~ 40–60 dp; Widgets sind nur noch große weiche Farbfelder: gelber Streifen = Notizen, lila/weißes Feld = Kalender, grüne Ringe = Batterie). Abdunklung ≈ 30 %. Wallpaper-Sterne ebenfalls verschwommen. Die **Farbflecken bluten über die Widget-Grenzen hinaus** bis an den linken Screen-Rand – Blur wird auf dem gesamten Panel-Layer gerechnet, nicht pro Widget geclippt. |
| 160–170° | unverändert | Blur-Radius nimmt deutlich ab (Batterie-Ringe und Kalendertext als Formen erkennbar, nicht lesbar). Helligkeit fast normal. |
| 175–180° | unverändert | Blur → 0, Abdunklung → 0. Ende. |

Ecken des Innen-Screens:
- **Oben rechts (Statusbar)**: immer scharf, nie geblurrt.
- **Rechts (vertikales Dock)**: immer scharf.
- **Mitte (Knick-Linie)**: Die Blur-Maske hat eine **weiche Kante ~ 5–8 % der Gesamtbreite**, beginnt ca. eine halbe Icon-Breite **links** der physischen Knick-Linie. Die linke Icon-Spalte (WhatsApp/Fotos/Telefon/Mail) bleibt dadurch scharf, ihre Labels (ganz links „WhatsApp“) werden teilweise angeschnitten/leicht weich. Der Wallpaper-Übergang an der Kante ist nahtlos (kein Hard-Cut, kein Linie).
- **Oben links / unten links (Panel-Außenecken)**: Am dunkelsten und am stärksten verwaschen; die Abdunklung wirkt wie eine leichte Vignette, die zum Hinge hin abnimmt. [abgeleitet]
- **Unten (Wallpaper-Wolken linkes Panel)**: mitgeblurrt.

Kein Element bewegt sich. Keine Scale. Kein Cross-Fade zweier Layouts.

---

## 4. ZUKLAPPEN – Innen-Screen [beobachtet]

Zeitachse: Blur auf dem linken Panel startet **sofort**, sobald θ unter ~178° fällt (Frame ~230, Gerät sieht noch praktisch flach aus). Bei θ ≈ 150° ist der Blur bereits stark; bei ≈ 90° maximal + dunkel. Das rechte Panel bleibt bis zum Verschwinden aus dem Blickfeld scharf und unverändert.

Also: **Zuklappen = exakt die Umkehrung von Abschnitt 3, winkelgetrieben**, derselbe Kurvenverlauf. Nichts Asymmetrisches auf dem Innen-Screen.

Detail: In den ersten Frames (θ 178°→165°) blurren zuerst die **großen hellen Flächen** sichtbar (Notizen-gelb, Kalender-weiß), während Batterie-Prozentzahlen noch kurz lesbar sind. Das ist einfach die Wahrnehmung eines wachsenden Blur-Radius – kein eigener Sweep. [beobachtet]

---

## 5. ZUKLAPPEN – Cover-Screen [beobachtet]

Zeitachse: Cover wird sichtbar ~Frame 282 (θ ≈ 20°), Gerät geschlossen ~Frame 286, Cover komplett scharf ~Frame 306–310. → **~0,55–0,6 s Entschärfung, die größtenteils NACH dem vollständigen Schließen abläuft (zeitbasiert, nicht winkelbasiert)**.

Phasen:
1. **θ ≈ 20°→0°: Cover ist fast schwarz.** Nur extrem schwache, stark verwaschene Farbschemen (grün von WhatsApp, weiß von Uhr) sind sichtbar. Helligkeit ≈ 10–20 %. [beobachtet]
2. **Nach Schließen, 0–200 ms: Helligkeit rampt auf 100 %** (ease-out). Blur noch maximal. [beobachtet]
3. **Nach Schließen, ~100–600 ms: Entschärfungs-Sweep vom Hinge-Rand (links) zum Außenrand (rechts).** Wetter-Widget, WhatsApp, Fotos, Telefon werden als erstes scharf; Uhr-Widget, Dock und rechte Icon-Spalten als letztes. Das ist die **exakte Spiegelung des Aufklapp-Sweeps** (dort: Blur-Front von rechts nach links; hier: Schärfe-Front von links nach rechts). [beobachtet]
4. Statusbar oben rechts ist sofort scharf, sobald der Screen hell ist. [beobachtet]

Ecken des Covers beim Zuklappen:
- **Oben links (Wetter)**: wird als erste Ecke scharf.
- **Unten links (Telefon/Mail)**: kurz danach.
- **Oben rechts (Uhr)**: als letztes Widget scharf; die Statusbar daneben ist aber schon scharf → optisch „die Zeit steht, die Uhr erscheint“.
- **Unten rechts (YouTube/YT Studio + Dock-Ende)**: allerletzte scharfe Elemente.

---

## 6. Implementierungs-Spezifikation (für den Launcher) [abgeleitet]

### 6.1 Eingangsgrößen
```
θ        Hinge-Winkel 0..180 (geglättet, z. B. Low-Pass 60–90 Hz, kein Jitter)
dθ/dt    Winkelgeschwindigkeit (Vorzeichen = öffnen/schließen)
g        Gravity-Vektor (gx, gy, gz) aus Gyroskop/Accelerometer-Fusion (TYPE_GRAVITY / ROTATION_VECTOR)
t_closed Zeit seit θ < 3° (für die zeitbasierte Cover-Entschärfung)
```

### 6.2 Layer-Aufbau pro Screen
```
Cover-Screen:
  [Wallpaper + Widgets + Grid + Dock]  → ein Render-Layer „coverContent“
  [Statusbar]                           → eigener Layer, NIE geblurrt
  Effekt auf coverContent: Blur(radius) mit horizontaler Masken-Rampe + Dim(alpha)

Innen-Screen:
  [Rechtes Panel (= Cover-Komposition) + Statusbar + Dock] → Layer „rightPanel“, NIE geblurrt
  [Linkes Panel (Batterie/Kalender/Notizen + Wallpaper-Hälfte)] → Layer „leftPanel“
  Effekt auf leftPanel: Blur(radius) + Dim(alpha) + weiche Maskenkante am Hinge
```
Blur = Gaussian / RenderEffect.createBlurEffect (API 31+), Fallback: RenderScript / vorgerechnete Mips. Blur immer auf dem **ganzen Panel-Layer** rechnen (Farben bluten über Widget-Grenzen), erst danach mit Screen-Rundung clippen.

### 6.3 Kurven

**Innen-Screen, linkes Panel (winkelgetrieben, symmetrisch auf/zu):**
```
p = clamp((θ - 120) / 60, 0, 1)        // 0 bei ≤120°, 1 bei 180°
e = p^3                                 // stark ease-in: fast alles passiert in den letzten 25°
blurRadius = BLUR_MAX_INNER * (1 - e)   // BLUR_MAX_INNER ≈ 48 dp
dimAlpha   = DIM_MAX_INNER  * (1 - e)   // DIM_MAX_INNER ≈ 0.35, plus Vignette zur Außenkante +0.15
// Unter 120°: zusätzlich dimAlpha → 0.6 linear bis 60°
```
Maske am Hinge: Blur-Stärke multipliziert mit `smoothstep(hingeX - 0.5*iconW, hingeX - 0.5*iconW - 0.07*screenW, x)` – d. h. volle Schärfe ab halber Icon-Breite links der Knicklinie, volle Unschärfe 7 % weiter links.

**Cover-Screen beim Aufklappen (winkelgetrieben + Sweep):**
```
q = clamp((θ - 3) / 22, 0, 1)                    // 3°..25° → 0..1
frontX = 1.0 - q * 1.35                           // Blur-Front läuft von x=1 (rechts/außen) nach x<0 (links/hinge)
blurAt(x) = BLUR_MAX_COVER * smoothstep(frontX + 0.35, frontX, x)   // 35 % Breite weiche Front
dimAlpha = 0.12 * q
```
BLUR_MAX_COVER ≈ 24 dp. Statusbar ausgenommen. Ab θ > 25° bleibt der Cover-Screen im Endzustand, bis er ausgeht.

**Cover-Screen beim Zuklappen (zeitgetrieben nach Schließen):**
```
// Phase A, während θ 25°→0°: Cover an, Helligkeit 0.15, Blur max, kein Sweep
// Phase B, t_closed:
brightness = easeOutCubic(clamp(t_closed / 200ms))          // 0.15 → 1.0
s = easeOutCubic(clamp((t_closed - 100ms) / 500ms))          // 0..1
frontX = -0.35 + s * 1.35                                    // Schärfe-Front von links (Hinge) nach rechts
blurAt(x) = BLUR_MAX_COVER * smoothstep(frontX, frontX + 0.35, x)
```
Wenn der Nutzer vor Ende von Phase B wieder öffnet: aktuellen Blur-Zustand als Startwert übernehmen, keine Sprünge.

### 6.4 Was NICHT passiert (bewusst weglassen)
- Kein Scale/Translate/Parallax der Icons oder Widgets.
- Kein Cross-Fade zwischen zwei unterschiedlichen Layouts.
- Keine Hinge-Linie, kein Schatten entlang der Knicklinie, keine Lichtkante.
- Kein Blur auf Statusbar, Dock oder rechtem Panel.
- Kein Flackern/Umschalten der Wallpaper-Position: Innen-Wallpaper = Cover-Wallpaper, rechts deckungsgleich, links Fortsetzung.

---

## 7. Gyroskop / Perspektive [Extrapolation – vom Auftraggeber gewünscht]

Im Video wird das Gerät in der Hand gehalten, Hinge-Achse ≈ vertikal, Kamera frontal. Das beobachtete Verhalten ist rein 2D (Blur-Maske parallel zur Hinge-Achse). Für den Launcher soll die Animation **perspektivisch auf die Lage im Raum reagieren**. Vorschlag, der die Beobachtung nicht verfälscht, sondern nur die *Maske* räumlich interpretiert:

1. **Physik-Modell:** Das geblurrte Panel wird behandelt wie eine Milchglas-Scheibe, die physisch auf dem aufschwingenden Panel liegt. Die Blur-Front (Cover) und die Hinge-Kante (Innen) werden mit einer **Projektion des realen Hinge-Vektors** auf die Screen-Ebene ausgerichtet, nicht fix vertikal.
   - Berechne `hingeDir` = Projektion der Hinge-Achse (aus `FoldingFeature.bounds` + Gravity `g`) auf die Screen-Ebene.
   - Maskenkante ist eine Linie senkrecht zu `hingeDir`-Normale; bei normaler Haltung ist das exakt die vertikale Linie aus Abschnitt 3/5 (→ Video-Verhalten bleibt der Default).
   - Bei gekipptem Gerät (Roll ±30°) kippt die Blur-Front um denselben Winkel mit, max. ±12° (Clamp, damit es nie „falsch“ wirkt).

2. **Perspektiv-Skew der Blur-Front (Cover):** Die Front ist kein Rechteck-Gradient, sondern ein Trapez: Die Seite, die physisch weiter vom Betrachter weg ist (aus `g` + θ abgeleitet), bekommt eine um bis zu 20 % breitere Übergangszone. Visuell: „der Blur rollt wie ein Vorhang schräg ab“, wenn man das Gerät von unten sieht.

3. **Dim/Vignette auf dem linken Innen-Panel nach Gravity:** `dimAlpha` bekommt eine lineare Komponente über das Panel: dunkler an der Kante, die laut `g` nach unten zeigt (+0.08), heller an der nach oben zeigenden (−0.05). Simuliert, dass Milchglas von oben mehr Licht bekommt.

4. **Wallpaper-Parallax, nur auf dem geblurrten Panel:** Während `blurRadius > 0` darf der verschwommene Wallpaper-Layer um bis zu 6 dp gegen `g` driften (Low-Pass 0.3 s). Scharfe Elemente nie. Damit wirkt die Unschärfe „tief“ statt flach. Bei blurRadius == 0 wieder exakt 0 dp (hart snappen vermeiden: 150 ms ease).

5. **Reduce-Motion:** Alle Punkte 1–4 abschaltbar; dann exakt das 2D-Verhalten aus Abschnitt 2–5.

Alle Gyro-Effekte sind additiv und klein; die Kernanimation (Blur/Dim/Sweep nach θ) darf sich durch Haltungsänderung nie um mehr als ~15 % verändern.

---

## 8. Abnahme-Checkliste (gegen das Video prüfen)

- [ ] Aufklappen: Cover-Uhr blurrt vor dem Wetter-Widget; Statusbar bleibt scharf.
- [ ] Aufklappen: Cover ist bei θ ≈ 25° komplett unscharf.
- [ ] Innen bei θ ≈ 90°: rechtes Panel 1:1 Cover-Layout, scharf; linkes Panel dunkel + maximal unscharf.
- [ ] Innen: linke Icon-Spalte am Knick immer scharf; Blur beginnt links daneben mit weicher Kante.
- [ ] Innen: Widget-Farben bluten im Blur über Widget-Grenzen bis zum Panel-Rand.
- [ ] Innen: Entschärfung fast komplett in den letzten 25° vor 180°.
- [ ] Zuklappen: linkes Panel blurrt ab dem ersten Grad unter 180°.
- [ ] Zuklappen: Cover kommt dunkel, hellt in ~200 ms auf, Schärfe-Sweep von Hinge-Seite nach außen über ~500 ms **nach** dem Schließen.
- [ ] Zuklappen: Wetter-Widget scharf vor Uhr-Widget; Statusbar sofort scharf.
- [ ] Kein Element bewegt oder skaliert sich in irgendeiner Phase.
- [ ] Unterbrechen (halb auf, wieder zu) erzeugt keine Sprünge.
- [ ] Gyro-Effekte an/aus unterscheidbar, aber nie dominant; Reduce-Motion → reines 2D.
