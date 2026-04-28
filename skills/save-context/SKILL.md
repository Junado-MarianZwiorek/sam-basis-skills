---
name: save-context
description: Speichert den aktuellen Session-Kontext als savecontext.md im Projektordner (SAVED-CONTEXT/). Erfasst Status, abgeschlossene Schritte, naechste Schritte und kritische Erkenntnisse. Ziel - nahtloser Wiedereinstieg in neuer Session. Aufruf /save-context [Projektpfad] [bit]
argument-hint: [Projektpfad] [bit] - "bit" = Quick-Save bei knappem Token-Budget (skippt Spike-Sync + Verzeichnisscan)
allowed-tools: Read, Glob, Grep, Bash, Write, Edit
---

# Save Context Skill

Du speicherst den aktuellen Arbeitskontext so, dass eine neue Claude-Session nahtlos weitermachen kann. Pfade kommen aus der Plugin-Config — nicht hardcoded.

---

## Schritt 0: Config laden + Umgebung pruefen

**Config lesen:** `~/.claude/sam-basis-skills/config.json`

```bash
CONFIG="$HOME/.claude/sam-basis-skills/config.json"
test -f "$CONFIG" || { echo "Config fehlt — bitte /setup-context ausfuehren"; exit 1; }
```

Aus der Config lesen und als Variablen merken:
- `WORKSPACE_ROOT` (Pflicht)
- `VAULT_ROOT` (kann null sein)
- `SPIKES_INDEX_PATH` (kann null sein)
- `MAIN_CLAUDE_MD` (kann null sein)

Dann Flags setzen:
- `VAULT_AVAILABLE`: `VAULT_ROOT != null && file exists ${SPIKES_INDEX_PATH}` (true/false)
- `SPIKE_AVAILABLE`: existiert eine Skill-Datei unter `~/.claude/skills/spike/` ODER im aktuellen Plugin-Skill-Pfad? Pruefe **case-insensitive** sowohl `SKILL.md` als auch `skill.md`. (true/false)

**Regel:** Vault kann ohne Spike-Skill da sein (Lesen OK, Schreiben unmoeglich → 3.5/3.6 trotzdem skippen). Spike-Skill ohne Vault macht keinen Sinn.

Effektive Spike-Sync-Bedingung: `VAULT_AVAILABLE && SPIKE_AVAILABLE && !BIT_MODE`.

**Falls Config fehlt:** Skill bricht ab mit Hinweis `Bitte erst /setup-context ausfuehren`.

---

## Schritt 1: Argumente parsen + Projektordner bestimmen

`$ARGUMENTS` kann zwei Tokens enthalten (Reihenfolge egal): einen Projektpfad und/oder das Schluesselwort `bit`.

- Token `bit` → Flag `BIT_MODE=true`, Token aus Argumentliste entfernen.
- Verbleibendes Token → Projektordner = `${WORKSPACE_ROOT}/{Token}`
- Kein verbleibendes Token → User fragen welches Projekt gespeichert werden soll.
- Pruefe ob Ordner existiert.

**`BIT_MODE=true` (Quick-Save bei knappem Token-Budget):**
- Schritt 2 nur ausfuehren wenn savecontext.md < 200 Zeilen
- Schritt 3b (Verzeichnisstruktur) komplett skippen
- Schritt 3.5, 3.6, 3.7 (Vault-Auslagerung, Spike-Sync, SPIKES-Match) komplett skippen
- Schritt 4: verkuerztes Mini-Template (siehe unten), Ziel < 40 Zeilen
- Schritt 5 (CLAUDE.md TEMPORAER-Block) trotzdem ausfuehren
- Schritt 6: explizit melden `[BIT-MODE] Spike-Sync + Vault-Auslagerung ausstehend → bitte naechste Session /save-context ohne bit nachziehen.`

---

## Schritt 2: Bestehenden Kontext einlesen

