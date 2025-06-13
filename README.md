![License](https://img.shields.io/badge/License-GNU%20GPL-blue.svg)
![Language](https://img.shields.io/badge/docker-%230db7ed.svg)
![License](https://img.shields.io/badge/mikrotik-routeros-orange)
![License](https://img.shields.io/badge/loki-logs-blueviolet)

### Description

This is a simplified logging stack for Mikrotik RouterOS devices based on
[syslog-ng](https://www.syslog-ng.com/),
[promtail](https://grafana.com/docs/loki/latest/clients/promtail/), and
[Loki](https://grafana.com/docs/loki/latest).

This stack provides centralized Mikrotik log processing and storage, designed to
work with an external Grafana instance for visualization.

### Requirements:

[Docker Compose](https://docs.docker.com/compose/install/)

### Install & Getting Started:

- Clone this repository (or download zip with wget)

```
git clone https://github.com/agildw/loki-stack.git
cd loki-stack
```

#### Start the logging stack

```
docker compose up -d
```

The stack will start the following services:

- **Loki** (port 3100): Log aggregation system
- **Promtail** (ports 1514, 9080): Log shipping agent
- **syslog-ng** (ports 514/udp, 601/tcp): Syslog server

#### Mikrotik Centralized Logging configuration

To configure your Mikrotik devices to send logs to this stack, you need to set
up remote logging. Replace XX.XX.XX.XX with your docker compose host IP address:

```
/system logging action
set remote bsd-syslog=yes name=remote remote=XX.XX.XX.XX remote-port=514 src-address=0.0.0.0 syslog-facility=local0 syslog-severity=auto target=remote
```

Next, configure relevant log topics to use this remote action:

```
/system logging
set 0 action=remote prefix=:Info
set 1 action=remote prefix=:Error
set 2 action=remote prefix=:Warning
set 3 action=remote prefix=:Critical

add action=remote disabled=no prefix=:Firewall topics=firewall
add action=remote disabled=no prefix=:Account topics=account
add action=remote disabled=no prefix=:Caps topics=caps
add action=remote disabled=no prefix=:Wireless topics=wireless
```

You can extend the list above as needed, following
[Mikrotik's description](https://help.mikrotik.com/docs/display/ROS/Log) of the
log topics used by various RouterOS facilities.

#### Connecting to External Grafana

To connect your external Grafana instance to this Loki stack:

1. In your Grafana instance, go to **Configuration > Data Sources**
2. Add a new **Loki** data source
3. Set the URL to: `http://YOUR_DOCKER_HOST_IP:3100`
4. Click **Save & Test** to verify the connection

You can then create dashboards to visualize your Mikrotik logs or import
existing Loki dashboard templates.

#### Accessing Services

- **Loki API**: http://localhost:3100
- **Promtail metrics**: http://localhost:9080
- **Syslog ingestion**:
  - UDP port 514 (standard syslog)
  - TCP port 601 (syslog-ng specific)
  - TCP port 1514 (promtail syslog receiver)

## Overview of components used in this project:

- [Loki](https://grafana.com/oss/loki/): an open-source log aggregation system
  inspired by Prometheus
- [promtail](https://grafana.com/docs/loki/latest/clients/promtail/): an
  open-source agent to deliver the logs to Loki
- [syslog-ng](https://www.syslog-ng.com/): an open-source log server
  implementing the syslog protocol

## Network Architecture

```
Mikrotik Devices → syslog-ng (UDP:514) → promtail (TCP:1514) → Loki (HTTP:3100) → External Grafana
```

## Troubleshooting

- If logs don't appear immediately after configuration, restart the containers:
  ```
  docker compose restart
  ```
- Check container logs for any issues:
  ```
  docker compose logs loki
  docker compose logs promtail
  docker compose logs syslog-ng
  ```
- Verify that your Mikrotik devices can reach the Docker host on the configured
  ports
- Ensure your external Grafana can access the Docker host on port 3100
