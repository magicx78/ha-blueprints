# PROGRESS.md — ha-blueprints

## Ziel
Sammlung qualitativ hochwertiger Home Assistant Blueprints entwickeln,
validieren und auf GitHub veröffentlichen.

---

## Blueprints — Übersicht

| Blueprint | Datei | Domain | Status | Notizen |
|-----------|-------|--------|--------|---------|
| Camera Health (Frigate FPS + Unavailable + Pulse Ping) | blueprints/automation/cam_active.yaml | automation | valide | author + min_version 2024.6.0 nachgetragen |
| Presence & Light v2 | blueprints/automation/presence_light.yaml | automation | valide | v2.1.0: Modus-"Deaktiviert"-Fix (Bewegung schaltete ein, hielt aber nicht), Türkontakt-Hold (Opt-in, Modi open/closed, mit Timer-Re-Arm), Tür-Trigger nur noch echte on/off-Übergänge (from/to), Timer-Refresh auch bei Licht an (Dunkelheit gated nur noch das Einschalten), Ausschalten nur bei Licht an, source_url + Versionsheader. Zuvor: author + min_version 2024.6.0 + Beschreibung nachgetragen |
| Tuer offen Alarm Pro v4 | blueprints/automation/tuer_alarm_pro.yaml | automation | valide | author nachgetragen; min_version 2024.10.0 war bereits vorhanden |
| Automation Log Viewer | blueprints/automation/log_viewer.yaml | automation | valide | author + min_version 2024.6.0 nachgetragen |
| GrowWarn | blueprints/automation/growwarn.yaml | automation | valide | v1.4: binary_sensor enabled-Guard Fix (OOM-Kill), min_version → 2024.1.0 |
| Blueprint mmWave Licht (Lux/Anwesenheit/Timer/Bypass) | blueprints/automation/mmwave_light.yaml | automation | valide | v1.6.0: Dämmerungs-Binärsensor(en) als neue optionale Dunkelheitsquelle (Mehrfachauswahl, `on` = dunkel, ODER-verknüpft mit Lux; darkness_on-Trigger analog lux_below; Hellwerden schaltet nie aktiv aus). v1.5.1: Trigger-Härtung (unavailable→off startet keinen Off-Timer mehr; Garage not_from unknown/unavailable) + Multi-Helper-Fix (bypass_off/instant_off prüfen verbleibende Helfer). v1.5.0: Garagentore (cover + binary_sensor) mit eigenem Garagenmodus. v1.4.0: mmWave-Sensor optional (Mehrfachauswahl, default []); mind. eine Aktivitätsquelle nötig. v1.3.0: Lux-Trigger (Einschalten bei Dämmerung trotz stehender mmWave-Präsenz). v1.2.0: Lux-Robustheit (leerer/unknown/unavailable Luxsensor = Prüfung aus); Bypass-Neubewertung; activity_active-DRY; Name-Typo behoben. Weiterhin: optionale Bewegungsmelder + Türkontakte; Bypass/Sofort-An/Luxsensor optional |

| Entity Watchdog (Ausfall-Benachrichtigung) | blueprints/automation/entity_watchdog.yaml | automation | valide | v1.0.0: Überwacht beliebige Entities auf unavailable/unknown; einstellbare Ausfall-Verzögerung (Default 5 min, 0 = sofort); Push an mehrere Companion-App-Geräte + optionale persistente HA-Benachrichtigung; Entwarnung nur nach echter Meldung (Dauer ≥ Verzögerung), ersetzt Push per tag und dismisst die persistente Meldung; continue_on_error je Zustellung; mode: queued. Begleiter zu mmwave_light/presence_light |

**Status-Legende:**
- in Entwicklung
- wird geprueft
- valide
- veroeffentlicht

---

## Aktueller Stand — 2026-07-18 (mmwave_light v1.6.0: Daemmerungs-Binaersensor, Release 1.8.0)

Nutzerwunsch: zusaetzlich zum Luxsensor ein Daemmerungs-Binaersensor
(Ja/Nein bei Dunkelheit) als weitere Dunkelheitsquelle.

- Neuer optionaler Eingang darkness_sensors (Mehrfachauswahl; binary_sensor /
  input_boolean / schedule; 'on' = dunkel; default [] -> Re-Import bricht
  keine bestehende Automation).
- Nutzer-Entscheidungen: feste ODER-Verknuepfung mit Lux (kein zusaetzlicher
  Modus-Input); Hellwerden (an -> aus) schaltet NICHT aktiv aus (konsistent
  zum Lux-Verhalten, Licht geht nur ueber den Praesenz-Timer aus).
