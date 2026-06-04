# ha-blueprints

Home Assistant Blueprints von [@magicx78](https://github.com/magicx78)

Alle Blueprints sind validiert (YAML-Syntax + HA-Schema) und erfordern mindestens Home Assistant 2024.6.0 (Ausnahme: Tür Alarm Pro benötigt 2024.10.0).

---

## Blueprints

### Camera Health (Frigate FPS + Unavailable + Pulse Ping)

Meldet Kamera- und Stream-Ausfälle über drei unabhängige Signale: die Kamera-Entity wird `unavailable`, der Frigate FPS-Wert fällt unter einen Schwellwert, oder ein Ping/Connectivity-Sensor geht auf `off`/`unavailable`. Optional werden Recovery-Meldungen gesendet. Eingebaut: Deduplizierung gegen Doppelmeldungen.

**Features:**
- Drei unabhängige Ausfallsignale (Unavailable, FPS, Ping)
- Optionale Recovery-Benachrichtigung
- Deduplizierung gegen Doppelmeldungen
- Kompatibel mit Frigate-Integrationen

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/magicx78/ha-blueprints/main/blueprints/automation/cam_active.yaml)

---

### Presence & Light v2 – Motion/Presence + Türkontakt + Timer + Dämmerung + Schalter

Anwesenheits- und bewegungsbasierte Lichtsteuerung. Unterstützt Bewegungsmelder, Präsenz-Sensoren, Türkontakte, Timer-Helfer, Dämmerungsprüfung (Lux und/oder Sonnenstand) sowie einen Sperr-Schalter. Das Licht schaltet sich bei Aktivität ein und nach Ablauf des Timers automatisch aus. Hinweis: Fehlt der Lux-Sensor oder liefert er unknown/unavailable, gilt dies als „dunkel“, damit die Automation weiter funktioniert.

**Features:**
- Bewegungsmelder und Präsenz-Sensoren kombinierbar
- Türkontakt als zusätzlicher Trigger
- Dämmerungsprüfung via Lux-Sensor oder Sonnenstand
- Sperr-Schalter zum manuellen Deaktivieren
- Helligkeit und Übergangszeiten konfigurierbar

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/magicx78/ha-blueprints/main/blueprints/automation/presence_light.yaml)

---

### Tür offen – Alarm Pro v4

Vollständiger Tür-Alarm mit TTS und Push-Benachrichtigung. TTS und Push sind getrennt steuerbar mit eigenen Schlafzeiten. Unterstützt Batterie-Warnungen für sowohl numerische Sensoren (`sensor.*` in %) als auch binäre Sensoren (`binary_sensor.*` LOWBAT), jeweils mit mehreren Entities. Wiederholungsansagen und Custom Actions möglich.

**Erfordert: Home Assistant 2024.10.0** (sections-Feature im Blueprint-Editor)

**Features:**
- TTS und Push-Benachrichtigung getrennt konfigurierbar
- Eigene Schlafzeiten für TTS und Push
- Batterie-Warnung: numerische und binäre Sensoren, mehrere Entities
- Wiederholungsansagen mit konfigurierbarem Intervall
- Custom Actions (vor/nach Alarm)
- Dankeschön-Ansage nach echtem Alarm-Trigger

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/magicx78/ha-blueprints/main/blueprints/automation/tuer_alarm_pro.yaml)

---

### Automation Log Viewer

Liest den Home Assistant Log gefiltert auf eine ausgewählte Automation aus und gibt das Ergebnis als Persistent Notification und/oder in einen `input_text`-Helper aus. Nützlich zur schnellen Diagnose von Automations-Problemen direkt aus der HA-Oberfläche.

> **Voraussetzung:** Dieser Blueprint setzt eine manuelle Einrichtung in `configuration.yaml` voraus:
> - Einen `command_line`-Sensor (z.B. `sensor.automation_log_reader`), der den Log ausliest
> - Einen `shell_command`-Eintrag, der den gefilterten Log-Abruf ausführt
>
> Ohne diese Konfiguration in `configuration.yaml` funktioniert der Blueprint nicht. Details zur Einrichtung liegen als Kommentar in der Blueprint-Datei.

**Features:**
- Filtert HA-Log auf gewählte Automation
- Ausgabe als Persistent Notification
- Optionale Ausgabe in `input_text`-Helper
- Auswahl der Automation über `input_select`-Dropdown

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/magicx78/ha-blueprints/main/blueprints/automation/log_viewer.yaml)

---

### GrowWarn

Überwacht Temperatur und Luftfeuchtigkeit in bis zu zwei Gewächshäusern (GH1 und GH2) und sendet Warnungen bei Über- oder Unterschreitung konfigurierbarer Schwellwerte. Unterstützt TTS, Push-Benachrichtigungen und Stale-Sensor-Erkennung (kein frisches Messdatum).

> **Voraussetzung:** Vor der ersten Ausführung müssen `input_text`-Helfer manuell angelegt werden.
>
> Pflicht (GH1):
> - `input_text.growwarn_gh1_temp`
> - `input_text.growwarn_gh1_hum`
> - `input_text.growwarn_gh1_temp_stale`
> - `input_text.growwarn_gh1_hum_stale`
>
> Optional (GH2, nur wenn GH2 konfiguriert):
> - `input_text.growwarn_gh2_temp`
> - `input_text.growwarn_gh2_hum`
> - `input_text.growwarn_gh2_temp_stale`
> - `input_text.growwarn_gh2_hum_stale`
>
> Helfer anlegen unter: Einstellungen → Geräte & Dienste → Helfer → + Helfer → Text (max. 255 Zeichen, Standardwert: `idle`)

