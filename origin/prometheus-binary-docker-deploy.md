# 部署Prometheus生态组件

# 一、二进制部署

## 1.1、下载安装

### **Prometheus、Node_exporter、Black_exporter**

```bash
prometheus_version=2.28.0 && \
blackbox_exporter_version=0.19.0 && \
node_exporter_version=1.1.2 && \
wget https://github.com/prometheus/prometheus/releases/download/v$prometheus_version/prometheus-$prometheus_version.linux-amd64.tar.gz -P /opt && \
tar -zxvf /opt/prometheus-$prometheus_version.linux-amd64.tar.gz -C /opt && \
ln -s /opt/prometheus-$prometheus_version.linux-amd64 /opt/prometheus && \
wget https://github.com/prometheus/blackbox_exporter/releases/download/v$0.19.0/blackbox_exporter-$0.19.0.linux-amd64.tar.gz -P /opt && \
tar -zxvf /opt/blackbox_exporter-$blackbox_exporter_version.linux-amd64.tar.gz -C /opt &&  \
ln -s /opt/blackbox_exporter-$blackbox_exporter_version.linux-amd64 /opt/blackbox_exporter && \
wget https://github.com/prometheus/node_exporter/releases/download/v$node_exporter_version/node_exporter-$node_exporter_version.linux-amd64.tar.gz -C /opt && \
tar -zxvf /opt/node_exporter-$node_exporter_version.linux-amd64.tar.gz -C /opt && \
ln -s /opt/node_exporter-1.1.2.linux-amd64 /opt/node_exporter && \
rm -rf /opt/*.tar.gz && \
echo -e "export PROMETHEUS_HOME=/opt/prometheus\nexport NODE_EXPORTER_HOME=/opt/node_exporter\nexport BLACK_EXPORTER_HOME=/opt/blackbox_exporter\nexport PATH=\$PATH:\$PROMETHEUS_HOME:\$NODE_EXPORTER_HOME:\$BLACK_EXPORTER_HOME" >> /etc/profile  && \
source /etc/profile  && \
prometheus --version && \
blackbox_exporter --version && \
node_exporter --version
```

### **Grafana**

```bash
apt-get install -y apt-transport-https
apt-get install -y software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
apt-get update
apt-get install grafana -y
grafana-server -vv
```

## 2、配置启动

### **Prometheus**

```bash
mkdir -p /data/prometheus/data && \
bash -c 'cat >/etc/systemd/system/prometheus.service << EOF
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
[Service]
Type=simple
ExecStart=/opt/prometheus/prometheus \
    --config.file=/opt/prometheus/prometheus.yml \
    --storage.tsdb.path=/data/prometheus/data \
    --web.enable-lifecycle
[Install]
WantedBy=multi-user.target
EOF' && \
systemctl daemon-reload && \
systemctl start prometheus.service && \
systemctl status prometheus.service && \
ps -ef |grep prometheus && \
netstat -lanp|grep 9090 && \
systemctl enable prometheus
```

### **Node exporter**

```bash
bash -c 'cat >/etc/systemd/system/node_exporter.service << EOF
[Unit]
Description=Node Exporter
[Service]
Type=simple
ExecStart=/opt/node_exporter/node_exporter
[Install]
WantedBy=multi-user.target
EOF' && \
systemctl daemon-reload && \
systemctl start node_exporter.service && \
systemctl status node_exporter.service && \
ps -ef |grep node_exporter && \
netstat -lanp|grep 9100 && \
systemctl enable node_exporter && \
journalctl -u node_exporter -f
```

### **Blackbox exporter**

```bash
bash -c 'cat >/etc/systemd/system/blackbox_exporter.service << EOF
[Unit]
Description=Blackbox Exporter
[Service]
Type=simple
ExecStart=/opt/blackbox_exporter/blackbox_exporter --config.file=/opt/blackbox_exporter/blackbox.yml
[Install]
WantedBy=multi-user.target
EOF' && \
systemctl daemon-reload && \
systemctl start blackbox_exporter.service && \
systemctl status blackbox_exporter.service && \
ps -ef |grep blackbox_exporter && \
netstat -lanp|grep 9115 && \
systemctl enable blackbox_exporter && \
journalctl -u blackbox_exporter -f
```

### **Grafana**

```bash
mkdir -p /data/grafana/{data,logs} && \
systemctl daemon-reload && \
systemctl start grafana-server && \
systemctl status grafana-server && \
netstat -lanp|grep 3000 && \
systemctl enable grafana-server
```

# 二、Docker部署



```bash
docker run \
    -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
    
docker run -d \
  -p 3000:3000 \
  --name=grafana \
  -e "GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource" \
  grafana/grafana
```











