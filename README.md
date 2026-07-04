🛡️ NetSentry — AI-Based Network Anomaly Detection System

NetSentry is a fully functional, machine-learning-based network intrusion detection system that identifies malicious network activity — port scans, DoS/DDoS floods, brute-force login attempts, and data exfiltration — *without relying on hand-written signatures or attack labels*.

It uses an *unsupervised ML ensemble* (Isolation Forest + Autoencoder) to learn what "normal" network traffic looks like, then flags anything that deviates from that baseline. This means it can catch attack patterns it has never seen before — not just known, pre-defined threats.

Includes a live SOC-style (Security Operations Center) web dashboard for real-time monitoring.

---

## 🚀 Features

- *Unsupervised anomaly detection* — no attack labels needed during training
- *Dual-model ensemble*:
  - Isolation Forest — catches structurally abnormal flows (e.g., a scan hitting hundreds of ports)
  - Autoencoder (MLP) — catches subtle multi-feature statistical drift
- *Detects 4 major attack types*:
  - 🔍 Port Scanning
  - 💥 DoS / DDoS Flooding
  - 🔑 Brute-Force Login Attempts
  - 📤 Data Exfiltration
- *Real-time packet capture* (via Scapy) for monitoring actual network interfaces
- *Synthetic traffic simulator* for testing/demo without needing live traffic
- *Live web dashboard* (Flask) — severity breakdown, top targeted ports, attack pattern distribution, auto-refreshing alert stream
- *SQLite-based alert logging* with human-readable explanations for each flagged flow
- *~99% recall, ~92% precision* on held-out synthetic test data

---

## 🧠 How It Works

network-anomaly-detector/
├── src/
│   ├── data_generator.py    # Synthetic traffic generator (normal + 4 attack types)
│   ├── feature_extractor.py # Raw flow → numeric feature vector
│   ├── train_model.py       # Trains Isolation Forest + Autoencoder
│   ├── detector.py          # Scoring engine + SQLite alert logging
│   ├── live_simulator.py    # Streams simulated traffic through the detector
│   └── live_capture.py      # Real packet capture via Scapy (requires admin/root)
├── dashboard/
│   ├── app.py                # Flask backend serving alerts as JSON
│   └── templates/index.html  # Live-updating dashboard UI
├── data/                     # Generated CSV datasets
├── models/                   # Trained model files (.pkl)
├── logs/alerts.db            # SQLite alert log
├── requirements.txt
└── README.md


---

## ⚙️ Installation

```bash
git clone https://github.com/your-username/network-anomaly-detector.git
cd network-anomaly-detector

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt



## ⚙️ Installation

git clone https://github.com/mahinhasnat/network-anomaly-detector.git
cd network-anomaly-detector

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt

For real packet capture (optional):
pip install scapy

Windows users also need Npcap installed (https://npcap.com/#download) with "WinPcap API-compatible Mode" enabled.


## ▶️ Usage

### 1. Generate synthetic training data
python src/data_generator.py

### 2. Train the anomaly detection models
python src/train_model.py

Outputs precision/recall metrics so you can verify detection quality before trusting it.

### 3a. Run with simulated traffic (demo/testing)
python src/live_simulator.py

### 3b. Run with real network traffic (monitors your own machine's interface)
sudo python3 src/live_capture.py --iface eth0        # Linux
sudo python3 src/live_capture.py --iface en0         # macOS
python src\live_capture.py --iface "Wi-Fi"           # Windows (as Administrator)

⚠️ Only monitor networks/interfaces you own or have explicit permission to monitor.

### 4. Launch the live dashboard
python dashboard/app.py

Open http://127.0.0.1:5000 to view live alerts, severity breakdown, and attack pattern statistics.


## 🎛️ Tuning for Your Environment

Parameter: contamination
Location: train_model.py
Effect: Expected % of anomalies in your traffic — lower it if your network is mostly clean

Parameter: error_threshold percentile
Location: train_model.py
Effect: Raise to reduce false positives, lower to catch more borderline cases

Parameter: WINDOW_SECONDS
Location: live_capture.py
Effect: How long to aggregate packets into one flow before scoring

Retrain periodically on your own captured "known-good" traffic so the baseline reflects your actual network rather than synthetic assumptions.


## ⚠️ Limitations

- The default training data is synthetic — it approximates attack shapes but is not a substitute for real datasets (e.g., CICIDS2017) or a live pilot before production use.

- failed_logins detection from real traffic requires application-layer parsing (SSH/FTP/RDP logs) — not populated by raw packet headers alone.

- This is a detection aid for human analysts, not an automated blocking/response system. Pair it with a firewall/IPS for automatic mitigation.
