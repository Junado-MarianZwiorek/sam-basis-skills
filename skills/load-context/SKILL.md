---
name: load-context
description: Laedt einen gespeicherten Session-Kontext aus SAVED-CONTEXT/savecontext.md und gibt eine strukturierte Zusammenfassung. Ermoeglicht nahtlosen Wiedereinstieg nach Kontextwechsel. Aufruf /load-context [Projektpfad] [bit]
argument-hint: [Projektpfad] [bit] - "bit" = Quick-Load nur savecontext.md, keine Pflicht-Dateien/CLAUDE.md/SPIKES
allowed-tools: Read, Glob, Grep, Bash
---

# Load Context Skill

Du liest einen gespeicherten Arbeitskontext ein und bereitest den User auf den Wiedereinstieg vor. Pfade kommen aus der Plugin-Config — nicht hardcoded.

---

## Schritt 0: Config laden + Umgebung pruefen

**Config lesen:** `~/.claude/sam-basis-skills/config.json`

```bash
CONFIG="$HOME/.claude/sam-basis-skills/config.json"
test -f "$CONFIG" || { echo "Config fehlt — bitte /setup-context ausfuehren"; exit 1; }
```

Aus der Config merken:
- `WORKSPACE_ROOT`
- `VAULT_ROOT` (kann null)
- `SPIKES_INDEX_PATH` (kann null)

Flag setzen:
- `VAULT_AVAILABLE`: `VAULT_ROOT != null && file exists ${SPIKES_INDEX_PATH}` (true/false)

Auf fremden Rechnern ohne Vault laeuft der Skill trotzdem sauber durch — die SPIKES-Schritte werden uebersprungen.

---

## Schritt 1: Argumente parsen + Projektordner bestimmen

`$ARGUMENTS` kann zwei Tokens enthalten (Reihenfolge egal): einen Projektpfad und/oder das Schluesselwort `bit`.

- Token `bit` → Flag `BIT_MODE=true`, Token aus Argumentliste entfernen.
- Verbleibendes Token → Projektordner = `${WORKSPACE_ROOT}/{Token}`
- Kein verbleibendes Token → alle vorhandenen `SAVED-CONTEXT/savecontext.md` im Workspace suchen + als Auswahl zeigen.
- Falls savecontext.md fehlt: User informieren, nichts weiter tun.

**`BIT_MODE=true` (Quick-Load):**
- Schritt 3 komplett skippen (keine Pflicht-Dateien, keine Projekt-CLAUDE.md, kein SPIKES.md)
- Schritt 4: verkuerzte Ausgabe
- Schritt 5: am Ende melden `[BIT-MODE] Nur savecontext.md geladen`

---

## Schritt 2: savecontext.md einlesen

Lies `{Projektordner}/SAVED-CONTEXT/savecontext.md` vollstaendig.

---

## Schritt 3: Kritische Dateien laden

**Bei `BIT_MODE=true` komplett skippen** und direkt zu Schritt 4.

Lies die unter „Dateien fuer Session-Start" aufgelisteten **Pflicht-Dateien**.

**Zusaetzlich IMMER:** Pruefe ob `{Projektordner}/CLAUDE.md` existiert und lies sie ein, auch wenn nicht in Pflicht-Liste.

**SPIKES-Verweise in savecontext.md:** Falls unter „Kritische Erkenntnisse" Eintraege der Form `→ SPIKES-Keyword: [keyword]` stehen, NICHT sofort die verlinkte Vault-Notiz laden. Stattdessen `${SPIKES_INDEX_PATH}` einmalig einlesen, passende Summaries extrahieren. Volltext nur nachladen wenn der aktuelle Arbeitsschritt es konkret braucht.

**Cache:** Das hier eingelesene SPIKES.md in Schritt 3b wiederverwenden.

**Gate:** Bei `VAULT_AVAILABLE=false` den SPIKES-Teil komplett ueberspringen, keine Fehlermeldung.

---

## Schritt 3b: Projekt-relevante SPIKES-Summaries laden

**Nur wenn `VAULT_AVAILABLE && !BIT_MODE`. Sonst skippen.**

Ziel: relevantes Hintergrundwissen im Kopf haben, ohne Volltexte zu laden.

Vorgehen:

1. `${SPIKES_INDEX_PATH}` (JSON-Array) lesen, falls noch nicht in Schritt 3.
2. Projektname aus Schritt 1 als Suchbegriff.
3. Filter — zwei Stufen:
   - **Stufe 1 (exakt):** Eintrag hat `projects`-Feld und Projektname ist enthalten → Treffer.
   - **Stufe 2 (Heuristik-Fallback, nur fuer Eintraege ohne `projects` oder mit `projects: []`):** Treffer wenn:
     - Projektname (case-insensitive Substring) in `path`
     - oder in einem `keywords`
     - oder in `summary`
   - Eintraege mit `projects` das den Projektnamen NICHT enthaelt → kein Treffer (explizite Zuordnung schlaegt Heuristik).
4. Top 10 nach `updated` desc.
5. In Schritt 4 als Block `### Verfuegbares Hintergrundwissen (SPIKES)` ausgeben — pro Treffer EINE Zeile: `- [Pfad-Basename ohne .md] — [summary]`.
6. **Volltexte NICHT preemptiv laden.**

Wenn keine Treffer: Block weglassen.

---

## Schritt 4: Zusammenfassung ausgeben

**Bei `BIT_MODE=true`** — Mini-Ausgabe (max 8 Zeilen):

```
## [Projektname] – Kontext geladen [BIT]

**Stand:** [Datum] | **Status:** [Einzeiler]
**Naechster Schritt:** [konkret aus savecontext.md]

[BIT-MODE] Nur savecontext.md geladen — fuer volle Tiefe /load-context ohne bit erneut aufrufen.
```

**Bei normalem Modus:**

```
## [Projektname] – Kontext geladen

**Letzter Stand:** [Datum]
**Status:** [Einzeiler]

### Was zuletzt passiert ist
[2-3 Saetze]

### Wo wir stehen
[Funktioniert / offen]

### Was als Naechstes ansteht
1. [konkret]
2. [danach]

### Im Kopf behalten
[Kritische Erkenntnisse — max 3-5 Punkte]

### Verfuegbares Hintergrundwissen (SPIKES)
[Aus Schritt 3b: Top 10. Block weglassen wenn keine Treffer oder VAULT_AVAILABLE=false.]
```

---

## Schritt 5: Weitermachen anbieten

> Soll ich mit **[naechste Aufgabe]** weitermachen?

Falls TEMPORAER-Block in Workspace-CLAUDE.md noch auf dieses Projekt zeigt, weise darauf hin dass er nach Abschluss entfernt werden sollte.

---

## Hinweise

- Antworte auf Deutsch
- Halte die Zusammenfassung kompakt — kein Roman
- Wenn referenzierte Dateien nicht mehr existieren: darauf hinweisen
