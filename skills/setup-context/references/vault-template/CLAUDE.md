# CLAUDE.md – Vault-Regeln

Dieses Vault ist eine persoenliche Wissensdatenbank (Zweites Gehirn).

## SPIKES – Flat Index

`SPIKES.md` im Vault-Root ist der zentrale Wissens-Index. Format: JSON-Array.

```json
[
  {
    "path": "02 Projekte/Beispielnotiz.md",
    "summary": "Knapper 1-2-Satz-Verweis worum es geht",
    "keywords": ["kw1", "kw2", "kw3"],
    "projects": ["projekt-a"],
    "updated": "2026-01-01"
  }
]
```

**Wie SPIKES.md genutzt wird:**
- Beim Suchen nach Wissen zuerst SPIKES.md lesen
- Auf keywords oder summary matchen → nur Treffer im Volltext lesen
- Mehrere Treffer sind willkommen

**Wie Eintraege entstehen:**
- Skill `spike` ausfuehren nach dem Erstellen oder Aendern einer Notiz in `02 Projekte/`, `03 Bereiche/`, `04 Ressourcen/`
- Scope: nur diese drei Ordner werden indiziert. Inbox, Daily Notes, Archiv bleiben aussen vor.

## Vault-Struktur

- `00 Kontext/` — Persoenliches Profil, Schreibstil, Branding
- `01 Inbox/` — Schnelle Gedanken, Brain Dumps, Unsortiertes
- `02 Projekte/` — Aktive Projekte mit Endziel und Datum
- `03 Bereiche/` — Laufende Verantwortungsbereiche ohne Enddatum
- `04 Ressourcen/` — Referenzmaterial, Wissen, Sammlungen
- `05 Daily Notes/` — Tagebuch, was passiert ist (`YYYY-MM-DD.md`)
- `06 Archiv/` — Abgeschlossene Projekte, inaktive Bereiche
- `07 Anhaenge/` — Bilder, PDFs, Medien

## Regeln

- Nutze `[[Wikilinks]]` fuer Querverweise
- Halte Notizen atomar — eine Idee pro Notiz wo moeglich
- Neue Notizen ohne klaren Platz kommen in `01 Inbox/`
- YAML-Frontmatter fuer `tags`, `status`, `date`
- Dateinamen normal, mit Leerzeichen und Grossbuchstaben (`Beschreibender Name.md`)
- Vor dem Loeschen oder Ueberschreiben fragen
- Wenn der User sagt „merke dir das" oder „speichere das" — thematisch passend einsortieren

## Session-Routinen

### Bei Session-Start
1. `SPIKES.md` als Wissensindex lesen
2. `01 Inbox/Brain Dump.md` (falls vorhanden) auf neue Notizen pruefen

### Bei Session-Ende
1. Daily Note in `05 Daily Notes/YYYY-MM-DD.md` anbieten
2. Neue Erkenntnisse als Notizen festhalten
