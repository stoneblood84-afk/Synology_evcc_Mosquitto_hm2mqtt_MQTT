# Synology: evcc, Mosquitto and hm2mqtt with Marstek B2500D

This guide sets up the display of Marstek B2500D battery values in evcc on a Synology NAS. IP addresses and MAC addresses are examples and must be replaced with the values from your installation. Active control of charging and discharging power is not configured by the existing custom meters.

## Overview

```text
B2500D → NAS:1890 (hm2mqtt proxy) → NAS:1883 (Mosquitto)
                                      ↕
                                   hm2mqtt
                              Raw data → JSON
                                      ↓
                                  Mosquitto → evcc → Browser NAS:7070
```

Three containers are used: `evcc`, `mosquitto` and `hm2mqtt`. The proxy and data processing run in the same hm2mqtt container.

| Component | Setting |
|---|---|
| NAS IP | `192.168.1.10` in the examples; replace it everywhere |
| evcc | Host network; browser access on TCP 7070 |
| Mosquitto | Host network; MQTT on TCP 1883 |
| hm2mqtt | Host network; MQTT proxy on TCP 1890 |
| Battery MQTT target | NAS IP, port 1890 |
| evcc, hm2mqtt, MQTT Explorer | Broker NAS IP, port 1883 |

You need a Synology NAS with Container Manager, a fixed NAS address, Wi-Fi-connected batteries that can reach the NAS, and free ports 1883, 1890 and 7070. Allow local traffic through the DSM firewall. Do not expose MQTT through router port forwarding. Host networking does not require additional port mappings.

## 1. Create the folders

Prepare this structure in the `docker` shared folder:

```text
/volume1/docker/
├── evcc/evcc.yaml
├── mosquitto/mosquitto.conf
├── hm2mqtt/docker-compose.yml
└── logs/
```

Change `/volume1` if your shared folder is on another volume. The YAML and configuration files must exist as files, not directories, before the containers start. Save them as UTF-8 and use spaces for YAML indentation.

## 2. Install evcc

1. In **Container Manager → Registry**, download `evcc/evcc:latest` and run it.
2. Use a name such as `evcc-evcc-1-1`, enable automatic restart, and select the **host** network.
3. Add these mounts:

   | NAS path | Container path | Permission |
   |---|---|---|
   | `/volume1/docker/evcc` | `/root/.evcc` | Read/write |
   | `/volume1/docker/evcc/evcc.yaml` | `/etc/evcc.yaml` | Read (read/write also works) |

4. Set `TZ=Europe/Berlin` and keep the image defaults for command and entrypoint.
5. Start evcc after Mosquitto is available. Open it at `http://192.168.x.x:7070`.

The directory mount preserves evcc's internal database. See [evcc Docker installation](https://docs.evcc.io/de/installation/docker/).

![evcc mounts and host network](docs/images/01-evcc.png)

## 3. Install Mosquitto

Use the configuration below in `/volume1/docker/mosquitto/mosquitto.conf`:

```conf
listener 1883
allow_anonymous true
allow_zero_length_clientid true
max_keepalive 120
log_dest stdout
log_type error
log_type warning
log_type notice
log_type information
connection_messages true
```

1. Download and run `eclipse-mosquitto:latest` as `mosquitto-1`.
2. Select the **host** network and enable automatic restart.
3. Mount the file as follows:

   | NAS file | Container file | Permission |
   |---|---|---|
   | `/volume1/docker/mosquitto/mosquitto.conf` | `/mosquitto/config/mosquitto.conf` | Read |

4. Keep `/usr/sbin/mosquitto -c /mosquitto/config/mosquitto.conf` as the command and check the container log after starting.

The mount target must match the path used by the command. As an alternative, mount the complete `mosquitto` folder at `/mosquitto/config`; use only one mount variant. This basic setup does not persist the MQTT database, so devices must publish fresh values after a broker restart.

![Mosquitto mount target](docs/images/02-mosquitto.png)

