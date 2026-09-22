<div align="center">

# 🛡️ IoT Threat Detection & Security Monitoring Platform

### An On-Premises, AI-Powered Intrusion Detection System for IoT Networks

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![React](https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://reactjs.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-336791?style=for-the-badge&logo=postgresql&logoColor=white)](https://postgresql.org)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue?style=for-the-badge)](LICENSE)

**Final Year Project — BS Software Engineering / Cybersecurity**

*"Because in the age of AI, every business is a target — and cloud-based security is no longer enough."*

</div>

---

## ⚠️ Why This Project Exists

> **Cyber threats have changed. Your security needs to change with them.**

The rise of AI has dramatically lowered the barrier to launching sophisticated cyber attacks. Tools that once required expert knowledge can now be automated, scaled, and directed at any connected device — including the IoT cameras and smart devices that run your business.

**The problem with today's IoT security:**

| Problem | Impact |
|---------|--------|
| Most IDS platforms are **cloud-based** | Your network traffic leaves your premises — privacy risk |
| Manual alert response | Attackers move in seconds; humans respond in minutes |
| Static, non-adaptive models | Zero-day attacks bypass signature-based detection |
| Black-box vendor solutions | No visibility into what is being detected or why |

This platform was built as a **final year university project** with one core purpose:

> **To demonstrate that a fully functional, on-premises IoT Intrusion Detection System can be built with open-source tools — and to raise awareness that every business, regardless of size, can and should deploy one.**

You own your data. You own your response. You own your security.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [System Architecture](#-system-architecture)
- [Features](#-features)
- [Technology Stack](#-technology-stack)
- [Hardware Setup](#-hardware-setup)
- [Network Topology](#-network-topology)
- [Pipeline Deep Dive](#-pipeline-deep-dive)
- [ML Models](#-ml-models)
- [SOAR Playbooks](#-soar-playbooks--automated-response)
- [Dashboard](#-dashboard)
- [Quick Start](#-quick-start)
- [Project Structure](#-project-structure)
- [Attack Testing](#-attack-testing)
- [Performance](#-performance)
- [Configuration Reference](#-configuration-reference)
- [Limitations & Future Work](#-limitations--future-work)

---

## 🔍 Overview

This platform is a **modular, real-time IoT security monitoring system** that operates entirely on-premises. It intercepts live network traffic via a Raspberry Pi edge node (SPAN port mirror), extracts behavioral features using Zeek, scores each network flow using ensemble machine learning, and automatically responds to confirmed threats — all without sending a single packet to the cloud.

### What it does

```
Live Network Traffic (WiFi Camera, BYOD Mobile, Kali Linux)
        ↓
Raspberry Pi — SPAN port mirror → TCP stream to backend
        ↓
7-Thread Detection Pipeline (Backend PC)
        ↓
Zeek — session feature extraction (38 features per flow)
        ↓
ML Inference — Random Forest + Isolation Forest + Baseline Calibration
        ↓
Selective Suricata DPI — on ML-flagged flows only
        ↓
Risk Scoring & Alert Generation
        ↓
SOAR Automated Response (4 playbooks — UFW + iptables)
        ↓
PostgreSQL SIEM Storage + WebSocket → React Dashboard
```

### Who it is for

- **Small and medium businesses** running on-premises IoT infrastructure (cameras, BYOD devices)
- **Security analysts** who need real-time visibility into their IoT network
- **IT administrators** who want automated first-response to network threats
- **Researchers and students** learning about network security, ML-based IDS, and SOAR

---

## 🏗️ System Architecture

![System Architecture Diagram](docs/diagrams/System%20Architecture%20Diagram.png)

> 📁 See [`docs/diagrams/`](docs/diagrams/README.md) for all architecture diagrams with detailed descriptions.


---

## ✨ Features

### 🔴 Real-Time Threat Detection
- Live packet capture from Raspberry Pi edge node over TCP (SPAN mirror)
- Zeek network monitoring framework extracts session-level features in pcap-replay mode
- 38-feature behavioral profile built per network flow
- All device traffic monitored — no IP whitelist, known devices are tagged not filtered
- Sub-second latency from packet ingestion to alert generation

### 🤖 AI-Powered Anomaly Detection
- **Random Forest** classifier — supervised detection of known attack patterns
- **Isolation Forest** — unsupervised detection of behavioral anomalies (novel attacks)
- **Baseline Calibration layer** — dampens RF probability on known-good IoT traffic shapes to reduce false positives on real-device deployments
- Ensemble risk score combines all three signals into a single 0.0–1.0 score
- Models trained on combined IoT datasets (TON_IoT, CICIDS2017, UNSW-NB15) + local testbed captures
- Hot-swap model reloading — retrain without pipeline restart

### 🔎 Deep Packet Inspection
- Suricata-based DPI — runs selectively on ML-flagged flows only (rf_proba > 30% OR ISO anomaly)
- Zeek protocol analysis provides session-level DPI for all flows
- Full session reconstruction from raw PCAP stream

### ⚡ SOAR Automated Response
- 4 pre-built playbooks triggered at configurable risk thresholds
- **Block Attacker** — UFW firewall rule for external threat sources
- **Camera Defense** — iptables rate limiting when IoT camera is targeted
- **Device Quarantine** — network isolation of compromised IoT endpoints
- **Scan Detection** — throttling of port scanning activity
- All playbooks can be **enabled/disabled live from the dashboard** — no restart needed
- Manual override buttons (Unblock / Release Quarantine) in the UI
- All device IPs loaded from `shared/config/system_config.yaml` — no hardcoded IPs in playbooks

### 📊 Interactive Dashboard
- Live alert feed with severity colour-coding (CRITICAL / HIGH / MEDIUM / LOW)
- **Full alert history modal** with severity/type/IP filters and all metadata columns
- Network topology map — interactive, live-updating with animated packet flows
- Attack timeline — ECG/heartbeat-style chart with severity-colored spikes
- Anomaly gauge — semicircular, colour-banded (green/yellow/orange/red)
- Top attackers leaderboard (backend/infrastructure IPs filtered out)
- System Health panel (CPU, memory, pipeline status)
- **SOAR panel** — runtime playbook toggles + recent automated actions + manual controls
- **System Control panel** — start/stop pipeline, NIC picker, log viewer, DB controls, sudo setup
- WebSocket on-connect replay — last 50 DB alerts pushed immediately on connect

### 🗄️ SIEM Storage & Analytics
- All alerts persisted to PostgreSQL with full metadata
- Schema with `NOT NULL` + `DEFAULT` constraints (hardened against partial inserts)
- Risk score, attack classification, protocol, ports, timestamps
- REST API for alert queries, statistics, and device status
- Analyst verdict feedback (TP/FP) — FP flows exported to `ml/datasets/fp_dataset.csv` for retraining

### 🔄 Continuous Learning
- Analyst-confirmed false positives feed back into the training pipeline
- Dataset balancing and preprocessing scripts
- Retrain script that hot-swaps models without downtime

---

## 🛠️ Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Packet Capture** | tcpdump + socat (on Pi) | Raw PCAP stream → backend TCP:9000 |
| **DPI / Feature Extraction** | Zeek 6.x (pcap-replay mode) | Session extraction + 38-feature profiling |
| **Selective DPI** | Suricata | Signature matching on ML-flagged flows |
| **ML — Supervised** | scikit-learn Random Forest | Binary attack/normal classification |
| **ML — Unsupervised** | scikit-learn Isolation Forest | Novel attack / anomaly detection |
| **Baseline Calibration** | Custom `baseline_calibration.py` | False-positive reduction for real-device traffic |
| **Preprocessing** | Custom `FeaturePreprocessor` | Categorical encoding, normalisation |
| **Config Management** | `shared/utils/config_loader.py` | Single-source config — no hardcoded IPs |
| **Pipeline** | Python threading | 7-thread concurrent processing pipeline |
| **API** | FastAPI + Uvicorn | REST endpoints + WebSocket broadcast |
| **Internal Auth** | `x-internal-token` + `secrets.compare_digest` | Secures pipeline→API broadcast endpoint |
| **Database** | PostgreSQL 15 | Alert SIEM storage |
| **SOAR** | Python + UFW/iptables | Automated playbook execution |
| **Dashboard** | React 18 + Recharts | Real-time UI with live WebSocket data |
| **Monitoring** | Prometheus + custom metrics | Pipeline health and throughput |
| **Edge Node** | Raspberry Pi 4 | SPAN destination + PCAP forwarding agent |

---

## 🔌 Hardware Setup

### Required Components

| Component | Role |
|-----------|------|
| **Raspberry Pi 4** (4 GB RAM) | Edge capture node — SPAN destination, streams PCAP to backend |
| **Managed Switch** (SPAN/port-mirror support) | Connects wired devices, mirrors all traffic to Pi |
| **WiFi Router** | Provides wireless access for IoT devices |
| **Backend PC** (i5/i7, 8 GB+ RAM, Ubuntu) | Central detection engine — pipeline, ML, API, dashboard |
| **WiFi Camera** (any RTSP/IP camera) | Monitored IoT device |
| **BYOD Mobile** (Android/iOS) | Monitored IoT device |
| **Kali Linux Machine** | Testbed operator — traffic generation and attack simulation |

### Minimum Backend Requirements

```
CPU:   4 cores (6+ recommended for real-time ML inference)
RAM:   8 GB (16 GB recommended)
OS:    Ubuntu 22.04 LTS
Disk:  20 GB free
```

---

## 🌐 Network Topology

![IoT Network Architecture Overview](docs/diagrams/Project%20Diagram.jpeg)

![IoT Network Architecture Detailed](docs/diagrams/IoT%20Network%20Architecture.png)

| Component | Role |
|-----------|------|
| **Raspberry Pi** | SPAN destination — captures all mirrored traffic, streams raw PCAP to backend |
| **Backend PC** | Central server — detection engine, ML pipeline, API, dashboard |
| **Managed Switch** | Connects all wired devices, mirrors source-port traffic to Pi via SPAN |
| **WiFi Router** | Provides wireless access for IoT devices |
| **WiFi Camera** | Monitored IoT device — captured and streamed video traffic |
| **BYOD Mobile** | Monitored IoT device — wireless network access |
| **Kali Linux** | Testbed operator — traffic generation and controlled attack simulation |
| **Analyst / Admin** | Access and manage the system via dashboard |

> All traffic from source switch ports is mirrored to the Raspberry Pi (SPAN destination).
> The Pi runs `tcpdump | socat` and streams raw PCAP bytes to the Backend PC on TCP port 9000.
>
> **Note:** The exact device IPs depend on your DHCP/static assignment. Always check
> `shared/config/system_config.yaml` for the current values and update `dashboard/.env` to match.

---

## ⚙️ Pipeline Deep Dive

The main pipeline runs as **7 concurrent threads**, each with a dedicated role:

```
Thread 1:  Packet Listener     — TCP server on :9000, receives raw PCAP from Pi
Thread 2:  Zeek Feeder         — Drains QUEUE_1 → /tmp/live.pcap → spawns short-lived zeek -r runs
Thread 3:  Zeek Parser         — Reads conn.log, builds 38-feature dicts, pushes to ML_QUEUE
Thread 4:  ML Inference Worker — Dequeues flows, runs RF + ISO Forest + Baseline Calibration
Thread 5:  DPI Worker          — Runs Suricata on ML-flagged flows, merges results, builds alerts
Thread 6:  SIEM Batch Writer   — Flushes alert queue to PostgreSQL every 2 seconds
Thread 7:  Metrics Worker      — Updates Prometheus queue depth gauge every 1 second
```

> Zeek runs in **pcap-replay mode** (`zeek -r`). Thread 2 manages short-lived Zeek processes.
> There is no persistent `zeek -i` process — this avoids conn.log race conditions.

### Flow of a Single Network Connection

```
1.  Pi captures packet from SPAN port (tcpdump on eth0)
2.  Pi pipes raw PCAP → socat → TCP to backend:9000
3.  T1 receives bytes → writes /tmp/live.pcap → QUEUE_1
4.  T2 reads QUEUE_1 → feeds /tmp/live.pcap to: zeek -r /tmp/live.pcap
5.  Zeek writes session data to conn.log
6.  T3 reads conn.log (inode-aware — resets position only on file replace, not append)
7.  T3 builds 38-feature dict per flow → ML_QUEUE
8.  T4 dequeues flow → runs RF.predict_proba() + IsolationForest.predict()
9.  Baseline Calibration dampens RF score on known-good IoT traffic shape
10. risk_score = 0.6 × rf_proba + 0.4 × iso_anomaly_score
11. send_to_dpi = True if rf_proba > 0.30 OR iso == -1
12. T5: if send_to_dpi → Suricata on raw packets → parse signatures
13. Alert dict built → queue_alert() [non-blocking]
14. SOAR evaluate() checks threshold + rolling window + cooldown
15. If trigger: run matching playbooks (UFW/iptables)
16. T6 batch-flushes alert queue → PostgreSQL every 2s
17. POST /internal/broadcast → API broadcasts to all WebSocket clients
18. Dashboard receives alert via WebSocket → updates all panels instantly
```

### The 38 Features

| Category | Features |
|----------|----------|
| **Volume** | bytes sent/received, packets sent/received, avg packet size |
| **Timing** | duration, inter-arrival time, connection rate |
| **Protocol** | protocol type (TCP/UDP/ICMP), service, connection state |
| **Ports** | source port, destination port, port range entropy |
| **Ratios** | byte ratio, packet ratio, SYN/ACK ratio |
| **History** | connection history flags (RSTRH, SF, REJ, S0, etc.) |

### Adaptive Backpressure

The pipeline uses 4-tier backpressure to prevent `ML_QUEUE` overflow under heavy traffic:

| ML_QUEUE Depth | Mode | Behaviour |
|----------------|------|-----------|
| < 300 | Full throughput | Every flow processed |
| 300 – 800 | 50% sample | Every 2nd flow processed |
| 800 – 2000 | 33% sample | Every 3rd flow processed |
| > 2000 | Sparse | Every 7th flow processed |

### Config Centralization

All device IPs (IoT devices, camera, backend) are loaded from a **single source of truth**:

```
shared/config/system_config.yaml  ←  loaded by shared/utils/config_loader.py
```

No IP is hardcoded in `main_pipeline.py`, `soar_engine.py`, `batch_writer.py`, or any playbook file.

---

## 🧠 ML Models

### Random Forest Classifier
- **Type:** Supervised binary classification (0 = normal, 1 = attack)
- **Training data:** Balanced dataset — 14,816 samples (50/50 normal/attack)
- **Source datasets:** TON_IoT + CICIDS2017 + UNSW-NB15 + local testbed captures
- **Performance:** ~87% accuracy, 0.89 F1-score on attack class
- **Output:** Binary label + continuous probability (0.0–1.0)

### Isolation Forest
- **Type:** Unsupervised anomaly detection
- **Purpose:** Catches novel attack patterns RF has not seen during training
- **Contamination:** 0.15 (calibrated to testbed normal traffic ratio)
- **Output:** +1 (normal) or -1 (anomaly)

### Baseline Calibration (`flow_service/core/baseline_calibration.py`)
- **Purpose:** Reduces false positives on real-device IoT traffic
- **How it works:** Detects known-good IoT flow shapes (normal duration, normal byte counts, stable connection states) and dampens the RF probability for those flows
- **Attack-shaped traffic** (S0/REJ states, near-zero bytes, very short duration) is never dampened
- Wired into `risk_scorer.py` — the `flow` dict is passed alongside the ML scores

### Ensemble Risk Score

```
risk_score = (0.6 × rf_probability) + (0.4 × iso_anomaly_score)

after Baseline Calibration adjustment

Severity mapping:
  CRITICAL  ≥ 0.80  →  SOAR trigger eligible
  HIGH      ≥ 0.60  →  Alert + toast notification
  MEDIUM    ≥ 0.35  →  Alert logged + dashboard
  LOW       < 0.35  →  Log only
```

### Retraining

```bash
# Prepare and balance dataset
python3 -m ml.training.dataset_preparer

# Train both models
python3 -m ml.training.model_training

# Models hot-swap on next inference cycle — no restart needed
# start_pipeline.sh also auto-trains if models are missing at startup
```

---

## 🚨 SOAR Playbooks — Automated Response

The SOAR engine evaluates every alert and triggers playbooks when:
- Risk score ≥ 0.80
- 3+ events from the same source within 60 seconds
- No cooldown active for that source IP (60-second cooldown per IP)
- Source IP passes `ipaddress.ip_address()` validation (command-injection protection)

### Playbook 1 — Block Attacker
```
Trigger:  CRITICAL alert from external IP (not an IoT device)
Action:   sudo ufw insert 1 deny from <ip> to any
Result:   All traffic from attacker dropped at OS firewall level
Revert:   Dashboard "UNBLOCK" button → API POST /api/soar/unblock/<ip>
Log:      logs/siem/soar_engine.log
```

### Playbook 2 — Camera Defense
```
Trigger:  Any attack targeting the WiFi Camera IP
Action:   iptables rate-limit + bandwidth throttle from attacker to camera
Result:   Camera remains operational; attacker's camera traffic throttled
Log:      logs/soar/camera_defense.log
```

### Playbook 3 — Device Quarantine
```
Trigger:  CRITICAL alert sourced FROM a known IoT device (compromised device)
Action:   iptables FORWARD DROP for the device's IP (network isolation)
Result:   Compromised IoT device cannot reach rest of network
Revert:   Dashboard "RELEASE" button → API POST /api/soar/unblock/<ip>
Log:      logs/soar/quarantine.log
```

### Playbook 4 — Scan Detection
```
Trigger:  20+ unique destination ports accessed within 30 seconds
Action:   iptables rate-limit + throttle on scanning source
Result:   Scanning activity slowed; analyst alerted via dashboard toast
Log:      logs/soar/scan_detection.log
```

### Runtime Playbook Toggle
All 4 playbooks can be **enabled or disabled live from the dashboard** without restarting the pipeline. The API writes the current state to `logs/soar/playbook_flags.json`. The SOAR engine reads this file on every evaluation:

```json
{
  "block_attacker":    true,
  "camera_defense":    true,
  "quarantine_device": true,
  "scan_detection":    true
}
```

---

## 📊 Dashboard

The React dashboard provides a full operational picture in real time. It connects via WebSocket — on connect, the last 50 DB alerts are replayed immediately so the feed is never empty.

### Panels

| Panel | Description |
|-------|-------------|
| **Stats Bar** | Total alerts, active threats, blocked IPs, model accuracy |
| **System Control** | Start/stop pipeline, NIC picker, sudo setup, log viewer, DB clear |
| **Live Alert Feed** | Scrolling real-time alerts — click "VIEW ALL" for full history modal |
| **All Alerts Modal** | Full history with severity/attack-type/IP filters + all metadata columns |
| **Network Topology** | Interactive live graph — wired/wireless zones, animated packet flows, node detail panel |
| **Attack Timeline** | ECG/heartbeat-style chart — severity-colored spikes per alert event |
| **Anomaly Gauge** | Semicircular gauge — colour-banded (green/yellow/orange/red) with needle |
| **Severity Pie** | Distribution of CRITICAL/HIGH/MEDIUM/LOW alerts |
| **Top Attackers** | Ranked list — backend/infrastructure IPs filtered out |
| **Devices Panel** | All known IoT devices with status (Normal / Suspicious / Quarantined) |
| **System Health** | CPU, memory, pipeline thread status, DB connection |
| **SOAR Panel** | Runtime playbook toggle switches + recent automated actions + manual override buttons |

### Real-Time Updates

The dashboard connects to `ws://<backend>:8000/ws`. Every alert is broadcast instantly. On first connect, the last 50 alerts are replayed from the database.

---

## 🚀 Quick Start

### Prerequisites

```bash
python3 --version          # 3.10+
node --version             # 16+
npm --version
/opt/zeek/bin/zeek --version
sudo systemctl status postgresql
sudo ufw status
suricata --version
```

### 1. Clone and Set Up

```bash
git clone https://github.com/your-username/iot-threat-detection.git
cd iot-threat-detection

python3 -m venv venv
source venv/bin/activate

pip install -r requirements.txt

cd dashboard && npm install && cd ..
```

### 2. Configure Device IPs

Edit `shared/config/system_config.yaml` to match your actual network:

```yaml
network:
  iot_devices:
    - "192.168.50.10"    # WiFi Camera
    - "192.168.50.40"    # BYOD Mobile
  camera_ip: "192.168.50.10"
  backend_ip: "192.168.2.101"
```

Create `dashboard/.env`:

```env
REACT_APP_API_URL=http://192.168.2.101:8000
REACT_APP_WS_URL=ws://192.168.2.101:8000/ws
```

### 3. Set Up Database

```bash
sudo -u postgres psql -c "CREATE DATABASE fyp_security;"

PGPASSWORD=postgres psql -U postgres -d fyp_security \
    -f services/siem-service/db/schema.sql
```

### 4. Train ML Models

```bash
python3 -m ml.training.dataset_preparer
python3 -m ml.training.model_training

# Verify
ls ml/training/models/
# rf_model.pkl  iso_model.pkl  preprocessor.pkl
```

> `start_pipeline.sh` will auto-train if models are missing.

### 5. Set Your NIC and Start the Backend

```bash
# Find your NIC name
ip link show

# Start pipeline (set ZEEK_INTERFACE to your actual NIC)
export ZEEK_INTERFACE=eth0
bash infrastructure/scripts/start_pipeline.sh

# Verify
curl http://localhost:8000/health
# {"status":"ok"}
```

### 6. Start the Dashboard

```bash
cd dashboard
npm start
# Open http://localhost:3000
```

Use the **System Control** panel to pick your NIC and start/stop the pipeline from the UI.

### 7. Connect the Pi Capture Agent

On the Raspberry Pi:

```bash
# Check your backend's IOT-LAB IP first
ip addr show wlxe009bf6913de | grep inet

# On Pi:
bash ~/edge/capture/capture.sh
```

Dashboard status indicator switches from **OFFLINE** to **LIVE**.

---

## 📁 Project Structure

```
iot-threat-detection/
│
├── dashboard/                     # React frontend
│   ├── .env                       # REACT_APP_API_URL, REACT_APP_WS_URL
│   └── src/
│       ├── App.js                 # Root layout + grid
│       ├── components/
│       │   ├── AlertFeed.js       # Live alert list + "VIEW ALL" trigger
│       │   ├── AllAlertsModal.js  # Full history with filters
│       │   ├── AnomalyGauge.js    # Semicircular colour-banded gauge
│       │   ├── AttackTimeline.js  # ECG-style timeline chart
│       │   ├── DevicesPanel.js    # IoT device status panel
│       │   ├── NetworkTopology.js # Interactive live topology map
│       │   ├── SeverityPie.js     # Alert severity distribution
│       │   ├── SoarPanel.js       # Playbook toggles + recent actions
│       │   ├── StatsBar.js        # Summary counters
│       │   ├── SystemControl.js   # Pipeline start/stop + NIC picker + logs
│       │   ├── SystemHealth.js    # CPU, memory, thread status
│       │   ├── ToastManager.js    # Pop-up notifications for CRITICAL alerts
│       │   └── TopAttackers.js    # Ranked attacker IPs
│       ├── hooks/
│       │   └── useWebSocket.js    # Auto-reconnecting WebSocket hook
│       └── services/
│           └── api.js             # REST API client
│
├── services/
│   ├── api-service/               # FastAPI REST + WebSocket
│   │   └── app/main.py
│   ├── capture-service/           # Queue infrastructure + ingest helpers
│   ├── dpi-service/               # Suricata DPI wrapper
│   │   └── parser/dpi_engine.py
│   ├── flow_service/              # Main pipeline (7-thread orchestrator)
│   │   ├── main_pipeline.py       # Thread orchestrator + watchdog
│   │   ├── packet_listener.py     # T1 — TCP PCAP receiver
│   │   ├── zeek_feeder.py         # T2 — PCAP → zeek -r
│   │   ├── zeek_parser.py         # T3 — conn.log → feature dicts
│   │   ├── state.py               # Shared queues + SYSTEM_STATE
│   │   └── core/
│   │       ├── inference_engine.py
│   │       ├── baseline_calibration.py   # IoT false-positive reduction
│   │       ├── risk_scorer.py
│   │       └── bridge.py
│   ├── siem-service/              # Alert persistence
│   │   ├── db/schema.sql          # Hardened schema (NOT NULL + DEFAULT)
│   │   └── writer/batch_writer.py
│   └── soar-service/              # SOAR engine + 4 playbooks
│       ├── engine/soar_engine.py
│       └── rules/
│           ├── playbook_block_attacker.py
│           ├── playbook_camera_defense.py
│           ├── playbook_quarantine_device.py
│           └── playbook_scan_detection.py
│
├── ml/
│   ├── training/
│   │   ├── model_training.py
│   │   ├── dataset_preparer.py
│   │   └── models/                # rf_model.pkl, iso_model.pkl, preprocessor.pkl
│   ├── features/
│   ├── datasets/
│   │   └── fp_dataset.csv         # False positive feedback dataset
│   └── configs/
│
├── infrastructure/
│   ├── scripts/
│   │   ├── start_pipeline.sh      # Master startup (THE ONLY WAY to start)
│   │   ├── stop_pipeline.sh       # Clean shutdown
│   │   ├── reset_firewall.sh      # Remove all SOAR-added firewall rules
│   │   └── setup_sudo.sh          # Configure sudo rules for pipeline
│   └── monitoring/
│       └── metrics.py             # Prometheus metrics server
│
├── shared/
│   ├── config/
│   │   ├── system_config.yaml     # Single source of truth for all IPs
│   │   └── thresholds.json        # Risk thresholds + SOAR trigger config
│   ├── utils/
│   │   └── config_loader.py       # Loads system_config.yaml for all services
│   └── schemas/
│
├── logs/
│   ├── pipeline/                  # Per-thread log files
│   ├── dashboard/                 # API + Uvicorn logs
│   ├── siem/                      # SOAR engine + batch writer logs
│   └── soar/                      # Per-playbook action logs + playbook_flags.json
│
├── docs/
│   └── RUNBOOK.md                 # Complete operations guide
│
├── requirements.txt
└── README.md
```

---

## 🗡️ Attack Testing

Validated against controlled attack scenarios inside the isolated testbed.

### Simulated Attack Types

| Attack | Method | Expected SOAR Response |
|--------|--------|------------------------|
| **ICMP Flood (DoS)** | `hping3 -1 --flood` | PB1: Block Attacker (UFW) |
| **SYN Flood** | `hping3 -S --flood` | PB1: Block + PB3: Quarantine BYOD |
| **Port Scan** | `nmap -sS` | PB4: Scan Detection + throttle |
| **Camera Flood** | Burst traffic to camera IP | PB2: Camera Defense |
| **Botnet Simulation** | Multi-vector from multiple IPs | Multi-playbook cascade |
| **ARP Spoofing** | `arpspoof` | High-risk alert + analyst notification |

### Quick Attack Test

```bash
# Reset firewall to clean state first
bash infrastructure/scripts/reset_firewall.sh

# ICMP flood from Kali
sudo hping3 -1 --flood <camera_ip>

# Watch SOAR trigger in real time
tail -f logs/siem/soar_engine.log

# After ~60 seconds, verify block applied
sudo ufw status numbered | grep DENY
```

---

## 📈 Performance

Results measured on the on-premises testbed with live traffic.

| Metric | Value |
|--------|-------|
| Alert latency (packet → dashboard) | < 3 seconds |
| ML inference time per flow | ~12 ms |
| Random Forest accuracy | ~87% |
| Attack class F1-score | 0.89 |
| Isolation Forest (novel attack detection) | ✅ |
| SOAR response time (trigger → firewall rule) | < 5 seconds |
| Dashboard WebSocket latency | < 100 ms |
| Pipeline throughput | ~500 flows/minute |
| False positive rate (with baseline calibration) | ~8% |

---

## 🔑 Configuration Reference

### `shared/config/system_config.yaml`

```yaml
network:
  monitored_ips:
    - "192.168.2.101"    # Backend PC
    - "192.168.50.10"    # WiFi Camera
    - "192.168.50.40"    # BYOD Mobile
  iot_devices:
    - "192.168.50.10"    # WiFi Camera
    - "192.168.50.40"    # BYOD Mobile
  camera_ip: "192.168.50.10"
  backend_ip: "192.168.2.101"

database:
  host: "localhost"
  port: 5432
  name: "fyp_security"
  user: "postgres"
  # password: set via DB_PASS env var — never commit credentials
```

> **Update this file** whenever device IPs change. All services load IPs from here via `config_loader.py`.

### `shared/config/thresholds.json`

```json
{
  "anomaly_threshold":            0.65,
  "high_risk_threshold":          0.85,
  "dpi_trigger_threshold":        0.30,
  "false_positive_feedback_weight": 0.3
}
```

### SOAR Severity → Action Mapping

| Severity | Risk Score | Auto Action |
|----------|-----------|-------------|
| CRITICAL | ≥ 0.80 | SOAR playbook eligible |
| HIGH | ≥ 0.60 | Alert + toast notification |
| MEDIUM | ≥ 0.35 | Alert logged + dashboard |
| LOW | < 0.35 | Log only |

### Key Ports

| Port | Service |
|------|---------|
| 3000 | React Dashboard |
| 8000 | FastAPI (REST + WebSocket) |
| 9000 | Pi PCAP TCP stream receiver |
| 9091 | Prometheus metrics |
| 5432 | PostgreSQL |

---

## 📖 Documentation

| Document | Location | Contents |
|----------|----------|----------|
| **Operations Runbook** | `docs/RUNBOOK.md` | Full setup, startup procedure, attack testing, troubleshooting |
| **flow_service** | `services/flow_service/README.md` | 7-thread pipeline, Zeek pcap mode, baseline calibration |
| **api-service** | `services/api-service/README.md` | All API endpoints, internal token, NIC picker |
| **soar-service** | `services/soar-service/README.md` | 4 playbooks, runtime toggle, IP validation |
| **siem-service** | `services/siem-service/README.md` | Batch writer, schema, FP export |
| **capture-service** | `services/capture-service/README.md` | Pi TCP stream path, device table |
| **dpi-service** | `services/dpi-service/README.md` | Suricata worker pool, selective DPI |
| **Dashboard** | `dashboard/README.md` | Components, WebSocket hook, env vars, setup |

---

## ⚠️ Limitations & Future Work

### Current Limitations

- **Testbed scale only** — validated on a small lab, not a full enterprise network
- **Single backend node** — no distributed processing or horizontal scaling
- **IPv4 only** — IPv6 traffic is not yet profiled or classified
- **Zeek pcap-replay mode** — short-lived Zeek processes per batch (not continuous streaming)
- **Device IPs may vary** — requires manual update of `system_config.yaml` per deployment

### Planned Extensions

- [ ] **Docker Compose deployment** — one-command setup for all services
- [ ] **Federated Learning** — distributed model training across edge nodes
- [ ] **MQTT/CoAP DPI** — deeper IoT protocol analysis
- [ ] **CNN-LSTM temporal model** — sequence-based attack detection
- [ ] **Grafana integration** — pre-built security dashboards over Prometheus
- [ ] **CVE correlation** — linking detected patterns to known vulnerability databases
- [ ] **Email/SMS alerting** — out-of-band notification for critical events
- [ ] **IPv6 support**

---

## 🧑‍💻 Contributing

This project is an academic final year project and is open for learning and extension.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add: your feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## 📜 License

Distributed under the **Apache License 2.0**. See `LICENSE` for the full text.

---

## 🙏 Acknowledgements

- **Zeek Network Security Monitor** — zeek.org
- **Suricata IDS** — suricata.io
- **scikit-learn** — ML framework
- **FastAPI** — async Python backend framework
- **React + Recharts** — dashboard UI
- **TON_IoT, CICIDS2017, UNSW-NB15** — public datasets used for model training

---

<div align="center">

**Built as a Final Year Project to demonstrate that on-premises, AI-powered IoT security is achievable by anyone.**

*In the age of AI, the question is not whether your network will be targeted — it is whether you will see it coming.*

PROJECT MEMBERS:
MUHAMMAD AHSAN
RIZWAN ALEEM TAHHA 
SAIM ASAD
MOIZ AHMED

</div>
