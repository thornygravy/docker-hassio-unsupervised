# Hassio Unsupervised
Replicating the HAOS experience, without the bloat and limitations.

Docker Containers
- [ghcr.io/home-assistant/home-assistant](#home-assistant)
- [causticlab/hass-configurator-docker](#hass-configurator)
- [eclipse-mosquitto](#eclipse-mosquitto)
- [erisamoe/cloudflared](#cloudflared)
- [esphome/esphome](#esphome)
- [smeagolworms4/mqtt-explorer](#mqtt-explorer)
- [nodered/node-red](#node-red)
- [danmed/tasmobackupv1](#tasmobackup)
- [koenkk/zigbee2mqtt](#zigbee2mqtt)
- [zwavejs/zwave-js-ui](#zwave-js-ui)


## Docker Compose

### Home Assistant

```yaml
services:
  homeassistant:
    container_name: homeassistant
    image: "ghcr.io/home-assistant/home-assistant:latest"
    devices:
      - /dev/dri/renderD128:/dev/dri/renderD128
    volumes:
      - /appdata/homeassistant/config:/config
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    privileged: true
    network_mode: host
```


### Hass Configurator

```yaml
services:
  hass-configurator-docker:
    image: causticlab/hass-configurator-docker
    container_name: hass-configurator
    restart: unless-stopped
    volumes:
      - /appdata/hass-configurator:/config
      - /appdata/homeassistant/config:/homeassistant
    ports:
      - 3218:3218
    environment:
      HC_BASEPATH: "/homeassistant"
    healthcheck:
      disable: true
```

### Eclipse Mosquitto 

```yaml
services:
  mosquitto:
    image: eclipse-mosquitto:latest
    hostname: mosquitto
    container_name: mosquitto
    restart: unless-stopped
    ports:
      - "1883:1883"
      - "9003:9001"
    volumes:
      - /appdata/mosquitto/config:/mosquitto/config:rw
      - /appdata/mosquitto/data:/mosquitto/data:rw
      - /appdata/mosquitto/log:/mosquitto/log:rw
```

### Cloudflared

```yaml
services:
  cloudflared:
    container_name: cloudflared
    image: erisamoe/cloudflared
    restart: unless-stopped
    privileged: true
    volumes:
      - /appdata/cloudflared:/etc/cloudflared
      - /appdata/cloudflared/oldcert:/.cloudflared
    command: tunnel run hassio
```

### Esphome

```yaml
version: '3'
services:
  esphome:
    container_name: esphome
    image: esphome/esphome:2023.11.3
    volumes:
      - /appdata/esphome/config:/config
      - /etc/localtime:/etc/localtime:ro
    environment:
      - ESPHOME_DASHBOARD_USE_PING=true
    restart: always
    privileged: true
    network_mode: host
    healthcheck:
      disable: true
```

### mqtt-explorer

```yaml
version: '3'
name: mqtt-explorer
services:
  mqtt-explorer:
      container_name: mqtt-explorer
      ports:
        - 4000:4000
      volumes:
        - /appdata/mqtt-explorer/config:/mqtt-explorer/config
      image: smeagolworms4/mqtt-explorer
      restart: unless-stopped
      healthcheck:
        disable: true
```

### node-red

```yaml
services:
  node-red:
    container_name: node-red
    image: nodered/node-red:latest
    environment:
      - TZ=America/New_York
      - NODE_RED_CREDENTIAL_SECRET:$password
    ports:
      - "1880:1880"
    networks:
      - node-red-net
    volumes:
      - /appdata/node-red/data:/data
    privileged: true
    restart: unless-stopped
    user: root
volumes:
  node-red-data:
networks:
  node-red-net:
```

### tasmobackup

```yaml
services:
  tasmobackup:
    ports:
      - '8259:80'
    volumes:
      - /appdata/tasmobackup/data:/var/www/html/data
    environment:
      - DBTYPE=sqlite
      - DBNAME=data/tasmobackup
    container_name: tasmobackup
    image: 'danmed/tasmobackupv1:latest'
    restart: unless-stopped
    privileged: true
```

### zigbee2mqtt

```yaml
services:
  zigbee2mqtt:
    container_name: zigbee2mqtt
    image: koenkk/zigbee2mqtt:latest
    environment:
      - TZ=America/New_York
    volumes:
      - /appdata/zigbee2mqtt:/app/data
      - /run/udev:/run/udev:ro
    ports:
      - 8099:8080
    restart: unless-stopped
```

### zwave-js-ui

```yaml
services:
  zwave-js-ui:
    container_name: zwave-js-ui
    image: zwavejs/zwave-js-ui:latest
    restart: always
    tty: true
    privileged: true
    stop_signal: SIGINT
    environment:
      - SESSION_SECRET=mysupersecretkey
      - TZ=America/New_York
    networks:
      - zwave
    devices:
      - /dev/serial/by-id/usb-Nabu_Casa_ZWA-2_EXAMPLE-if00:/dev/zwave
    volumes:
      - /appdata/zwave-js-ui/zwave-config:/usr/src/app/store
    ports:
      - '8091:8091' # interface
      - '3000:3000' # websocket
networks:
  zwave:
volumes:
  zwave-config:
    name: zwave-config
```

## Notes


## License

MIT License. See [LICENSE](LICENSE) for details.
