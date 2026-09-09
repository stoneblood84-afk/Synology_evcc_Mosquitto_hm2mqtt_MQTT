# EVCC, Mosquitto und hm2mqtt

Konfigurationen aus der Aufgabe „Update hm2mqtt and evcc YAML“.

- `evcc.yaml`: aus `evcc.neu.neu.yaml` unverändert übernommen.
- `mosquitto.conf`: unverändert aus dem Download-Ordner übernommen.
- `hm2mqtt.yaml`: letzte Compose-Konfiguration aus der Aufgabe mit Host-Netzwerk.

Docker-Host: Synology unter `10.168.200.41`. Die Speicher verbinden sich mit dem hm2mqtt-Proxy auf Port `1890`. hm2mqtt und evcc verwenden Mosquitto auf Port `1883`.

In der ursprünglichen Aufgabe wurde bestätigt, dass beide Speicher mit unterschiedlichen MQTT-Benutzernamen stabil verbunden sind. Diese Benutzernamen werden direkt in den Speichern eingestellt.

Die Dateien wurden zur Versionierung übernommen. Es wurde keine Bereitstellung auf der Synology durchgeführt. `hm2mqtt.yaml` enthält nur den hm2mqtt-Dienst, keinen vollständigen Compose-Stack für alle drei Dienste.