Falls vorhanden: `{Projektordner}/SAVED-CONTEXT/savecontext.md` einlesen. Das ist der Vorgaenger-Kontext (kumulative Historie).

**Bit-Marker-Erkennung:** Falls Header `[BIT-PENDING-SYNC]` enthaelt, Flag `BIT_SYNC_PENDING=true` setzen. Auswirkung im normalen Modus:
- 3.5/3.6/3.7 laufen normal — ziehen jetzt nach was Bit uebersprungen hat
- Beim Schreiben in Schritt 4 den Marker NICHT wieder eintragen
- Schritt 6 meldet zusaetzlich: „Bit-Schuld nachgezogen"

---

## Schritt 3: Kontext sammeln

### a) Git-Status (falls Git-Repo)
```bash
git log --oneline -20
git status
git diff --stat
```

### b) Verzeichnisstruktur (bei `BIT_MODE=true` skippen)
```bash
find {Projektordner} -type f -not -path "*/node_modules/*" -not -path "*/__pycache__/*" -not -path "*/.git/*" -not -path "*/SAVED-CONTEXT/*" | head -80
```

### c) Geaenderte Dateien
Aus Konversations-Kontext und git diff identifizieren.

### d) Session-Inhalte
- Was wurde erledigt?
- Was ist der aktuelle Stand?
- Was steht als Naechstes an?
- Welche kritischen Erkenntnisse gab es?

### e) Vorgaenger-Kontext
Falls vorhanden: abgeschlossene Schritte kumulativ uebernehmen.

---

## Schritt 3.5: Domain-Wissen in Vault auslagern (autonom, sparsam)

**Nur wenn `VAULT_AVAILABLE && SPIKE_AVAILABLE && !BIT_MODE`. Sonst skippen.**

Dauerhaftes Domain-Wissen gehoert in den Vault, nicht in savecontext.md. Diese Stufe laeuft autonom.

### 3.5a Kandidaten identifizieren (harter Filter)

Vault-wuerdig wenn **alle drei**:

1. **Persistent** — gilt auch in uebernaechster Session noch (kein Session-Zustand)
2. **Nicht ableitbar** — steht weder im Code noch in CLAUDE.md
3. **Konkret** — API-Quirk, Schema-Regel, Endpoint-Verhalten, Architektur-Invariante. Keine weichen Praeferenzen.

Im Zweifel: **nicht auslagern**.

### 3.5b Dupe-Check gegen SPIKES.md

Fuer jeden Kandidaten **vor** dem Anlegen:

1. `${SPIKES_INDEX_PATH}` lesen (JSON-Array, einmalig cachen)
2. Keyword-Match: Kandidat-Kernbegriffe gegen alle `keywords`-Arrays
3. **Treffer (≥2 Keyword-Overlap):** bestehende Notiz unter `path` lesen
   - Neue Info ergaenzt → Notiz per `Edit` aktualisieren (additiv)
   - Widerspricht → aktualisieren + alte Passage markieren/ueberschreiben
   - Bereits enthalten → nichts tun
4. **Kein Treffer:** atomare Notiz in passendem Unterordner anlegen (default `${VAULT_ROOT}/04 Ressourcen/<Thema>/`)

### 3.5c Folgeschritte

Fuer jede neu/geaenderte Notiz: Schritt 3.6 (Spike-Sync) gilt automatisch.

In savecontext.md Eintrag durch Kurz-Referenz ersetzen: `- [Kurztitel] → SPIKES: [keyword]`.

---

## Schritt 3.6: Spike-Sync fuer geaenderte Vault-Notizen (Pflicht)

**Nur wenn `VAULT_AVAILABLE && SPIKE_AVAILABLE && !BIT_MODE`. Sonst skippen + in Schritt 6 melden.**

Sammle alle Notizen in `${VAULT_ROOT}/02 Projekte/`, `${VAULT_ROOT}/03 Bereiche/`, `${VAULT_ROOT}/04 Ressourcen/` die in dieser Session erstellt oder geaendert wurden. Kein Scope: `01 Inbox/`, `05 Daily Notes/`, `00 Kontext/`, `06 Archiv/`, `07 Anhaenge/`.

