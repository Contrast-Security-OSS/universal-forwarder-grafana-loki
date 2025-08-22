# Universal Forwarder to Grafana Loki Example

This repository provides an example configuration for sending events from [Contrast's Universal Forwarder](https://docs.contrastsecurity.com/en/adr-universal-forwarder.html) to [Grafana Loki](https://grafana.com/oss/loki/) using [Vector](https://vector.dev/) as the log shipper. The setup is containerized using Docker Compose for easy deployment.


## Overview

- **Contrast Universal Forwarder**: Sends detailed attack event data from Contrast ADR agents.
- **Vector**: Collects and forwards logs from the Universal Forwarder to Grafana Loki.
- **Grafana Loki**: Stores and indexes logs for querying and visualization in Grafana.
- **Docker Compose**: Orchestrates the services for local development and testing.
- **Prometheus (Optional)**: Receives metrics from Vector for monitoring.

---

## Example Dashboards

Below are example screenshots of what you can achieve with this setup in Grafana:

### Contrast Dashboards in Grafana

![Contrast Dashboards in Grafana](media/Contrast%20-%20Dashboards%20-%20Grafana.png)

Dashboard JSON is available for [import to Grafana](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/import-dashboards/) in the [`dashboards`](dashboards/) folder.

### Logs Drilldown in Grafana

![Grafana Logs Drilldown - Contrast](media/Grafana%20Logs%20Drilldown%20-%20Drilldown%20-%20Contrast.png)

---

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/)
- Contrast Universal Forwarder credentials and configuration
- Loki endpoint (cloud or self-hosted)

---

## Setup

### 1. Clone the Repository

```bash
git clone https://github.com/Contrast-Security-OSS/universal-forwarder-grafana-loki.git
cd universal-forwarder-grafana-loki
```

### 2. Configure Secrets


Copy the `.env.tmpl` file to `.env` and fill in the required secrets:

```bash
cp .env.tmpl .env
```

Edit `.env` and provide the necessary values:

```
# Loki configuration
LOKI_ENDPOINT="your-loki-endpoint-url"
LOKI_USER="your-loki-username (if required)"
LOKI_TOKEN="your-loki-api-token (if required)"

# Prometheus configuration (optional, for metrics)
PROMETHEUS_ENDPOINT="your-prometheus-endpoint-url (optional)"
PROMETHEUS_USER="your-prometheus-username (if required)"
PROMETHEUS_TOKEN="your-prometheus-api-token (if required)"
```

> [!NOTE]
> Never commit your `.env` file to version control. Use `.env.tmpl` as a template for sharing required variables.


### 3. (Optional) Configure Vector

Edit `conf/contrast.yaml` to customize sources, sinks, and transforms if needed. By default, it is set up to:
- Ingest logs from Contrast Universal Forwarder
- Forward logs to Loki using the `loki` sink
- Expose metrics for Prometheus

If you do not have Prometheus setup or do not want metrics from Vector to be sent to Prometheus, remove `conf/vector.yaml` before running.

### 4. Start the Stack

```bash
docker-compose up -d
```

This will start Vector.

> [!IMPORTANT]
> The Vector instance must be reachable from the [outbound IP addresses for your Contrast SaaS instance](https://support.contrastsecurity.com/hc/en-us/articles/360000502003-What-is-the-IP-for-Contrast-SaaS) so that the Contrast Universal Forwarder can send JSON events to it. This typically means exposing the relevant Vector port as defined in your `contrast.yaml` and `docker-compose.yml`.

### 5. Configure Universal Forwarder
Follow the [Universal Forwarder documentation](https://docs.contrastsecurity.com/en/adr-universal-forwarder.html) providing the externally accessible Vector URL (e.g., `http://your-public-ip:9000` or your domain name) as the destination for log forwarding.

> [!CAUTION] 
>It is **strongly recommended** to setup TLS on the Vector `http_server` source and to use a `https` URL. Refer to the [Vector documentation](https://vector.dev/docs/reference/configuration/sources/http_server/) for more details on this configuration. Not doing so will mean event data is transmitted in the clear.

### 6. Verify Logs in Loki

- Access your Grafana instance and add Loki as a data source if not already configured.
- Query logs to verify events are being ingested.

---

## References

- [Contrast Universal Forwarder](https://docs.contrastsecurity.com/en/adr-universal-forwarder.html)
- [Vector Documentation](https://vector.dev/docs/)
- [Grafana Loki](https://grafana.com/oss/loki/)
- [Docker Compose](https://docs.docker.com/compose/)
