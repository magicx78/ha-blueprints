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
| GrowWarn | blueprints/automation/growwarn.yaml | automation | valide | v1.4: binary_sensor enabled-Guard Fix (OOM-Kill), min_version → 2024.1.0 |
| Blueprint mmWave Licht (Lux/Anwesenheit/Timer/Bypass) | blueprints/automation/mmwave_light.yaml | automation | valide | v1.2.0: Lux-Robustheit (leerer/unknown/unavailable Luxsensor = Prüfung aus); Bypass-Neubewertung beim Ausschalten; activity_active-DRY; Name-Typo "Bluepoint" behoben. Weiterhin: optionale Bewegungsmelder + Türkontakte (door_mode none/trigger_on_open/hold_while_open); Bypass/Sofort-An/Luxsensor optional |

**Status-Legende:**
- in Entwicklung
- wird geprueft
- valide
- veroeffentlicht

---

## Aktueller Stand — 2026-07-03 (v1.2.0: mmWave Review + Lux-Robustheit)

Vollstaendiges Review des mmWave-Blueprints nach dem Blueprint-Review-Template.
Ergebnis: wahrscheinlich funktionsfaehig / produktiv einsetzbar, keine harten
YAML-/Syntaxfehler. Zwei Punkte umgesetzt, Version 1.1.0 -> 1.2.0:

- Lux-Robustheit (Verhaltensaenderung, analog presence_light):
  Neue Variable lux_ok. Ein leerer ODER unknown/unavailable Luxsensor
  deaktiviert die Luxpruefung jetzt automatisch, statt das Einschalten zu
  blockieren (bisher float(9999)-Fallback = "zu hell" -> Licht blieb nachts aus,
  wenn der Sensor ausfiel oder die Schwelle nicht auf 0 stand). Schwelle 0
  deaktiviert die Pruefung weiterhin ebenfalls. Lux-Checks im Einschalt- und im
  Bypass-Aus-Zweig nutzen nun lux_ok.
- Aufraeumung (keine Verhaltensaenderung): die bereits definierte, aber
  ungenutzte Variable activity_active ersetzt den dreifach wiederholten Ausdruck
  "mmwave on OR motion OR (hold_while_open AND door offen)" (Bypass-Aus-Zweig,
  Ausschalt-Vorpruefung, Sofort-An-Aus-Zweig). Der LIVE-Recheck NACH dem delay
  (bypass_now/instant_now/... ) bleibt bewusst inline/live und wird NICHT durch
  den Trigger-Zeit-Snapshot activity_active ersetzt.
- Aus dem 1.1.1-Entwurf uebernommen: Bypass-Neubewertung beim Ausschalten
  (bypass_off laeuft trotz Top-Level-Bypass-Blockade durch und schaltet je nach
  Aktivitaet ein/aus) sowie Name-Typo-Fix "Bluepoint" -> "Blueprint".
- Doku (Blueprint-description, Input-Texte, README) an das neue Lux-Verhalten
  angepasst; VERSION auf 1.2.0 gebumpt (loest Release v1.2.0 aus).

---

## Aktueller Stand — 2026-06-04 (Nachtrag 4: Release-Automatisierung)

Problem: Tags/Releases lassen sich nicht aus der Arbeitsumgebung pushen (Git 403,
kein Release-API-Tool). Dauerhafte Loesung:
- Neue Datei VERSION (aktuell 1.1.0).
- Neuer Workflow .github/workflows/release.yml (permissions: contents: write):
  loest bei Aenderung von VERSION auf main aus (und per workflow_dispatch),
  erstellt via GITHUB_TOKEN + gh automatisch Tag + Release vX.Y.Z (--generate-notes),
  idempotent (kein Doppel-Release).
- Damit entsteht beim Merge automatisch Release v1.1.0; kuenftig genuegt ein
  Bump der VERSION-Datei.
- README um Abschnitt "Versionierung & Releases" ergaenzt.

---

## Aktueller Stand — 2026-06-04 (Nachtrag 3: Bewegungsmelder + Türkontakte)

mmWave-Blueprint um optionale Aktivitaetsquellen erweitert (bestehende Logik unveraendert):
- Neue Inputs: motion_sensors (multiple, device_class motion/occupancy, default []),
  door_sensors (multiple, device_class door/opening/window, default []),
  door_mode (select: none / trigger_on_open / hold_while_open, default none).
- mmWave-Trigger-IDs zu mmwave_on/mmwave_off umbenannt; motion_on/off jetzt fuer
  echte Bewegungsmelder; neue Trigger door_open/door_closed.
- Einschalten: zusaetzlich ueber Bewegungsmelder oder (je nach Tuermodus) Tueroeffnung.
- Ausschalten: blockiert solange mmWave on ODER ein Bewegungsmelder on ODER (im Modus
  hold_while_open) eine Tuer offen ODER Sofort-An aktiv. Nach Delay LIVE-Neupruefung
  via variables-Action (inkl. Bypass), damit kein veralteter Snapshot ausschaltet.
- Zentrale Template-Variablen: bypass_active, instant_on_active, motion_active, door_open
  (nur explizit 'on' = aktiv; leere Liste/unknown/unavailable => nicht aktiv).