**Features:**
- Bis zu zwei Gewächshaus-Zonen (GH1 + GH2)
- Temperatur- und Feuchtigkeitsüberwachung mit individuellen Schwellwerten
- Stale-Sensor-Erkennung (veraltete Messwerte)
- TTS und Push-Benachrichtigung (Piper / Google / Nabu Casa, HA Companion App)

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/magicx78/ha-blueprints/main/blueprints/automation/growwarn.yaml)

---

### Bluepoint mmWave Licht mit Lux, Anwesenheit, Timer und Bypass

Lichtsteuerung mit mmWave-Präsenzsensor. Das Licht schaltet nur ein, wenn jemand anwesend ist (Person/Gruppe/Helper auf `home`/`on`/`present`/`detected`) und es dunkel genug ist (Lux ≤ Schwellwert). Nach Wegfall der Präsenz wird das Licht nach einer konfigurierbaren Verzögerung ausgeschaltet. Ein Sofort-An-Helfer schaltet das Licht unabhängig von Lux **und** Anwesenheit ein und nimmt es vom automatischen Ausschalten aus. Ein Bypass-Helfer deaktiviert die Automation komplett – bei aktivem Bypass wird weder ein- noch ausgeschaltet.

**Erfordert: Home Assistant 2024.10.0** (neue `triggers:`/`actions:`-Syntax)

**Features:**
- mmWave-Präsenzsensor als Trigger (Ein/Aus)
- **Optionale Bewegungsmelder** (motion/occupancy, Mehrfachauswahl) als zusätzliche Aktivitätsquelle
- **Optionale Türkontakte** (door/opening/window) mit Türmodus: ignorieren / Öffnung schaltet ein / offen hält Licht an
- Anwesenheitsprüfung (Person / Gruppe / Helper)
- Dämmerungsprüfung über Lux-Schwellwert – **Wert `0` deaktiviert die Luxprüfung**
- Konfigurierbare Ausschaltverzögerung nach Wegfall der Präsenz
- Sofort-An ohne Timer (Helfer): sofort ein (ignoriert Lux + Anwesenheit), kein Auto-Aus
- Bypass-Helfer deaktiviert die Automation komplett (Ein und Aus)
- Steuert Lichter und/oder Schalter

> **Optionale Eingaben:** Bewegungsmelder, Türkontakte, Bypass-Helfer, Sofort-An-Helfer und Luxsensor sind optional und können leer bleiben — die Automation läuft dann genauso wie ohne sie. Es gilt immer nur explizit `on` als aktiv; fehlende/leere/`unknown`/`unavailable` Entities blockieren nichts.
>
> **Türmodus:**
> - `none`: Türkontakte werden ignoriert.
> - `trigger_on_open`: Türöffnung schaltet ein, verhindert aber kein Ausschalten.
> - `hold_while_open`: solange mindestens ein Türkontakt offen ist, wird nicht ausgeschaltet.
>
> **Wichtig:**
> - Räume ohne brauchbaren Luxsensor: Luxsensor leer lassen **und** Dämmerungswert auf `0` setzen.
> - Bypass und Sofort-An **nicht** denselben Helfer verwenden (die Bedeutungen widersprechen sich).
> - Keinen vorhandenen Raum-Deaktivieren-Helper als Dummy missbrauchen — sonst werden andere Räume versehentlich beeinflusst.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/magicx78/ha-blueprints/main/blueprints/automation/mmwave_light.yaml)

---

## Installation

1. Auf den "Import Blueprint" Button des gewünschten Blueprints klicken.
2. Home Assistant öffnet sich automatisch mit dem Import-Dialog.
3. Blueprint bestätigen und anschließend eine neue Automation daraus erstellen.

Bei Blueprints mit Voraussetzungen (Log Viewer, GrowWarn) zuerst die beschriebenen Helfer und `configuration.yaml`-Einträge anlegen.

---

## Anforderungen

| Blueprint | Min. HA-Version |
|-----------|----------------|
| Camera Health | 2024.6.0 |
| Presence & Light v2 | 2024.6.0 |
| Tür Alarm Pro v4 | 2024.10.0 |
| Automation Log Viewer | 2024.6.0 |
| GrowWarn | 2024.6.0 |
| Bluepoint mmWave Licht | 2024.10.0 |

---

## Versionierung & Releases

Releases werden automatisch über GitHub Actions erstellt (Workflow `.github/workflows/release.yml`).

**Neuen Release erzeugen:**
1. Die Datei [`VERSION`](VERSION) auf die neue Versionsnummer setzen (z.B. `1.2.0`).
2. Änderung nach `main` bringen (Commit/PR-Merge).
3. Der Workflow legt automatisch Tag **und** GitHub-Release `vX.Y.Z` mit generierten Notes an.

Alternativ lässt sich der Workflow manuell über **Actions → Release → Run workflow** auslösen.

Aktuelle Version: siehe [`VERSION`](VERSION). Der mmWave-Blueprint trägt seine Version zusätzlich in der Beschreibung und besitzt eine `source_url` für den Re-Import/Update in Home Assistant.

---

## Lizenz

MIT

