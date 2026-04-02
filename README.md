# 📊 Grafana & Prometheus, Node Exporter Installation on AWS EC2

## 🔹 Environment Details

| Component     | Value          |
| ------------- | -------------- |
| Cloud         | AWS            |
| Service       | EC2            |
| Instance Type | `t3.medium`    |
| OS            | Amazon Linux 2 |
| Key Pair      | `grafana.pem`  |
| User          | `ec2-user`     |

---

## 🧩 Architecture Overview

* **Grafana** → Visualization layer (Port `3000`)
* **Prometheus** → Metrics collection & storage (Port `9090`)
* **EC2** → Hosts both services
* **Systemd** → Manages Prometheus & Grafana as services

---

## 🔹 Step 1: Update EC2 System

```bash
sudo yum update -y
```

---

## 🔹 Step 2: Install Grafana (Enterprise Edition)

### Install Grafana RPM

```bash
sudo yum update -y
sudo yum install wget tar -y
sudo yum install make -y
sudo yum install -y https://dl.grafana.com/grafana-enterprise/release/12.2.1/grafana-enterprise_12.2.1_18655849634_linux_amd64.rpm
```

### Verify Grafana Version

```bash
grafana-server --version
```

---

## 🔹 Step 3: Start & Enable Grafana Service

```bash
sudo systemctl start grafana-server
sudo systemctl enable  grafana-server
sudo systemctl status grafana-server
```

✅ **Grafana Web UI**

```
http://<EC2-PUBLIC-IP>:3000
```

**Default Credentials**

* Username: `admin`
* Password: `admin`

---

## WinSCP  >> Install 
https://winscp.net/eng/download.php

## Download Prometheus Setup | LTS/Linux
https://prometheus.io/download/



## 🔹 Step 4: Copy Prometheus Binary to EC2

### SCP from Local Machine (macOS/Linux)

```bash
scp -i /Users/atul/Downloads/monitor.pem \
/Users/atul/Downloads/prometheus-3.5.1.linux-amd64.tar.gz \
ec2-user@ec2-174-129-108-160.compute-1.amazonaws.com:/home/ec2-user/
```

---

## 🔹 Step 5: Move & Extract Prometheus

```bash
sudo mv prometheus-3.5.1.linux-amd64.tar.gz /opt
cd /opt
sudo tar -xvf prometheus-3.5.1.linux-amd64.tar.gz
sudo mv prometheus-3.5.1.linux-amd64 prometheus
sudo rm prometheus-3.5.1.linux-amd64.tar.gz
```

---

## 🔹 Step 6: Create Prometheus System User

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

---

## 🔹 Step 7: Configure Prometheus Directories

```bash
cd /opt/prometheus

sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/

sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

sudo cp prometheus.yml /etc/prometheus/
```

### Set Permissions

```bash
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

---

## 🔹 Step 8: Create Prometheus systemd Service

```bash
sudo nano /etc/systemd/system/prometheus.service
```

### 📄 Paste the Following Configuration

```ini
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

---

## 🔹 Step 9: Update Prometheus Config

edit config file 
```
sudo nano /etc/prometheus/prometheus.yml
```
```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

Start Prometheus Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

✅ **Prometheus Web UI**

```
http://<EC2-PUBLIC-IP>:9090
```

---

# 📊 Node Exporter Setup (Linux)

## 🔹 Overview

**Node Exporter** is a component of Prometheus used to collect system-level metrics like CPU, memory, disk, and network from Linux servers.

---

## 🔹 Step 1: Download & Extract

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
tar xvfz node_exporter-1.10.2.linux-amd64.tar.gz
cd node_exporter-1.10.2.linux-amd64
```

---

## 🔹 Step 2: Run Manually (Test)

```bash
./node_exporter
```

👉 Access in browser:

```
http://<SERVER-IP>:9100/metrics
```

---

## 🔹 Step 3: Install Binary & Create User

```bash
sudo cp node_exporter /usr/local/bin
sudo useradd node_exporter --no-create-home --shell /bin/false
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

---

## 🔹 Step 4: Create Systemd Service

```bash
sudo vi /etc/systemd/system/node_exporter.service
```

### Add below content:

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

---

## 🔹 Step 5: Start & Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
```

---

## 🔹 Step 6: Verify

Open browser:

```
http://<SERVER-IP>:9100/metrics
```

✔ You should see system metrics output

---

## 🔹 Common Metrics Collected

* CPU usage
* Memory usage
* Disk I/O
* Network traffic
* File system stats

---

## 🔹 Integration with Prometheus

Add this in Prometheus config:

```yaml
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['<SERVER-IP>:9100']
```

---

## 🔹 Quick Notes (Exam / Interview)

* Default port → **9100**
* Stateless exporter
* Pull-based monitoring (Prometheus scrapes metrics)
* Works with Grafana dashboards

---

## 🔹 Troubleshooting

* Port not accessible → Check firewall (port 9100)
* Service not starting → `journalctl -u node_exporter`
* Binary issue → Ensure correct architecture (amd64)

---

## 🔹 Step 10: AWS Security Group Configuration

| Service    | Port   | Protocol |
| ---------- | ------ | -------- |
| Grafana    | `3000` | TCP      |
| Prometheus | `9090` | TCP      |
| Node Exporter | `9100` | TCP      |
| SSH        | `22`   | TCP      |

---

## 🎯 Final Validation Checklist

✔ Grafana running
✔ Prometheus running
✔ systemd services enabled
✔ Ports opened in Security Group
✔ Accessible via browser

---

## 🚀 Next Steps (Recommended)

* Add **Prometheus as Grafana Data Source**
* Install **Node Exporter**
* Import **Grafana Dashboards**
* Enable **Alertmanager**
* Add **TLS + Authentication**
* Convert setup into **Terraform / Ansible**

---

## 👨‍💻 Author

**Atul Kamble**

- 💼 [LinkedIn](https://www.linkedin.com/in/atuljkamble)
- 🐙 [GitHub](https://github.com/atulkamble)
- 🐦 [X](https://x.com/Atul_Kamble)
- 📷 [Instagram](https://www.instagram.com/atuljkamble)
- 🌐 [Website](https://www.atulkamble.in)


---