- lux_ok -> darkness_ok mit Stimmen-Logik: nicht konfigurierte bzw.
  unknown/unavailable Quellen zaehlen nicht; ist KEINE gueltige Quelle
  konfiguriert, ist die Pruefung deaktiviert (Robustheit aus 1.2.0 erhalten;
  Schwelle 0 = Lux aus wie bisher).
- Neuer Template-Trigger darkness_on (analog lux_below) mit mmWave-Guard:
  feuert nur bei stehender mmWave-Praesenz, damit mode:restart keinen
  laufenden Ausschalt-Timer abbricht.
- README-Abschnitt aktualisiert (Features, optionale Eingaben);
  VERSION 1.7.0 -> 1.8.0.

---

## Aktueller Stand — 2026-07-18 (entity_watchdog v1.0.0: neues Blueprint, Release 1.7.0)

Neues Blueprint entity_watchdog.yaml (Folgeauftrag nach mmwave_light v1.5.1 —
die Diagnose zeigte regelmaessige Sensor-Ausfaelle als Hauptursache):

- Ueberwacht beliebige Entities auf unavailable/unknown; Trigger `failed` mit
  `for: !input fail_delay` (Default 5 min, 0 = sofort) — kurze Reconnects
  unterhalb der Verzoegerung melden nicht.
- Zustellung an mehrere Companion-App-Geraete (device-Selector, notify-Dienst
  aus Geraetename via device_attr+slugify) + optionale persistente
  HA-Benachrichtigung (notification_id pro Entity).
- Entwarnung (`recovered`) nur wenn Ausfalldauer >= fail_delay (sonst Spam bei
  kurzen Dropouts); ersetzt Push per tag, dismisst persistente Meldung.
- Fehlertoleranz: continue_on_error je Zustellaktion; mode: queued (max 20)
  fuer gleichzeitige Ausfaelle. BEWUSST separates Blueprint statt Integration
  in mmwave_light: dessen mode:restart wuerde durch Watchdog-Trigger laufende
  Ausschalt-Timer abbrechen; ausserdem kann ein Watchdog-Fehler so nie die
  Lichtsteuerung blockieren.
- README-Abschnitt + Versions-Tabelle; VERSION 1.6.1 -> 1.7.0.
- In der Produktiv-HA: Blueprint importiert + EINE zentrale Watchdog-Automation
  fuer die Sensoren/Kontakte/Lichter der 5 Licht-Automationen angelegt
  (Ziel: iphonemw + iphonecw + persistente Benachrichtigung).

---

## Aktueller Stand — 2026-07-18 (mmwave_light v1.5.1: Trigger-Haertung, Release 1.6.1)

Symptom "Licht geht manchmal einfach aus" fuer die mmWave-Automationen per
Live-Diagnose in der Produktiv-HA untersucht (Traces + Sensor-Historie der
5 Blueprint-Instanzen: Eingang, Kueche, Flur BLE, Lager, Hof):

- Hauptursache liegt auf Sensorebene: die HLK-LD2450-Sensoren verlieren
  still sitzende Personen (Kueche: Person 8 min "unsichtbar", Licht ging
  nach 3-min-Timer korrekt aus) und fallen regelmaessig auf unavailable
  (Kueche ~7x/24h, einmal 74 min offline). Die Blueprint-Logik arbeitete in
  den geprueften Traces korrekt.
- Blueprint-Bug A (gefixt): mmwave_off/motion_off-Trigger ohne `from: "on"`
  feuerten auch bei unavailable->off (Geraete-Reconnect, HA-Neustart) und
  starteten spurious Off-Timer. Fix: `from: "on"` ergaenzt; Garage-Trigger
  analog mit `not_from: [unknown, unavailable]` gehaertet.
- Blueprint-Bug B (gefixt): bei MEHREREN Bypass- bzw. Sofort-An-Helfern
  reagierte die Automation schon beim Ausschalten EINES Helfers, obwohl ein
  anderer Helfer derselben Gruppe noch 'on' war. Fix: Zusatzbedingungen
  `not bypass_active` (bypass_off-Zweig) und `not instant_on_active`
  (instant_off-Zweig).
- Nutzer-Entscheidung: unavailable zaehlt im Live-Recheck weiterhin als
  "keine Praesenz" (kein Fail-Safe-Anlassen) — nur Trigger gehaertet.
- VERSION 1.6.0 -> 1.6.1 (Repo-Release; Blueprint-interne Version 1.5.1).
- Offene Empfehlungen (ausserhalb Blueprint): LD2450-Tuning + WLAN-
  Stabilitaet der ESPHome-Geraete pruefen; off_delay in betroffenen
  Automationen erhoehen (z. B. Kueche 3 -> 10 min); klaeren, ob
  automation.hof_licht absichtlich auf die Lager-Sensoren
  (ble_lager_sensor_*) triggert.

---