Fuer jede Notiz:

1. `/spike {relativer-pfad}` aufrufen → Frontmatter `spikes.summary`+`spikes.keywords`+`spikes.projects` aktualisieren, SPIKES.md neu schreiben.
2. Verwandte Notizen neu rechnen:
   - SPIKES.md neu lesen, Keyword-Overlap zu allen anderen berechnen
   - Score ≥ 2 → Link; Score < 2 → kein Link; Top 5 absteigend
   - Block `## Verwandte Notizen` mit Kommentar `<!-- auto-generated by spike – nicht manuell bearbeiten -->` neu schreiben
   - Bidirektional: jede Gegenseite ebenfalls neu rechnen (manuelle Bloecke ohne Auto-Kommentar nicht ueberschreiben, nur additiv)

Wenn keine Vault-Notizen in Scope geaendert: Schritt ueberspringen, in Schritt 6 melden „Kein Spike-Sync noetig".

---

## Schritt 3.7: Kritische Erkenntnisse gegen SPIKES.md matchen

**Nur wenn `VAULT_AVAILABLE && !BIT_MODE`. Sonst skippen.** (SPIKE_AVAILABLE nicht noetig — nur Lesen.)

**Ziel:** Doppellungen Vault ↔ savecontext vermeiden.

Vorgehen:
1. Sammle die rohen „Kritische Erkenntnisse"-Punkte aus 3d (vor dem Schreiben)
2. SPIKES.md-Cache verwenden (oder einmalig lesen)
3. Pro Erkenntnis-Punkt:
   - Kernbegriffe extrahieren
   - Gegen alle SPIKES-Eintraege matchen — Treffer wenn ≥2 Keyword-Overlap ODER klarer Substring-Overlap zwischen Erkenntnis und Spike-`summary`
   - **Treffer:** Punkt durch `- [Kurztitel] → SPIKES: [erstes_match_keyword]` ersetzen
   - **Kein Treffer:** unveraendert lassen
4. **Konservative Schwelle** — im Zweifel nicht ersetzen
5. Anzahl ersetzter Punkte fuer Schritt 6 merken

**Verhaeltnis zu 3.5:** 3.5 schreibt **neue** Notizen. 3.7 ersetzt **bestehende** Spike-Treffer durch Referenzen. Reihenfolge: erst 3.5, dann 3.7.

---

## Schritt 4: savecontext.md schreiben

Erstelle `{Projektordner}/SAVED-CONTEXT/` falls nicht existent.

### Bei `BIT_MODE=true` — Mini-Template (Ziel < 40 Zeilen)

```markdown
# Save Context – [Projektname] [BIT]

**Gespeichert:** [YYYY-MM-DD] | **Status:** [BIT-PENDING-SYNC] [Einzeiler] | **Naechste Aufgabe:** [konkret]

> ⚠ BIT-SAVE — Spike-Sync + Vault-Auslagerung ausstehend.

## Diese Session (kurz)
- [Stichpunkt]

## Naechster Schritt
[1-3 Zeilen]

## Kritische Erkenntnisse (max 3)
- [nur wenn neu und nicht im Code]

## Dateien fuer Session-Start
**Pflicht:** `{Projektpfad}/CLAUDE.md` (falls vorhanden)
```

Vorgaenger-Datei NICHT vollstaendig uebernehmen — nur „Sessions 1-N: ..." als Einzeiler.

### Bei normalem Modus — volles Template

