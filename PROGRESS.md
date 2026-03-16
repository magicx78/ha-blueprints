# PROGRESS.md — ha-blueprints

## Ziel
Sammlung qualitativ hochwertiger Home Assistant Blueprints entwickeln,
validieren und auf GitHub veröffentlichen.

---

## Blueprints — Übersicht

| Blueprint | Datei | Domain | Status | Notizen |
|-----------|-------|--------|--------|---------|
| Camera Health (Frigate FPS + Unavailable + Pulse Ping) | blueprints/automation/cam_active.yaml | automation | valide | author + min_version 2024.6.0 nachgetragen |
| Presence & Light v2 | blueprints/automation/presence_light.yaml | automation | valide | author + min_version 2024.6.0 + Beschreibung nachgetragen |
| Tuer offen Alarm Pro v4 | blueprints/automation/tuer_alarm_pro.yaml | automation | valide | author nachgetragen; min_version 2024.10.0 war bereits vorhanden |
| Automation Log Viewer | blueprints/automation/log_viewer.yaml | automation | valide | author + min_version 2024.6.0 nachgetragen |
| GrowWarn | blueprints/automation/growwarn.yaml | automation | valide | author + min_version 2024.6.0 nachgetragen |

**Status-Legende:**
- in Entwicklung
- wird geprueft
- valide
- veroeffentlicht

---

## Aktueller Stand — 2026-03-16

Alle 5 Blueprints wurden in die korrekte Verzeichnisstruktur verschoben
(blueprints/automation/, snake_case-Dateinamen ohne Leerzeichen).

Validator-Ergebnisse:
- YAML-Syntax: alle 5 OK
- Schema-Pruefung: alle 5 PASS (nach Korrekturen durch Developer)

Developer-Korrekturen durchgefuehrt:
- cam_active.yaml:      author + homeassistant.min_version ergaenzt
- presence_light.yaml:  author + homeassistant.min_version + vollstaendige Beschreibung ergaenzt
- tuer_alarm_pro.yaml:  author ergaenzt
- log_viewer.yaml:      author + homeassistant.min_version ergaenzt
- growwarn.yaml:        author + homeassistant.min_version ergaenzt

---

## Stabilitaets-Checkliste (Stand: 2026-03-16)

- [x] Alle Blueprints: YAML-Syntax korrekt
- [x] Alle Blueprints: name, domain, description vorhanden
- [x] Alle Blueprints: homeassistant.min_version gesetzt
- [x] Alle Blueprints: author: "magicx78" gesetzt
- [x] Alle Blueprints: Inputs haben name + selector
- [x] README.md: Import-URLs vorhanden  (Platzhalter-URLs — werden korrekt nach GitHub Push)
- [x] README.md: Kurzbeschreibung jedes Blueprints
- [x] CI Workflow: validate.yml vorhanden
- [x] Kein Blueprint mit TODO oder Platzhalter

---

## Offene Aufgaben

- [ ] GitHub Repo erstellen (manuell auf github.com — Name: ha-blueprints, Public)
- [ ] git remote add origin + git push -u origin main
- [ ] git tag v1.0.0 + git push origin v1.0.0
- [ ] GitHub Release v1.0.0 auf github.com anlegen

---

## Letzte Session — 2026-03-16

Blueprints stabilisiert und in korrekte Struktur gebracht.
Alle Pflichtfelder nachgetragen. Bereit fuer README + GitHub-Publish.

Publisher-Schritt abgeschlossen (2026-03-16):
- git init + .gitignore erstellt
- README.md mit Beschreibungen und Import-Buttons fuer alle 5 Blueprints
- .github/workflows/validate.yml (yamllint + Python Schema-Check)
- PROGRESS.md aktualisiert
- Initialer Commit: release v1.0.0

---

## Entscheidungen & Kontext

- Alle Blueprints liegen unter blueprints/automation/ mit snake_case-Namen
- min_version: Für tuer_alarm_pro 2024.10.0 (sections-Feature), alle anderen 2024.6.0
- presence_light.yaml: Urspruengliche Beschreibung war nur "VERSION 2" —
  wurde durch aussagekraeftige Beschreibung ersetzt (kein Inhalt geaendert)

---

## Bekannte Probleme / Hinweise

- log_viewer.yaml: Setzt einen command_line-Sensor (sensor.automation_log_reader)
  und einen shell_command in configuration.yaml voraus. In README kommunizieren.
- growwarn.yaml: Erfordert manuell angelegte input_text-Helpers vor dem ersten
  Einsatz (Dokumentation liegt als Kommentar in der Blueprint-Datei).

---

## Release-Status

- [x] v1.0.0 — lokales Repo bereit, initialer Commit erstellt
- [ ] v1.0.0 — GitHub Push + Release ausstehend (manueller Schritt)
