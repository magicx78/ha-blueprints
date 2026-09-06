# ha-blueprints

Home Assistant Blueprints von [@magicx78](https://github.com/magicx78)

Alle Blueprints sind validiert (YAML-Syntax + HA-Schema) und erfordern mindestens Home Assistant 2024.6.0. Ausnahmen: GrowWarn benötigt 2024.8.0; mmWave Licht, Tür Alarm Pro, Entity Watchdog und Türöffnung BLE benötigen 2024.10.0. Zuletzt gegen die Release-Notes bis Home Assistant 2026.9 geprüft (2026-09-06).

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
> Ohne diese Konfiguration in `configuration.yaml` funktioniert der Blueprint nicht. Ein vollständiges Beispiel liegt als Kommentar am Anfang der Blueprint-Datei. Der Blueprint übergibt `automation_name` und `log_level` als Variablen an den `shell_command`.

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

### Blueprint mmWave Licht mit Lux, Anwesenheit, Timer und Bypass

Lichtsteuerung mit mmWave-Präsenzsensor. Das Licht schaltet nur ein, wenn jemand anwesend ist (Person/Gruppe/Helper auf `home`/`on`/`present`/`detected`) und es dunkel genug ist (Lux ≤ Schwellwert **oder** ein Dämmerungssensor meldet Dunkelheit). Nach Wegfall der Präsenz wird das Licht nach einer konfigurierbaren Verzögerung ausgeschaltet. Ein Sofort-An-Helfer schaltet das Licht unabhängig von Dunkelheit **und** Anwesenheit ein und nimmt es vom automatischen Ausschalten aus. Ein Bypass-Helfer deaktiviert die Automation komplett – bei aktivem Bypass wird weder ein- noch ausgeschaltet; beim Ausschalten des Bypass wird die Situation neu bewertet.

**Erfordert: Home Assistant 2024.10.0** (neue `triggers:`/`actions:`-Syntax)

> **Hinweis:** Alle Aktivitätsquellen sind optional, aber **mindestens eine** (mmWave-Sensor, Bewegungsmelder, Türkontakt oder Garagentor im passenden Modus) muss konfiguriert sein, sonst geht das Licht nie automatisch an.

**Features:**
- **Optionale mmWave-Präsenzsensoren** (Mehrfachauswahl) als Trigger (Ein/Aus)
- **Optionale Bewegungsmelder** (motion/occupancy, Mehrfachauswahl) als zusätzliche Aktivitätsquelle
- **Optionale Türkontakte** (door/opening/window) mit Türmodus: ignorieren / Öffnung schaltet ein / offen hält Licht an
- **Optionale Garagentore** (`cover` oder `binary_sensor`) mit eigenem Garagenmodus — für Garagentore, die sich nicht als Türkontakt auswählen lassen (Zustände `on`/`open`/`opening` gelten als offen)
- Anwesenheitsprüfung (Person / Gruppe / Helper)
- Dämmerungsprüfung über Lux-Schwellwert – **Wert `0` deaktiviert die Luxprüfung**; ein leerer / `unknown` / `unavailable` Luxsensor deaktiviert sie automatisch (Licht funktioniert weiter)
- **Optionale Dämmerungssensor(en)** (`binary_sensor` / `input_boolean` / Zeitplan, Mehrfachauswahl, `on` = dunkel) als zweite Dunkelheitsquelle — **ODER-verknüpft** mit dem Luxsensor: dunkel genug, wenn Lux ≤ Schwellwert **oder** mindestens ein Dämmerungssensor an ist; `unknown`/`unavailable` werden ignoriert, Hellwerden schaltet nie aktiv aus
- **Dunkel-Trigger:** sinkt die Helligkeit unter den Schwellwert oder meldet ein Dämmerungssensor Dunkelheit, während der mmWave-Sensor bereits Präsenz meldet, schaltet das Licht automatisch ein (auch ohne neues Bewegungsevent)
- Konfigurierbare Ausschaltverzögerung nach Wegfall der Präsenz
- Sofort-An ohne Timer (Helfer): sofort ein (ignoriert Lux + Anwesenheit), kein Auto-Aus
- Bypass-Helfer deaktiviert die Automation komplett (Ein und Aus)
- Steuert Lichter und/oder Schalter

> **Optionale Eingaben:** Bewegungsmelder, Türkontakte, Garagentore, Bypass-Helfer, Sofort-An-Helfer, Luxsensor und Dämmerungssensoren sind optional und können leer bleiben — die Automation läuft dann genauso wie ohne sie. Es gilt immer nur explizit `on` als aktiv; fehlende/leere/`unknown`/`unavailable` Entities blockieren nichts.
>
> **Türmodus:**
> - `none`: Türkontakte werden ignoriert.
> - `trigger_on_open`: Türöffnung schaltet ein, verhindert aber kein Ausschalten.
> - `hold_while_open`: solange mindestens ein Türkontakt offen ist, wird nicht ausgeschaltet.
>
> **Wichtig:**
> - Räume ohne brauchbaren Luxsensor: Luxsensor einfach leer lassen — die Luxprüfung wird dann automatisch übersprungen (ein Dämmerungswert von `0` bewirkt dasselbe).
> - Bypass und Sofort-An **nicht** denselben Helfer verwenden (die Bedeutungen widersprechen sich).
> - Keinen vorhandenen Raum-Deaktivieren-Helper als Dummy missbrauchen — sonst werden andere Räume versehentlich beeinflusst.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/magicx78/ha-blueprints/main/blueprints/automation/mmwave_light.yaml)

---

### Entity Watchdog — Ausfall-Benachrichtigung (Sensoren, Kontakte, Lichter)

