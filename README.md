# Spring Boot Observability Stack with Prometheus, Grafana, Loki & Promtail

This project demonstrates how to monitor a Spring Boot application using Prometheus and Grafana, and how to collect logs using Loki and Promtail.

---

## ✅ Prerequisites

- Spring Boot App using `application.yml`
- Maven
- RHEL-based Linux Server
- Internet access

---

## 📦 Step 1: Add Dependencies & Configuration

### Micrometer Metrics for Prometheus

Add the following to `pom.xml`:
```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### application.yml for Metrics & Logging
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus

logging:
  file:
    path: /var/log/myapp
    name: app.log
  level:
    root: INFO

server:
  tomcat:
    accesslog:
      enabled: true
      directory: /var/log/myapp
      prefix: access_log
      suffix: .log
      pattern: common
```

---

## 🚀 Start Your App

```bash
./mvnw spring-boot:run
```

---

> The app will expose metrics at: `http://<IP>:9090/actuator/prometheus`

---

## 📈 Step 2: Install & Run Prometheus

```bash
cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
tar -xzf prometheus-2.52.0.linux-amd64.tar.gz
mv prometheus-2.52.0.linux-amd64 prometheus
vim /opt/prometheus/prometheus.yml  
```
Refer to the config file: [Prometheus/prometheus.yml](./Prometheus/prometheus.yml)

Start Prometheus:
```bash
cd /opt/prometheus
./prometheus --config.file=prometheus.yml --web.listen-address=":9091"
```

---

## 📊 Step 3: Install & Configure Grafana

```bash
sudo tee /etc/yum.repos.d/grafana.repo <<EOF
[grafana]
name=Grafana OSS
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
EOF

sudo yum clean all
sudo yum makecache
sudo yum install grafana -y
sudo systemctl enable --now grafana-server
```

### In Grafana UI:
- Add Prometheus as a data source (port `9091`)
- Go to Dashboards and choose the metrics with Prmetheus Datasource.
- Select the metrics and add queries to our desire.
- Save the changes and the Dashboards are ready. 

---

Refer to some of the Dashboards that we have create here - [Dashboards](./Dashboards)

## 📦 Step 4: Install & Configure Loki

```bash
cd /opt
wget https://github.com/grafana/loki/releases/latest/download/loki-linux-amd64.zip
unzip loki-linux-amd64.zip
chmod +x loki-linux-amd64
sudo mv loki-linux-amd64 /usr/local/bin/loki
```

Create config:
```bash
vim /etc/loki-config.yml   
```

Copy from this repo [Loki/loki-config.yml](./Loki/loki-config.yml)

Run Loki:
```bash
nohup loki -config.file=/etc/loki-config.yml > /var/log/loki.log 2>&1 &
```

---

## 📦 Step 5: Install & Configure Promtail

```bash
cd /opt
curl -LO https://github.com/grafana/loki/releases/download/v3.5.2/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
mv promtail-linux-amd64 /usr/local/bin/promtail
chmod +x /usr/local/bin/promtail
```

Create config:
```bash
vim /etc/promtail-config.yml   
```
Refer the yml file on this Repo [Promtail/promtail-config.yml](./Promtail/promtail-config.yml)

Run Promtail:
```bash
nohup promtail -config.file=/etc/promtail-config.yml > /var/log/promtail.log 2>&1 &
```

---

## 📌 Final Step: Grafana Log Integration

- Add **Loki** data source in Grafana (port `3100`)
- Use **Explore** tab to view logs
- Filter logs using labels like `{job="app", type="access"}`

The Logs will be available for querying like this - [Dashboards/Logs_3_Services.jpg](./Dashboards/Logs_3_Services.jpg)


Happy Observability! 🌟