- Leere Entity-Listen in Triggern bleiben gueltig (default []), keine Dummy-Entities.
- Verifikation: CI-Schema + yamllint OK; 17 Logik-Szenarien per Jinja2-Sim bestanden
  (alle Akzeptanzkriterien).

---

## Aktueller Stand — 2026-06-04 (Nachtrag 2: optionale Helfer + Lux 0)

mmWave-Blueprint robuster gemacht (Pflicht-Helfer fuehrten zu kaputten Logiken,
wenn Dummy-/geteilte Helfer eingetragen wurden):
- bypass_helper jetzt optional (default ""), Bedingung von is_state(...,'off')
  auf "not is_state(...,'on')" geaendert -> fehlender/leerer/unavailable Helfer
  blockiert nicht; nur echter Helfer mit 'on' blockiert.
- instant_on_helper jetzt optional als Mehrfach-Auswahl (default []), da ein leerer
  Einzel-Entity-Wert in State-Triggern ungueltig ist (Muster wie presence_light).
  Auswertung live via expand(instant_on_helper)|selectattr('state','eq','on').
  WICHTIG: Live-Template statt Variable, damit der Re-Check NACH dem delay korrekt ist.
- lux_sensor optional (default ""); Lux-Bedingung um "lux_threshold|float <= 0"
  erweitert -> Schwellwert 0 deaktiviert die Luxpruefung (Raeume ohne Luxsensor:
  leer lassen + Schwelle 0).
- Doku in Blueprint-description, README und PROGRESS ergaenzt (Lux 0 = aus;
  Bypass/Instant-On nicht denselben Helfer; keine Raum-Deaktivieren-Helper als Dummy).
- YAML/Schema/yamllint bestanden.

---

## Aktueller Stand — 2026-06-04 (Nachtrag: Logik-Fix)

Code-Review des neuen mmWave-Blueprints ergab zwei Abweichungen zur Beschreibung,
beide gefixt:
- Bypass wirkte bisher nur beim Einschalten -> als globale Top-Level-Bedingung
  (conditions:) umgesetzt; bei aktivem Bypass laeuft KEIN Zweig mehr (Ein + Aus).
  Redundanter Bypass-Check im Einschalt-Zweig entfernt.
- Sofort-An (instant_on) erforderte bisher Anwesenheit -> Praesenz-Bedingung
  per "trigger.id == 'instant_on' or ..." uebersprungen; Sofort-An ignoriert nun
  Lux UND Anwesenheit (nur Bypass kann es noch verhindern).
- README + PROGRESS aktualisiert; YAML/Schema-Check bestanden.

---

## Aktueller Stand — 2026-06-04

Neues Blueprint "Bluepoint mmWave Licht mit Lux, Anwesenheit, Timer und Bypass"
hinzugefuegt (blueprints/automation/mmwave_light.yaml):
- mmWave-Trigger (on/off) + Sofort-An-Helfer-Trigger (on/off)
- Einschalten nur bei Anwesenheit (home/on/present/detected), Bypass aus,
  und Lux <= Schwellwert (bei Sofort-An wird Lux ignoriert)
- Ausschalten nach off_delay, wenn weiterhin keine Praesenz und Sofort-An aus
- Sofort-An aus -> Licht aus, sofern mmWave aus
- Pflichtfelder ergaenzt: author "magicx78", homeassistant.min_version 2024.10.0
  (neue triggers:/actions:-Syntax), Beschreibungen je Input
- README + PROGRESS aktualisiert; YAML- und Schema-Check bestanden

---

## Aktueller Stand — 2026-04-03

Presence & Light v2:
- Lux-/Dunkelheitspruefung robuster gemacht: fehlt der Lux-Sensor oder liefert er
  unknown/unavailable, wird dies als "dunkel" behandelt, damit die Automation
  weiter funktioniert.
- Beschreibung im Blueprint entsprechend ergaenzt.

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
- growwarn.yaml (v1.4): binary_sensor Trigger-Storm Fix — enabled-Guard ergaenzt

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

## Letzte Session — 2026-04-03

Presence & Light v2:
- Lux-Fallback (unknown/unavailable/fehlender Sensor => dunkel) umgesetzt
- Beschreibung im Blueprint ergaenzt

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
- presence_light.yaml: Lux-Sensor optional; unknown/unavailable oder fehlender Sensor
  gilt als "dunkel", damit die Automation nicht blockiert

---

## Bekannte Probleme / Hinweise

- log_viewer.yaml: Setzt einen command_line-Sensor (sensor.automation_log_reader)
  und einen shell_command in configuration.yaml voraus. In README kommunizieren.
- growwarn.yaml: Erfordert manuell angelegte input_text-Helpers vor dem ersten
  Einsatz (Dokumentation liegt als Kommentar in der Blueprint-Datei).

---

## Release-Status

- [x] v1.0.0 — lokales Repo bereit, initialer Commit erstellt
- [x] v1.0.0 — GitHub Push erfolgt
- [ ] v1.1.0 — growwarn v1.4 Fix — GitHub Push + Release ausstehend
- [ ] v1.1.0 — presence_light: Lux-Fallback (unknown/unavailable/fehlend => dunkel) + README/PROGRESS Hinweis
