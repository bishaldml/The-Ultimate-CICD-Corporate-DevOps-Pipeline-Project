# Monitoring
1. System-level Monitoring (CPU and RAM) using node-exporter.
2. Website monitoring using blackbox-exporter.
3. Leveraged (Prometheus and Grafana) for effective Visualization and Analysis.

### Step-1: Create an EC2 instance for monitoring.
```
Name: Monitor
AMI: Ubuntu
Instance_Type: t2.medium
Storage: 25
```

### Step-2: Install prometheus
```
sudo apt update 
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
tar -xvf prometheus-2.52.0.linux-amd64.tar.gz 
rm prometheus-2.52.0.linux-amd64.tar.gz
cd prometheus-2.52.0.linux-amd64/
./prometheus &  
```

### Step-3: Install Grafana
```
sudo apt-get install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_10.4.0_amd64.deb
sudo dpkg -i grafana-enterprise_10.4.0_amd64.deb
sudo /bin/systemctl start grafana-server
```

### Step-4: Install BlackBox for WebSite monitoring
```
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar -xvf blackbox_exporter-0.25.0.linux-amd64.tar.gz
rm blackbox_exporter-0.25.0.linux-amd64.tar.gz
cd blackbox_exporter-0.25.0.linux-amd64/
./blackbox_exporter &
```

### Step-5: Install Node Exporter on Ec2 instances you want to monitor
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.8.1.linux-amd64.tar.gz 
rm node_exporter-1.8.1.linux-amd64.tar.gz
cd node_exporter-1.8.1.linux-amd64/
./node_exporter &
```

---

Browse Promethues on: ```Monitor_instace_IP:9090```

Browse Grafana on: ```Monitor_instace_IP:3000```

Browse BlackBox on: ```Monitor_instace_IP:9115```