Überwacht beliebige Entities (mmWave-/Bewegungs-/Türsensoren, Lichter, Schalter …) auf Ausfall (`unavailable`/`unknown`) und schickt eine Benachrichtigung mit Handlungs-Hinweis an ein oder mehrere Handys (HA Companion App) und/oder als persistente Benachrichtigung in HA. Gedacht als Begleiter zu den Licht-Blueprints (mmWave Licht, Presence & Light): fällt ein Sensor aus, werten die Licht-Automationen ihn als „keine Präsenz" — der Watchdog sagt rechtzeitig Bescheid, was los ist und was zu tun ist (z. B. Bypass aktivieren, Licht manuell schalten).

**Erfordert: Home Assistant 2024.10.0** (neue `triggers:`/`actions:`-Syntax)

**Features:**
- **Ausfall-Verzögerung einstellbar:** `0` = sofortige Meldung; z. B. 5 min = erst melden, wenn die Entity durchgehend so lange ausgefallen ist (kurze WLAN-Reconnects lösen keine Meldung aus)
- **Mehrere Ziel-Handys** wählbar (Companion App, Geräteauswahl); zusätzlich optionale persistente Benachrichtigung in HA
- **Entwarnung** bei Wiederverfügbarkeit (optional, Standard an) — kommt nur, wenn der Ausfall vorher wirklich gemeldet wurde; ersetzt die Ausfall-Meldung auf dem Handy (Push-`tag`) und entfernt die persistente Benachrichtigung
- **Fehlertolerant:** jede Zustellung läuft mit `continue_on_error` — ein nicht erreichbares Handy stoppt weder die übrigen Zustellungen noch die Automation; `mode: queued` verarbeitet mehrere gleichzeitige Ausfälle nacheinander
- **Handlungs-Hinweis anpassbar** (Freitext mit sinnvollem Default)
- Bewusst als **separate Automation** statt in die Licht-Blueprints integriert: ein Fehler im Watchdog kann die Lichtsteuerung nie blockieren

> **Hinweis:** Der notify-Dienst wird aus dem Gerätenamen abgeleitet (`notify.mobile_app_<geraetename>`). Wurde das Gerät in HA umbenannt, der notify-Dienst aber nicht, den Original-Namen prüfen (Entwicklerwerkzeuge → Aktionen).

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/magicx78/ha-blueprints/main/blueprints/automation/entity_watchdog.yaml)

---

### Automatische Türöffnung – Private BLE (IRK)

Schließt die Tür automatisch auf, wenn eine Person zuerst eine Zone betritt und danach ihr Bluetooth-Gerät im Tür-Bereich auftaucht. Gedacht für Apple-Geräte mit rotierender MAC-Adresse: die Identität kommt aus **Private BLE Device** (IRK), die Bereichsauflösung aus **Bermuda**, das die Private-BLE-Geräte direkt ausliest. Ein iBeacon-Sender auf dem Telefon wird **nicht** benötigt.

Mehrere Geräte sind gleichzeitig auswählbar (z. B. iPhone **und** Apple Watch) — es genügt, wenn eines davon den Tür-Bereich erreicht. So funktioniert es unabhängig davon, was gerade mitgeführt wird.

**Erfordert: Home Assistant 2024.10.0** (neue `triggers:`/`actions:`-Syntax)

> **Voraussetzungen:**
> - Integration **Private BLE Device** mit dem IRK des Geräts eingerichtet
> - Integration **Bermuda BLE Trilateration** installiert — sie legt je Private-BLE-Gerät einen „Area"-Sensor an (`sensor.<gerät>_area`)
> - Mindestens ein Bluetooth-Proxy im Tür-Bereich, und der Bereich ist dem Proxy-Gerät zugewiesen

**Features:**
- **Zweistufig:** erst Zonen-Eintritt (schaltet scharf), dann Tür-Bereich (schließt auf) — ein zufälliger Zonen-Eintritt öffnet nichts
- **Mehrere Geräte** (Telefon, Uhr) mit ODER-Logik — es reicht, wenn eines erkannt wird
- **Mindest-Haltezeit** im Tür-Bereich gegen springende Area-Sensoren (Default 10 s)
- **Abbrechen-Knopf** direkt in der Push-Benachrichtigung — anders als beim Vorbild ist dafür keine zweite Automation nötig
- **`lock.open` statt `lock.unlock`** optional, für Antriebe mit Türöffner-Funktion
- **Nur öffnen, wenn verriegelt** (Default an) — keine unnötigen Schaltvorgänge und Meldungen
- Verglichen wird das Attribut `area_id`, nicht der Anzeigename — den Bereich umbenennen bricht nichts

> **Wichtig:**
> - Bermuda-Area-Sensoren können springen. Die Mindest-Haltezeit deshalb nicht zu klein wählen und den Timeout kurz halten. Die Automation ist ohnehin nur nach einem Zonen-Eintritt scharf.
> - Bei mehreren Instanzen (mehrere Personen oder Türen) je Automation eine eigene **Kennung des Abbrechen-Knopfes** vergeben, sonst bricht ein Knopfdruck alle Instanzen gleichzeitig ab.

*Idee nach [diesem Community-Thread](https://community.home-assistant.io/t/automatically-unlock-your-door-when-getting-in-bluetooth-range/747598); eigenständige Umsetzung ohne iBeacon und ohne Telegram.*

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/magicx78/ha-blueprints/main/blueprints/automation/door_unlock_ble.yaml)

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
| Blueprint mmWave Licht | 2024.10.0 |
| Entity Watchdog | 2024.10.0 |
| Automatische Türöffnung – Private BLE (IRK) | 2024.10.0 |

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

