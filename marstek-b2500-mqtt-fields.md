# Marstek B2500 (V2) – MQTT-Feld-Dokumentation

> ⚠️ **Hinweis:** Marstek veröffentlicht kein offizielles Feld-Wörterbuch für dieses MQTT-Topic. Diese Zuordnung basiert größtenteils auf Community-Reverse-Engineering (Home Assistant-Foren, GitHub-Integrationen, insbesondere [hm2mqtt](https://github.com)) und ist **nicht zu 100 % offiziell bestätigt**.
>
> Modellerkennung: V2-Modell, da Felder wie `md`, `sg`/`sp`/`st`, `c0`/`c1`, `m0`–`m3`, `lmo`/`lmi` vorhanden sind.

## Inhaltsverzeichnis

- [Solar-Eingang](#solar-eingang)
- [Batterie](#batterie)
- [Ausgang](#ausgang)
- [Geräteinfo](#geräteinfo)
- [Temperatur](#temperatur)
- [Szene / Modus](#szene--modus)
- [Zeitfenster (Timer)](#zeitfenster-timer-bis-zu-5)
- [Tagesenergie-Statistik](#tagesenergie-statistik)
- [CT / Smart Meter](#ct--smart-meter)
- [Leistungsgrenzen](#leistungsgrenzen)
- [Offene Punkte / Unsicherheiten](#offene-punkte--unsicherheiten)

---

## Solar-Eingang

| Feld | Beispielwert | Bedeutung |
|---|---|---|
| `p1` | 1 | Status Eingang 1 (Bit 0 = lädt, Bit 1 = Durchleitung) |
| `p2` | 1 | Status Eingang 2 |
| `w1` | 50 | Leistung Eingang 1 (W) |
| `w2` | 53 | Leistung Eingang 2 (W) |

## Batterie

| Feld | Beispielwert | Bedeutung |
|---|---|---|
| `pe` | 21 | Ladezustand (SoC) gesamt, in % |
| `kn` | 470 | Batteriekapazität gesamt (Wh) |
| `do` | 100 | DoD-Einstellung (Entladetiefe), in % |
| `lv` | 8 | Entladeschwelle (min. Last vor Ausgangs-Aktivierung), W |
| `b1`, `b2` | 0, 0 | Zusatz-Akkupack 1/2 angeschlossen (nein) |
| `a0` | 21 | SoC Hauptbatterie, % |
| `a1`, `a2` | 0, 0 | SoC Zusatzbatterie 1/2, % |
| `l0` | 2 | Statusflags Hauptbatterie (Bitmaske: Bit1 = lädt) |
| `l1` | 0 | Statusflags Zusatzbatterien |

## Ausgang

| Feld | Beispielwert | Bedeutung |
|---|---|---|
| `o1`, `o2` | 1, 1 | Ausgang 1/2 aktiv |
| `g1`, `g2` | 0, 0 | Ausgangsleistung 1/2 (W) |
| `cd` | 1 | Entlademodus (V1-Bitmaske) bzw. Discharge-Setting (siehe [Hinweise](#offene-punkte--unsicherheiten)) |

## Geräteinfo

| Feld | Beispielwert | Bedeutung |
|---|---|---|
| `vv` | 116 | Firmware-Version (Haupt) |
| `sv` | 6 | Firmware-Subversion |
| `fc` | 202310231502 | FC4.2D-Chip-Version (Datumsstring) |
| `id` | 5 | Geräte-ID |
| `uv` | 107 | Bootloader-Version |
| `am` | 0 | Temperaturwarnstatus |
| `ws` | -35 | WLAN-Signalstärke (dBm) |
| `tc_dis` | 0 | Überschusseinspeisung deaktiviert? (0 = aktiviert) |
| `sm` | 0 | Multi-Geräte-Modus |
| `bn` | 0 | unbekannt |
| `fktc` | 0 | unbekannt |

## Temperatur

| Feld | Beispielwert | Bedeutung |
|---|---|---|
| `tl` | 28 | Min. Zelltemperatur, °C |
| `th` | 29 | Max. Zelltemperatur, °C |
| `tc` | 0 | Lade-Temperaturalarm |
| `tf` | 0 | Entlade-Temperaturalarm |

## Szene / Modus

| Feld | Beispielwert | Bedeutung |
|---|---|---|
| `cj` | 2 | Szene: 0 = Tag, 1 = Nacht, 2 = Dämmerung |
| `cs` | 0 | Lademodus: 0 = gleichzeitig laden+entladen, 1 = erst laden dann entladen |
| `md` | 1 | Adaptivmodus aktiviert |

## Zeitfenster (Timer, bis zu 5)

| Feld | Beispiel | Bedeutung |
|---|---|---|
| `d1`=1, `e1`=18:0, `f1`=23:59, `h1`=210 | Zeitfenster 1 | aktiv, 18:00–23:59, 210 W |
| `d2`=1, `e2`=0:0, `f2`=8:0, `h2`=210 | Zeitfenster 2 | aktiv, 00:00–08:00, 210 W |
| `d3`=0, `e3`=0:0, `f3`=23:59, `h3`=80 | Zeitfenster 3 | inaktiv |
| `d4`=0, `e4`=0:0, `f4`=23:59, `h4`=80 | Zeitfenster 4 | inaktiv |
| `d5`=0, `e5`=0:0, `f5`=23:59, `h5`=80 | Zeitfenster 5 | inaktiv |

Schema pro Zeitfenster `n`: `dn` = aktiv (0/1), `en` = Startzeit, `fn` = Endzeit, `hn` = Leistungsgrenze (W).

## Tagesenergie-Statistik

| Feld | Beispielwert | Bedeutung |
|---|---|---|
| `bc` | 695 | Tägliche Batterieladeenergie (Wh) |
| `bs` | 897 | Tägliche Batterieentladeenergie (Wh) |
| `pt` | 846 | Tägliche PV-Ladeenergie (Wh) |
| `it` | 974 | Tägliche Mikrowechselrichter-Ausgangsenergie (Wh) |

## CT / Smart Meter

| Feld | Beispielwert | Bedeutung |
|---|---|---|
| `sg` | 1 | CT-Sensor verbunden |
| `sp` | 8 | CT-Sollleistung (automatisch), W |
| `st` | 33 | CT gemessene/übertragene Leistung, W |
| `c0` | 3 | Angeschlossene Phase (3 = sucht) |
| `c1` | 0 | CT-Status (0 = nicht in Diagnose) |
| `m0` | -71 | CT-Klemme 1 gemessene Leistung, W |
| `m1` | -77 | CT-Klemme 2 gemessene Leistung, W |
| `m2` | 181 | CT-Klemme 3 gemessene Leistung, W |
| `m3` | 0 | Mikrowechselrichter-Echtzeitleistung, W |
| `ct_t` | 4 | Konfigurierter Zählertyp (4 = Shelly Pro 3EM) |

## Leistungsgrenzen

| Feld | Beispielwert | Bedeutung |
|---|---|---|
| `lmo` | 1698 | Nennausgangsleistung, W |
| `lmi` | 1409 | Nenneingangsleistung, W |
| `lmf` | 0 | Leistung aktuell begrenzt |

---

## Offene Punkte / Unsicherheiten

- **`cd`**: Taucht sowohl als "Discharge Mode" als auch in älteren Dokumenten unterschiedlich belegt auf. Bei V2 wird es laut Dokumentation eher als Steuercode für Anfragen verwendet (z. B. `cd=01`, `cd=15`), nicht als reiner Statuswert. Im hier gezeigten Antwort-Payload ist es jedoch offensichtlich der zurückgemeldete Discharge-Mode-Wert.
- **`fktc`, `bn`, `sm`**: Laut Quelle selbst bei den Entwicklern noch nicht abschließend geklärt.

## Quellen

- Community-Reverse-Engineering (Home Assistant-Foren)
- [hm2mqtt](https://github.com) – ausführlichste bekannte Feldbeschreibung

---

*Diese Dokumentation ist inoffiziell und kann Fehler enthalten. Beiträge/Korrekturen willkommen.*