Source: [official Mosquitto image](https://hub.docker.com/_/eclipse-mosquitto).

## 4. Install hm2mqtt as a project

Create a project named `hm2mqtt` at `/volume1/docker/hm2mqtt` and use this Compose configuration:

```yaml
services:
  hm2mqtt:
    container_name: hm2mqtt
    image: ghcr.io/tomquist/hm2mqtt:latest
    restart: unless-stopped
    network_mode: host
    environment:
      MQTT_BROKER_URL: "mqtt://192.168.x.x:1883"
      MQTT_POLLING_INTERVAL: "60"
      MQTT_RESPONSE_TIMEOUT: "30"
      POLL_CELL_DATA: "false"
      POLL_EXTRA_BATTERY_DATA: "false"
      POLL_CALIBRATION_DATA: "false"
      MQTT_PROXY_ENABLED: "true"
      MQTT_PROXY_PORT: "1890"
      DEVICE_0: "HMJ-2:001a2b3c4d5f"
      DEVICE_1: "HMJ-2:001a2b3c4d5e"
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

Replace the NAS IP, device type and Bluetooth MAC addresses. Do not enable a web portal: port 1890 is the MQTT proxy. Apply changed Compose environment variables by deploying the project again; a simple restart is not enough. See [Synology projects](https://kb.synology.com/de-de/DSM/help/ContainerManager/docker_project?version=7).

The project does not require a volume mount. For the bridge-network alternative shown in the screenshot, remove `network_mode: host` and add:

```yaml
    ports:
      - "1890:1890"
```

Keep the broker URL pointed at the NAS. In bridge mode, `localhost` refers to the hm2mqtt container. Configure the batteries with the NAS IP, never the internal `172.x` address.

![hm2mqtt with bridge networking](docs/images/03-hm2mqtt-container.png)

![hm2mqtt project path](docs/images/04-hm2mqtt-projekt.png)

## 5. Connect the Marstek B2500D to evcc

### Configure MQTT on each battery

Open [hmjs](https://tomquist.github.io/hmjs/) in Chrome on a Bluetooth-capable device, connect to the battery, record the current values, then configure:

| Setting | Battery 1 | Battery 2 |
|---|---|---|
| MQTT host | `192.168.x.x` | `192.168.x.x` |
| MQTT port | `1890` | `1890` |
| MQTT username | e.g. `b2500_1` | e.g. `b2500_2` |

Use different usernames for multiple batteries. Read the **Bluetooth MAC**, not the Wi-Fi MAC. For `DEVICE_n`, use lowercase hexadecimal without colons. `HMJ-2` is only an example device type. The project describes the proxy behavior for firmware 226.5/108.7 and newer versions in its [setup guide](https://github.com/tomquist/hm2mqtt#step-by-step-setup).

The link labelled `esphome-b2500` refers to hm2mqtt. The separate [esphome-b2500 project](https://github.com/tomquist/esphome-b2500) uses an ESP32 and is not required for this MQTT setup.

### Verify MQTT data

In [MQTT Explorer](https://mqtt-explorer.com/), connect to the NAS on port `1883`, without TLS or credentials for this basic setup. Do not use proxy port `1890`. Check for current messages such as:

```text
hm2mqtt/HMJ-2/device/<MAC-address>/data
```

The evcc configuration expects `batteryPercentage`, `outputPower.total`, `solarPower.total` and `batteryCapacity`. Confirm new messages over several polling intervals.

### Configure evcc

For one battery, merge the following sections into the existing `evcc.yaml` without creating duplicate top-level keys:

```yaml
mqtt:
  broker: 192.168.x.x:1883
  topic: evcc

site:
  title: My energy system
  meters:
    battery:
      - Marstek_Battery_1

meters:
  - name: Marstek_Battery_1
    type: custom
    soc:
      source: mqtt
      topic: hm2mqtt/HMJ-2/device/<MAC-address>/data
      jq: .batteryPercentage
      timeout: 120s
    power:
      source: mqtt
      topic: hm2mqtt/HMJ-2/device/<MAC-address>/data
      jq: .outputPower.total - .solarPower.total
      timeout: 120s
    capacity:
      source: mqtt
      topic: hm2mqtt/HMJ-2/device/<MAC-address>/data
      jq: .batteryCapacity / 1000
      timeout: 120s
```

Use the opposite subtraction order if the power sign is reversed. A value of −300 W means charging in the example formula; +200 W means discharging. Division by 1000 is correct only when `batteryCapacity` is in Wh. If that field is unavailable, omit `capacity` until the unit is confirmed.

For one battery, remove the second battery meter and its `DEVICE_1` entry. Adapt or remove the installation-specific APsystems PV meters and their `site.meters.pv` references. The battery display does not replace a complete household-consumption meter.

Sources: [evcc MQTT](https://docs.evcc.io/en/docs/reference/configuration/mqtt), [evcc plugins](https://docs.evcc.io/de/reference/plugins/), and [custom devices](https://docs.evcc.io/de/user-defined-devices/).

### Final test

Start Mosquitto, hm2mqtt and evcc in that order. Verify current JSON messages in MQTT Explorer, restart evcc after YAML changes, then open the NAS address on port 7070. A green container status only means the process is running; current and correct values must appear in evcc.

## 6. Troubleshooting

| Symptom | Check |
|---|---|
| Mosquitto settings have no effect | Match the mount target to the startup command. |
| `not a directory` | Confirm that the YAML/conf mount source is a file. |
| `permission denied` | Check DSM permissions and that the evcc data directory is writable. |
| Connection refused | Check service status, NAS IP, ports, firewall and port conflicts. |
| Batteries reconnect alternately | Use different MQTT usernames and proxy port 1890. |
| No hm2mqtt device data | Check MQTT target, device type and Bluetooth MAC. |
| Stale evcc values | Check polling frequency and increase the timeout beyond 120s when required. |
| Both batteries show the same data | Check that every meter uses its own device ID and topic. |
| Implausible power or capacity | Check raw values, units and the jq calculation. |
| Old hm2mqtt settings remain | Redeploy the project after changing Compose variables. |

Validate the evcc file when supported:

```sh
sudo docker exec evcc-evcc-1-1 evcc checkconfig --config /etc/evcc.yaml
```

This does not replace a live measurement test. See [evcc checkconfig](https://docs.evcc.io/de/reference/cli/evcc_checkconfig/).

## 7. View container logs

Enable SSH in DSM under **Control Panel → Terminal & SNMP → Terminal**, then connect with PuTTY. Find actual container names, including stopped containers:

```sh
sudo docker ps -a --format 'table {{.Names}}\t{{.Status}}\t{{.Image}}'
```

Follow logs in separate PuTTY windows:

```sh
sudo docker logs --tail 200 --timestamps --follow evcc-evcc-1-1
sudo docker logs --tail 200 --timestamps --follow mosquitto-1
sudo docker logs --tail 200 --timestamps --follow hm2mqtt
```

`Ctrl+C` stops the display, not the container. Use `--since 1h` for recent output. Compose logs show only hm2mqtt because evcc and Mosquitto are separate containers:

```sh
cd /volume1/docker/hm2mqtt
sudo docker compose logs --tail 200 --timestamps --follow
```

You can also inspect logs in Container Manager or subscribe to MQTT payloads directly:

```sh
sudo docker exec -it mosquitto-1 mosquitto_sub -h 127.0.0.1 -p 1883 -t 'hm2mqtt/#' -v
```

For optional file logging, mount `/volume1/docker/mosquitto/log` at `/mosquitto/log`, add `log_dest file /mosquitto/log/mosquitto.log`, keep `log_dest stdout`, and configure rotation. Source: [Docker logs](https://docs.docker.com/reference/cli/docker/container/logs/).