## Aktueller Stand — 2026-07-18 (presence_light v2.1.0: Bugfixes + Tuerkontakt-Hold)

Ursache fuer "Licht geht manchmal einfach aus" analysiert und behoben
(statische Logik-Analyse + Zweit-Review; Blueprint ist in der Test-Instanz
nicht instanziiert, daher keine Traces):

- B1: Bewegungsmelder-Modus "Deaktiviert" war widerspruechlich — Motion
  schaltete das Licht ein (active_trigger), hielt es aber nicht
  (v_is_active ignorierte Motion) -> Licht ging nach Timer-Ablauf trotz
  Bewegung aus. Fix: neues v_activation_event ignoriert Motion-Events bei
  Modus 'off' vollstaendig (weder Einschalten noch Timer).
- B2: v_contact_active war toter Code — Tueren hielten nie. Fix: Opt-in
  Input `door_contact_hold` (boolean, default false = Alt-Verhalten).
  Aktiv: Modus 'open' haelt bei offener, Modus 'closed' bei geschlossener
  Tuer (Bad-Szenario); 'both'/'off' halten NIE (sonst unsterbliches
  Licht). Endet der Hold, re-armt FALL 2 den Ausschalt-Timer
  (v_deactivation_event, tuer-seitig nur bei aktivem Hold -> Alt-Timing
  ohne Hold exakt erhalten).
- B3: Tuer-Trigger ohne from/to feuerte auch bei unavailable/unknown und
  reinen Attribut-Updates (Modus "Beides": jede Aenderung = Aktivierung).
  Fix: zwei Trigger door_open (off->on) und door_closed (on->off);
  v_contact_event_ok arbeitet nur noch ueber trigger-id + Modus (kein
  to_state-Zugriff mehr).
- B4 (Hauptursache fuer "manchmal"): Das Dunkelheits-Gate in FALL 1
  blockierte auch den Timer-Refresh (Lux-Feedback: Licht macht Raum hell).
  Der Timer lief dann unbemerkt weiter; traf der Ablauf einen kurzen
  mmWave-/PIR-Aussetzer, ging das Licht trotz Anwesenheit aus. Fix:
  FALL-1-Gate ist jetzt `v_any_light_on or v_is_dark` — Refresh laeuft
  bei brennendem Licht immer, Einschalten bleibt dunkel-gated.
- B5: FALL 3 schaltet nur noch aus, wenn mind. ein Ziel-Licht an ist
  (fremde/verwaiste timer.finished = No-Op). Doku-Hinweise ergaenzt:
  Timer-Helfer pro Automation dediziert + restore aktivieren;
  unknown/unavailable zaehlt als inaktiv; manuelle Lichter werden
  uebernommen -> Sperr-Schalter nutzen.
- B6: source_url ergaenzt (Re-Import-Updates jetzt moeglich),
  Versionsheader + Changelog in der description.
- Intern: Trigger-IDs umbenannt (motion_on/motion_off, presence_on/
  presence_off, door_open/door_closed, timer_finished); v_trigger_id mit
  trigger-is-defined-Guard macht manuelle Ausfuehrung zum sauberen No-Op.
  Keine Input-Keys geaendert — Bestandsautomationen laufen nach Re-Import
  unveraendert (door_contact_hold default false).
- README (Features + Wichtig-Block) und VERSION 1.5.0 -> 1.6.0.
- Verifikation lokal: yamllint relaxed, CI-Schema-Check, Jinja2-Simulation
  der echten Templates (9216 Faelle: Modus x Kontaktmodus x Hold x Event x
  Licht x Dunkel x Sensorzustaende inkl. unavailable; Invarianten I1-I10)
  + 4 End-to-End-Regressionsstories (B1, B2 open, B2 closed/Bad, B4) +
  statische Trigger-Checks (from/to, IDs, genau ein timer.start je Fall).

---

## Aktueller Stand — 2026-07-03 (v1.5.0: Garagentore)

Garagentor-Unterstuetzung ergaenzt (Garagentore lassen sich nicht als
Tuerkontakt auswaehlen: cover-Domain bzw. device_class garage passt nicht in den
door/opening/window-Filter):

- Neuer Eingang `garage_doors` (multiple, default []), Selector domain
  [binary_sensor, cover] OHNE device_class-Filter, damit sowohl cover- als auch
  binary_sensor-Garagentore erscheinen.
- Neuer Eingang `garage_mode` (none / trigger_on_open / hold_while_open,
  default hold_while_open) — unabhaengig vom Tuermodus.
- "Offen" fuer Garagentore = Zustand in ['on','open','opening'] (cover UND
  binary_sensor abgedeckt); via `selectattr('state','in',[...])`.
