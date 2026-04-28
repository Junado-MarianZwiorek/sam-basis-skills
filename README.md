# sam-basis-skills

Plugin fuer Claude Code mit vier Skills fuer Session-Kontext-Management und Vault-Indexierung — plus optional vier externe Skills auf Wunsch.

## Skills

| Skill | Zweck |
|---|---|
| `/save-context` | Aktuellen Arbeitskontext als `savecontext.md` im Projekt speichern (kumulativ) |
| `/load-context` | Gespeicherten Kontext laden + projekt-relevante Vault-Spikes mitbringen |
| `/spike` | Vault-Notiz indizieren (Frontmatter + zentraler `SPIKES.md`-Index) |
| `/setup-context` | Erst-Setup-Wizard — fragt Pfade ab, schreibt Config |

## Installation

1. Plugin installieren (z.B. `/plugin install <pfad-oder-url>`)
2. **`/setup-context` einmal ausfuehren** — der Wizard fragt:
   - Workspace-Root (Ordner mit Projekt-Unterordnern)
   - Wissens-Vault: vorhanden / nicht vorhanden / neu anlegen (Minimal-Template)
   - Optionale Workspace-`CLAUDE.md` fuer TEMPORAER-Bloecke
   - Optionale externe Skills (siehe unten)
3. Fertig. Die anderen Skills lesen die Config automatisch.

## Konzept

**Drei-System-Persistenz** zwischen Sessions:

```
Session-Wissen     →  savecontext.md  (im Projektordner)
Domain-Wissen      →  Vault-Notizen   (im Wissens-Vault)
Wissens-Index      →  SPIKES.md       (im Vault-Root)
```

`save-context` schreibt Session-Wissen nach `{projekt}/SAVED-CONTEXT/savecontext.md`. Wenn ein Vault konfiguriert ist, lagert es persistentes Domain-Wissen autonom dorthin aus und ersetzt es in der savecontext.md durch Kurz-Referenzen (`→ SPIKES: keyword`).

`load-context` liest savecontext.md zurueck und zieht beim Start die Top-10 projekt-relevanten Vault-Spikes als Hintergrundwissen mit rein.

`spike` haelt SPIKES.md aktuell und berechnet automatisch Verwandte-Notizen-Verlinkungen.

## Bit-Modus

`/save-context bit` und `/load-context bit` — Quick-Save/Load bei knappem Token-Budget. Skippt Spike-Sync und Verzeichnisscan. Der Marker `[BIT-PENDING-SYNC]` wird in der savecontext.md gesetzt; beim naechsten vollen `/save-context` wird der Spike-Sync nachgezogen.

## Optionale externe Skills

Beim Setup kann der Wizard zusaetzliche Skills via Git-Clone in `~/.claude/skills/` ablegen:

| Skill | Zweck | URL |
|---|---|---|
| `html` | HTML-Praesentationen | https://github.com/zarazhangrui/frontend-slides |
| `prompt-master` | Prompt-Optimierung | https://github.com/nidhinjs/prompt-master |
| `obsidian-skills` | Vault-Tools (Markdown, Bases, JSON-Canvas) | https://github.com/kepano/obsidian-skills |
| `remotion-best-practices` | React-Video Best Practices | https://skills.sh/remotion-dev/skills/remotion-best-practices |

Diese Skills sind **nicht Teil** dieses Plugins — sie werden vom Setup-Wizard auf Wunsch geclonet. Voraussetzung: `git` ist installiert.

## Config

Der Setup-Wizard schreibt:

`~/.claude/sam-basis-skills/config.json`

```json
{
  "version": 1,
  "workspace_root": "C:\\Users\\foo\\code",
  "vault_root": "C:\\Users\\foo\\notes",
  "spikes_index_path": "C:\\Users\\foo\\notes\\SPIKES.md",
  "main_claude_md": "C:\\Users\\foo\\code\\CLAUDE.md",
  "vault_type": "obsidian",
  "installed_optional_skills": ["html", "prompt-master"]
}
```

`vault_root`, `spikes_index_path` und `main_claude_md` koennen `null` sein. Die Skills passen ihr Verhalten entsprechend an (z.B. ueberspringen Spike-Sync wenn kein Vault).

## Re-Run

`/setup-context` ist jederzeit re-runfaehig — bestehende Werte werden als Defaults angeboten, einzelne Felder koennen geaendert werden.

## Vault-Bauplan (Minimal-Template)

Wenn der Setup-Wizard einen neuen Vault anlegt:

```
{vault_root}/
├── 00 Kontext/
├── 01 Inbox/
├── 02 Projekte/
├── 03 Bereiche/
├── 04 Ressourcen/
├── 05 Daily Notes/
├── 06 Archiv/
├── 07 Anhaenge/
├── .obsidian/        (leer — Obsidian erkennt das Vault)
├── CLAUDE.md         (Vault-Regeln + Spike-Workflow)
└── SPIKES.md         ([])
```

Nur `02 Projekte/`, `03 Bereiche/` und `04 Ressourcen/` werden vom Spike-Skill indiziert.

## Lizenz

Eigenentwicklung. Externe Skills (siehe oben) haben jeweils eigene Lizenzen.
