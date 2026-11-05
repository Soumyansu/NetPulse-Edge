# NetPulse Edge

> Edge network quality monitor on Raspberry Pi 4. It measures telecom KPIs locally, flags SLA breaches at the edge, and streams summaries to AWS IoT Core.

![Python](https://img.shields.io/badge/Python-3.9+-blue)
![AWS IoT Core](https://img.shields.io/badge/AWS-IoT%20Core-orange)
![MQTT](https://img.shields.io/badge/Protocol-MQTT-purple)
![Power BI](https://img.shields.io/badge/Dashboard-Power%20BI-yellow)

## Overview

NetPulse Edge runs on a Raspberry Pi 4 and continuously measures the quality of the network it is connected to. Raw readings are processed on the device, so only aggregated metrics and breach alerts are sent to the cloud. This reduces bandwidth and storage while keeping the data useful for analysis.

## Key Features

- Measures **latency, jitter, packet loss, throughput and Wi-Fi signal strength (RSSI)**
- **Edge aggregation**: computes rolling averages on the device
- **SLA breach detection** with configurable thresholds
- Real-time publishing over **MQTT** to **AWS IoT Core**
- Storage in **DynamoDB** and KPI dashboards in **Power BI**
- Runs as a background service on the Pi

## Architecture

```
[Raspberry Pi 4]
  Collectors (ping / iperf3 / iwconfig)
        |
  Edge Processing (aggregation + SLA checks)
        |  MQTT (TLS)
        v
  AWS IoT Core --> IoT Rule --> DynamoDB --> Power BI
```

## Tech Stack

| Layer | Tools |
|---|---|
| Edge device | Raspberry Pi 4, Raspberry Pi OS |
| Language | Python |
| Messaging | MQTT, AWS IoT Core |
| Storage | DynamoDB (cloud), SQLite (local buffer, if used) |
| Visualisation | Power BI |

## KPIs Tracked

| KPI | Description |
|---|---|
| Latency | Round-trip time to a reference host (ms) |
| Jitter | Variation in latency between consecutive probes (ms) |
| Packet loss | Percentage of probes lost |
| Throughput | Download/upload speed (Mbps) |
| Signal strength | Wi-Fi RSSI (dBm) |

## Setup

### Prerequisites
- Raspberry Pi 4 with Raspberry Pi OS
- AWS account with an IoT Core "Thing", certificates and an IoT Rule to DynamoDB
- Python 3.9+

### Installation
```bash
git clone https://github.com/Soumyansu/[REPO-NAME].git
cd [REPO-NAME]
pip install -r requirements.txt
```

### Configuration
Copy `config.example.json` to `config.json` and fill in your values:

```json
{
  "endpoint": "<your-iot-endpoint>.amazonaws.com",
  "client_id": "netpulse-pi-01",
  "topic": "netpulse/metrics",
  "cert_path": "certs/device.pem.crt",
  "key_path": "certs/private.pem.key",
  "ca_path": "certs/AmazonRootCA1.pem",
  "interval_seconds": 60,
  "sla": { "latency_ms": 100, "packet_loss_pct": 2 }
}
```

> Never commit certificates or keys. Add `certs/` and `config.json` to `.gitignore`.

### Run
```bash
python main.py
```

To run continuously, set it up as a `systemd` service (see `deploy/netpulse.service`).

## Results

- Monitored [N] networks over [X] hours
- Edge aggregation reduced data sent to the cloud by [X]%
- Key finding: [e.g., Network A had X% higher latency during Y time window]

## Project Structure

```
.
├── main.py
├── collectors/
├── edge/
├── mqtt/
├── deploy/
├── config.example.json
└── requirements.txt
```
*(Adjust to match your actual folders.)*

## Future Work

- Add alerting through SNS or email
- Support multiple Pi nodes
- Add anomaly detection on the collected KPIs

## Author

**Soumyansu Sekhar Deo** | [LinkedIn](https://linkedin.com/in/soumyansudeo) | [GitHub](https://github.com/Soumyansu)
