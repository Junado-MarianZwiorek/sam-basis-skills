---
name: spike
description: Generiert oder aktualisiert einen SPIKES.md-Eintrag fuer eine Vault-Notiz. Pflegt Frontmatter (summary, keywords, projects) und den Flat-Index. Aufruf /spike [pfad]
argument-hint: [pfad relativ vom Vault-Root, z.B. "02 Projekte/Beispiel.md"]
allowed-tools: Read, Edit, Write, Bash
---

# Spike — Flat Index Generator

Generiert einen SPIKES.md-Eintrag fuer eine Vault-Notiz und haelt den zentralen Wissens-Index aktuell. Pfade kommen aus der Plugin-Config.

---

## Schritt 0: Config laden

```bash
CONFIG="$HOME/.claude/sam-basis-skills/config.json"
test -f "$CONFIG" || { echo "Config fehlt — bitte /setup-context ausfuehren"; exit 1; }
```

Aus der Config:
- `VAULT_ROOT` (Pflicht — falls null: Skill bricht ab mit „Kein Vault konfiguriert")
- `SPIKES_INDEX_PATH` (Pflicht)

Alle weiteren Pfade in dieser Anleitung sind **relativ zu `${VAULT_ROOT}`** (ohne fuehrenden Slash).

---

## Aufruf

- Ohne Argument: Pfad aus aktueller Notiz im Kontext verwenden
- Mit Argument: relativer Pfad ab `${VAULT_ROOT}`

Beispiel: `spike 02 Projekte/Beispiel.md` → operiert auf `${VAULT_ROOT}/02 Projekte/Beispiel.md`

---

## Prozess

### 1. Notiz lesen

Lies die Notiz vollstaendig. Lies auch verlinkte Notizen falls noetig fuers Verstaendnis.

### 2. Spikes generieren

**summary** — knackiger Verweis, kein Wissensdump. Ziel: Leser entscheidet in 2 Sekunden „lese ich oder nicht".

Harte Regeln:
- **Max. 200 Zeichen** (hart — kuerzen statt umformulieren)
- **Max. 2 Saetze**: (1) Mechanik/Kern-Regel, (2) wichtigster Fallstrick. Zweiter Satz optional.
- **Verboten** in Summary:
  - konkrete IDs, SKUs, Preise, Bestaende
  - Datumsangaben („verifiziert am…", „Stand 2026-…")
  - Test-Werte, Beispielzahlen, Korrektur-Historie
  - Zaehlungen von Zeilen/Eintraegen
  - Aufzaehlung aller Endpoints/Fallstricke (Keywords machen den Job)
- **Erlaubt**: Fachbegriffe, Systemnamen, Kern-Mechanik in einem Satz
- **Generisch vermeiden**: „Diese Notiz beschreibt…", „Uebersicht ueber…" → direkt mit der Sache anfangen

Gut: „CE PATCH /v2/products/extra-data setzt Custom-Field-Werte, aber Last-Write-Wins ueberschreibt beim naechsten Feed-Lauf."

**keywords** — 6-8 flache Begriffe. Regeln:
- So wie der Begriff in der Notiz lebt (`k_Grandparent` bleibt `k_Grandparent`)
- Systemname, Konzept, Fachbegriff, Tool — gemischt OK
- Keine generischen Woerter: Notiz, System, Prozess, wichtig, update
- Sprache: wie im Kontext (DE/EN gemischt OK)

Gut: `run1_tags`, `k_Grandparent`, `CE-Huelle`, `ChannelEngine`, `Spedition-Filter`, `GraphQL`
Schlecht: `shopify`, `api`, `automation`

**projects** — Liste der Projektordner-Namen (relativ zu `${WORKSPACE_ROOT}`) zu denen die Notiz fachlich gehoert. Ermoeglicht load-context exakten Projektfilter statt Keyword-Heuristik.

Regeln:
- Genau die Projektordner-Namen verwenden
- Mehrfach-Zuordnung erlaubt (`[katana, shopify-api]`)
- **Leere Liste `[]`** wenn projektuebergreifend (Heuristik-Fallback greift dann in load-context)
- Im Zweifel: leer lassen statt falsch zuordnen

Gut: `[katana, shopify-api]` / `[google-feed-bkt]` / `[]`

### 3. Frontmatter der Notiz aktualisieren

Bestehende Felder (`tags`, `status`, `date`, etc.) unveraendert lassen. Hinzufuegen oder aktualisieren:

```yaml
---
spikes:
  summary: "Ein praegnanter Satz"
  keywords: [kw1, kw2, kw3, kw4, kw5, kw6]
  projects: [projekt-a, projekt-b]
---
```

Falls kein Frontmatter existiert: neu anlegen am Dateianfang.

### 4. SPIKES.md aktualisieren

`${SPIKES_INDEX_PATH}` ist die Index-Datei. Format: JSON-Array.

Beispiel-Eintrag:
```json
{
  "path": "02 Projekte/Beispiel.md",
  "summary": "Knapper Verweis worum es geht",
  "keywords": ["kw1", "kw2", "kw3"],
  "projects": ["projekt-a"],
  "updated": "2026-01-01"
}
```

`projects`-Feld 1:1 aus Frontmatter uebernehmen. Bei Migration alter Eintraege ohne `projects`: leeres Array.

Vorgehen:
1. SPIKES.md lesen
2. Eintrag fuer `path` suchen
3. Existiert → ersetzen (summary + keywords + projects + updated)
4. Existiert nicht → ans Ende anhaengen
5. Als valides JSON zurueckschreiben
6. `updated` = heutiges Datum `YYYY-MM-DD`

---

## Scope — nur diese Ordner erhalten Spikes

- `02 Projekte/`
- `03 Bereiche/`
- `04 Ressourcen/`

Fuer `01 Inbox/`, `05 Daily Notes/`, `00 Kontext/`, `06 Archiv/`, `07 Anhaenge/`: ablehnen mit Hinweis.

---

## Abschluss-Feedback

Kurz melden:
- Pfad der Notiz
- Generierte keywords (zur Kontrolle)
- Eintrag in SPIKES.md neu erstellt oder aktualisiert
