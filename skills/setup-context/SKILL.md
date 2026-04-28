---
name: setup-context
description: Erst-Setup-Wizard fuer das sam-basis-skills Plugin. Fragt Workspace-Pfad, Vault-Pfad und optionale Skills ab, schreibt Config nach ~/.claude/sam-basis-skills/config.json. Aufruf /setup-context. Bei wiederholtem Aufruf werden bestehende Werte als Default angeboten.
argument-hint: (keine Argumente — interaktiv)
allowed-tools: Read, Write, Edit, Bash, Glob
---

# Setup Context Skill

Du fuehrst den User Schritt fuer Schritt durch die Erst-Einrichtung des sam-basis-skills Plugins. Ergebnis: eine Config-Datei unter `~/.claude/sam-basis-skills/config.json`, die alle anderen Skills (save-context, load-context, spike) lesen.

**Sprache:** Deutsch. Keine Anrede festlegen — der User entscheidet selbst.

---

## Schritt 0: Bestehende Config pruefen

Pruefe via Bash:

```bash
test -f "$HOME/.claude/sam-basis-skills/config.json" && cat "$HOME/.claude/sam-basis-skills/config.json"
```

- **Existiert:** Inhalt anzeigen, fragen ob neu eingerichtet werden soll oder nur einzelne Felder geaendert werden. Bestehende Werte als Defaults verwenden.
- **Existiert nicht:** Voll-Wizard starten.

---

## Schritt 1: Workspace-Root erfragen

Frage:

> **1) Wo liegt dein Workspace-Root?**
> Das ist der Ordner mit deinen Projekt-Unterordnern. Beispiel: `C:\Users\foo\code` oder `/home/foo/projects`.
>
> Default: aktueller Arbeitsordner (cwd).

Pruefe via Bash dass der Pfad existiert. Falls nicht — nachfragen.

---

## Schritt 2: Vault-Pfad erfragen

Frage:

> **2) Hast du eine Wissensdatenbank (Obsidian-Vault o.ae.)?**
>
> - `〔J〕` Ja, Pfad eingeben
> - `〔N〕` Nein, ueberspringen (Skills laufen ohne Vault-Funktionen)
> - `〔A〕` Nein, aber bitte einen anlegen (Minimal-Template)

### Bei `〔J〕`
Pfad einlesen, pruefen ob existiert. Pruefen ob `SPIKES.md` schon im Root liegt — falls ja, melden „SPIKES-Index gefunden". Falls nein, anbieten einen leeren `SPIKES.md` (`[]`) anzulegen.

### Bei `〔N〕`
`vault_root: null` in Config eintragen. Schritt 3 ueberspringen.

### Bei `〔A〕`
Pfad fragen wo der Vault angelegt werden soll. Dann:

1. Ordner anlegen
2. Standard-Unterordner anlegen: `00 Kontext`, `01 Inbox`, `02 Projekte`, `03 Bereiche`, `04 Ressourcen`, `05 Daily Notes`, `06 Archiv`, `07 Anhaenge`
3. Template-Dateien aus `${CLAUDE_PLUGIN_ROOT}/skills/setup-context/references/vault-template/` kopieren:
   - `CLAUDE.md` ins Vault-Root
   - `SPIKES.md` ins Vault-Root (Inhalt: `[]`)
4. Leeren `.obsidian/` Ordner anlegen (Obsidian erkennt das Vault dann automatisch)

---

## Schritt 3: Workspace-CLAUDE.md erfragen (optional)

Falls der User eine zentrale Workspace-`CLAUDE.md` mit TEMPORAER-Bloecken benutzt (siehe save-context Schritt 5):

> **3) Hast du eine zentrale CLAUDE.md im Workspace-Root, in der save-context die TEMPORAER-Bloecke pflegen soll?**
>
> - `〔J〕` Ja, Pfad: ... (Default: `{workspace_root}/CLAUDE.md`)
> - `〔N〕` Nein, ueberspringen

Bei `〔J〕`: pruefen ob Datei existiert. Falls nicht: anbieten anzulegen.

---

## Schritt 4: Optionale externe Skills anbieten

Lies `${CLAUDE_PLUGIN_ROOT}/skills/setup-context/references/optional-skills.json`:

```json
[
  {"name": "html", "url": "https://github.com/zarazhangrui/frontend-slides", "desc": "HTML-Praesentationen aus Markdown/PPT"},
  {"name": "prompt-master", "url": "https://github.com/nidhinjs/prompt-master", "desc": "Prompt-Optimierung fuer LLMs"},
  {"name": "obsidian-skills", "url": "https://github.com/kepano/obsidian-skills", "desc": "Vault-Tools (Markdown, Bases, JSON-Canvas)"},
  {"name": "remotion-best-practices", "url": "https://skills.sh/remotion-dev/skills/remotion-best-practices", "desc": "React-Video Best Practices"}
]
```

Liste anzeigen mit Checkboxen:

> **4) Welche zusaetzlichen Skills moechtest du installieren?**
>
> | | Skill | Zweck |
> |---|---|---|
> | `[ ]` | html | HTML-Praesentationen aus Markdown/PPT |
> | `[ ]` | prompt-master | Prompt-Optimierung fuer LLMs |
> | `[ ]` | obsidian-skills | Vault-Tools |
> | `[ ]` | remotion-best-practices | React-Video Best Practices |
>
> Antworte mit den Namen oder `alle` / `keine`.

Fuer jeden ausgewaehlten Skill:

```bash
mkdir -p "$HOME/.claude/skills/{name}"
git clone {url} "$HOME/.claude/skills/{name}"
```

Falls Git nicht installiert: Fehler ausgeben + Hinweis manuelles Clonen.

---

## Schritt 5: Config schreiben

Schreibe `~/.claude/sam-basis-skills/config.json`:

```json
{
  "version": 1,
  "workspace_root": "...",
  "vault_root": "..." | null,
  "spikes_index_path": "..." | null,
  "main_claude_md": "..." | null,
  "vault_type": "obsidian" | "none",
  "installed_optional_skills": ["html", "prompt-master"]
}
```

`spikes_index_path` = `{vault_root}/SPIKES.md` falls Vault da ist, sonst `null`.

Vor dem Schreiben Verzeichnis sicherstellen: `mkdir -p ~/.claude/sam-basis-skills`.

---

## Schritt 6: Bestaetigung

Zeige dem User:

```
✅ Setup abgeschlossen.

Config: ~/.claude/sam-basis-skills/config.json

Workspace: {pfad}
Vault:     {pfad oder "—"}
CLAUDE.md: {pfad oder "—"}

Installierte Zusatz-Skills: {liste oder "keine"}

Aktive Plugin-Skills:
- /save-context  – Session-Kontext speichern
- /load-context  – Session-Kontext laden
- /spike         – Vault-Notiz indizieren (nur mit Vault)

Bei Aenderungen: einfach /setup-context erneut aufrufen.
```

---

## Hinweise

- Config-Pfad ist absolut, plattformuebergreifend: `~/.claude/sam-basis-skills/config.json` (Windows: `%USERPROFILE%\.claude\sam-basis-skills\config.json`)
- Auf Windows: Bash via Git-Bash/WSL-aehnliche Shell. Falls `$HOME` nicht gesetzt: `%USERPROFILE%` als Fallback.
- Pfade mit Leerzeichen IMMER quoten.
- Falls Git fuer optionale Skills fehlt: zeige manuelle Clone-Befehle, brich Setup nicht ab.