- Neue Trigger garage_open (to: on/open/opening) und garage_closed
  (to: off/closed) — ohne `from`, damit cover-Zwischenzustaende sauber greifen.
- Neue Variable `garage_open`; in activity_active, Einschalt-Aktivitaetsquelle,
  Off-Delay-Trigger-Liste und LIVE-Recheck (`garage_hold_now`) integriert —
  vollstaendig parallel zur bestehenden Tuer-Logik.
- description + README + Garagenmodus-Doku ergaenzt. VERSION 1.4.0 -> 1.5.0.
- Verifikation lokal: yamllint relaxed, Schema-Check, Jinja-Compile (inkl.
  'in'-Test in selectattr) + End-to-End-Simulation (cover open/opening/closed,
  binary_sensor on/off, trigger_on_open vs hold_while_open).

---

## Aktueller Stand — 2026-07-03 (v1.4.0: mmWave-Sensor optional)

mmWave-Praesenzsensor auf Wunsch optional gemacht:

- Input `mmwave_sensor`: von Pflicht-Einzel-Entity auf `multiple: true` +
  `default: []` umgestellt (Muster wie motion_sensors). Der Input-KEY bleibt
  `mmwave_sensor` -> bestehende Configs mit einem einzelnen Sensor laufen zur
  Laufzeit unveraendert weiter (String wird durchgereicht; State-Trigger mit
  Einzel-Entity gueltig; `expand()` akzeptiert String UND Liste). Falls das Feld
  im Editor nach dem Update leer erscheint: Sensor einmal neu auswaehlen.
- Warum Liste statt optionaler Einzel-Entity: ein optionaler Einzel-Entity waere
  bei leerem Feld `entity_id: ""` -> die State-Trigger mmwave_on/off wuerden die
  Automation beim Laden ungueltig machen. Leere Liste `[]` in einem State-Trigger
  ist dagegen gueltig (kein Trigger) — derselbe Grund wie bei motion_sensors.
- Neue Variable `mmwave_active`
  (`expand(mmwave_sensor)|selectattr('state','eq','on')|list|count>0`). Ersetzt
  alle bisherigen `is_state(mmwave_sensor,'on')` (activity_active, Einschalt-
  Aktivitaetsquelle, LIVE-Recheck mmwave_now_on, lux_below-Trigger via expand).
- Semantik: mind. eine Aktivitaetsquelle (mmWave, Bewegungsmelder oder Tuerkontakt
  im passenden Modus) muss konfiguriert sein, sonst geht das Licht nie an —
  in description + README dokumentiert.
- VERSION 1.3.0 -> 1.4.0. Verifikation lokal: yamllint relaxed (nur Line-Length-
  Warnings), Schema-Check, Jinja-Compile + Logik-Simulation (mmWave leer / einzeln
  / Liste; Off-Delay-Recheck).

---

## Aktueller Stand — 2026-07-03 (v1.3.0: Lux-Trigger)

Follow-up aus dem v1.2.0-Review umgesetzt: das Licht ging bislang bei sinkender
Helligkeit NICHT an, wenn der mmWave-Sensor bereits 'on' war (stehende Praesenz
ohne neues Bewegungsevent). Behoben mit einem neuen Lux-Trigger:

- Neuer Template-Trigger id=lux_below. Feuert beim Uebergang false->true, wenn:
  Luxsensor gesetzt + gueltiger Wert, Schwelle > 0, Lux <= Schwelle UND mmWave on.
- Bewusst KEIN numeric_state-Trigger: der Luxsensor ist optional (default ""),
  ein numeric_state-Trigger mit entity_id "" wuerde die Automation beim Laden
  ungueltig machen. Ein Template-Trigger behandelt leere/unknown/unavailable
  Sensoren sauber (Ausdruck bleibt einfach false).
- Neuer Top-Level-Block trigger_variables (tv_lux_sensor/tv_lux_threshold/
  tv_mmwave_sensor), da automations-`variables:` in Triggern NICHT verfuegbar
  sind, `trigger_variables:` aber schon.
- Die mmWave-on-Klausel im Trigger verhindert, dass lux_below waehrend eines
  laufenden Ausschalt-Delays (mmWave off) feuert und diesen per mode:restart
  abbricht.
- lux_below in die Trigger-Liste des Einschalt-Zweigs aufgenommen; die
  bestehenden Bedingungen (Anwesenheit, Aktivitaetsquelle=mmWave on, lux_ok)
  greifen unveraendert und gaten den neuen Trigger korrekt.
- VERSION 1.2.0 -> 1.3.0 (loest Release v1.3.0 aus); README/description ergaenzt.
- Verifikation lokal: yamllint relaxed (nur Line-Length-Warnings), Schema-Check,
  Jinja-Compile der Trigger-/Condition-Templates.

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