```markdown
# Save Context – [Projektname]

**Gespeichert:** [YYYY-MM-DD] | **Status:** [Einzeiler] | **Naechste Aufgabe:** [konkret]

---

## Abgeschlossene Schritte

**Fruehere Sessions (1-N):** [EIN Einzeiler der alles vor letzter Session zusammenfasst]

**[Datum] Session N (diese Session):**
- [Stichpunkt, kein Narrativ]

## Aktueller Stand

### Funktioniert
- [Stichpunkte]

### Nicht getestet / Offen
- [Stichpunkte]

### Bekannte Stoerungen
- [Stichpunkte oder „unveraendert aus Session X"]

## Naechste Schritte

1. **[SOFORT]** ...
2. ...

## Kritische Erkenntnisse

[NUR Dinge die NICHT aus Code/Git hervorgehen. Stichpunkte.]

- [Erkenntnis 1-2 Zeilen]

## Dateien fuer Session-Start

**Pflicht:** `{Projektpfad}/CLAUDE.md` (falls vorhanden), [weitere kritische Dateien]
**Bei Bedarf:** [Optionale Dateien]
```

**Pflicht-Liste-Regel:** Wenn `{Projektordner}/CLAUDE.md` existiert, IMMER als ersten Pflicht-Eintrag nach savecontext.md eintragen.

**Kontextfenster-Disziplin:**
- Zielgroesse: ~60-80 Zeilen / ~600 Tokens
- Stichpunkte, keine Prosa
- „Abgeschlossene Schritte": nur letzte Session im Detail
- „Kritische Erkenntnisse": nur was NICHT im Code steht
- KEIN Verzeichnisbaum
- KEIN Setup/Credentials-Block

---

## Schritt 5: Workspace-CLAUDE.md TEMPORAER-Block aktualisieren (autonom)

**Nur wenn `MAIN_CLAUDE_MD != null && file exists`. Sonst skippen + in Schritt 6 melden.**

Diese Stufe laeuft ohne Rueckfrage.

Lies `${MAIN_CLAUDE_MD}`.

### Falls TEMPORAER-Block fuer dieses Projekt existiert
Aktualisiere mit neuen Pfaden + naechstem Schritt.

### Falls kein Block existiert
Fuege neuen Block ein (nach letztem TEMPORAER-Block oder nach `### Bei Session-Start`):

```markdown
### TEMPORAER: [Projektname] Kontext laden (nach Abschluss fragen ob dieser Block geloescht werden soll!)
Beim naechsten Session-Start diese Datei einlesen, um am [Projektname] Projekt weiterzuarbeiten:
1. `{Projektpfad}/SAVED-CONTEXT/savecontext.md` – Gesamtkontext
[2. Weitere kritische Dateien falls noetig]
Erster Schritt: [Naechste Aufgabe aus savecontext.md]
Danach den User fragen: "Soll ich den temporaeren Block aus CLAUDE.md wieder entfernen?"
```

---

## Schritt 6: Bestaetigung

Kurze Zusammenfassung:
- Wo die Datei gespeichert wurde
- Was als naechster Schritt hinterlegt ist
- Ob der CLAUDE.md-Block aktualisiert/erstellt wurde (falls `MAIN_CLAUDE_MD` gesetzt)
- **Bit-Schuld-Status:** falls `BIT_SYNC_PENDING=true` war: „Bit-Schuld nachgezogen". Sonst weglassen.
- **Phase B Match-Ergebnis:** falls 3.7 lief: „X von Y Erkenntnissen via SPIKES-Referenz ersetzt".
- **Spike-Sync-Ergebnis:** einer der Faelle:
  - `[BIT-MODE] Spike-Sync ausstehend` (BIT_MODE)
  - „Spike-Sync uebersprungen (Vault nicht gefunden)" (VAULT_AVAILABLE=false)
  - „Spike-Sync uebersprungen (Skill /spike nicht installiert)" (SPIKE_AVAILABLE=false)
  - welche Vault-Notizen gespiket wurden
  - „keine Aenderungen in 02/03/04"

---

## Hinweise

- Antworte auf Deutsch
- CLAUDE.md-Update in Schritt 5 laeuft ohne Rueckfrage (Skill-Ausnahme)
- Datei wird bei jedem Aufruf ueberschrieben — kein Archiv, Git reicht
