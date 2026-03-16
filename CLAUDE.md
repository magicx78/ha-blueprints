# ha-blueprints — Claude Code Framework

## Startup-Ritual (IMMER beim Start)

```bash
# 1. Kontext laden
cat PROGRESS.md 2>/dev/null || echo "Keine PROGRESS.md"
git log --oneline -5 2>/dev/null || echo "Kein Git-Repo"

# 2. Blueprint-Status scannen
find blueprints/ -name "*.yaml" 2>/dev/null | sort

# 3. Coordinator aktivieren
```
Coordinator zeigt Diagnose + fragt: *"Was soll heute passieren?"*

---

## Modell-Strategie — Haiku first

```bash
# Standard setzen
claude config set model claude-haiku-4-5-20251001
```

| Phase | Modell |
|-------|--------|
| Diagnose, Planung, Validierung | Haiku 4.5 |
| Komplexe Blueprint-Logik | Sonnet (nur wenn nötig) |

---

## Projekt-Struktur

```
ha-blueprints/
├── blueprints/
│   ├── automation/     ← Automatisierungs-Blueprints
│   ├── script/         ← Script-Blueprints
│   └── template/       ← Template-Blueprints
├── .github/
│   └── workflows/
│       └── validate.yml  ← CI: Validierung bei jedem Push
├── CLAUDE.md
├── PROGRESS.md
└── README.md
```

---

## Agent-Architektur

```
coordinator      ← IMMER erster Kontakt
                   Diagnostiziert + plant mit Nutzer
     │
     ├──► validator    ← YAML + HA-Schema prüfen
     ├──► developer    ← Blueprints schreiben / fixen
     └──► publisher    ← GitHub Repo + CI + Release
```

---

## Blueprint Schema — Pflichtfelder

```yaml
blueprint:
  name: "Kurzer beschreibender Name"
  description: |
    Was macht dieser Blueprint?
    Was braucht der Nutzer?
  domain: automation  # automation | script | template
  author: "magicx78"
  homeassistant:
    min_version: "2024.1.0"
  input:
    my_input:
      name: "Input Name"
      description: "Was erwartet dieser Input?"
      selector:
        entity:
          domain: sensor
```

## Publish-Wege

| Weg | Voraussetzung |
|-----|--------------|
| GitHub Import-Button | Öffentliches Repo, raw YAML-URL |
| HA Community Forum | Thread + raw URL einstellen |
| Awesome HA Blueprints | Qualitätsprüfung + PR |

---

## Commit-Konventionen

```
feat:     neuer Blueprint
fix:      Blueprint-Fehler behoben
docs:     README / Beschreibung
chore:    CI, PROGRESS.md
release:  stabile Version veröffentlicht
```
