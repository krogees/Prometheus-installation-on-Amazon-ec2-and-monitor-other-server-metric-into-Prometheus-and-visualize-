# Prometheus-installation-on-Amazon-ec2-and-monitor-other-server-metric-into-Prometheus-and-visualize-
In this scenario we are going to install the Prometheus in amazon ec2 server and after that we will configure the node exporter agents in other ec2 server so that Prometheus server can fetch the metrics from other server.

GUIDE

Installation of prometheus
Step1 : Launch amazon linux2 server in public or private server subnet
SSH into that server using key or if server is in private subnet need bastion server to enter into it (ssh -i key_name ec2-user@<private_ip>)
Run the update command
sudo yum update -y
installing the package
Step 2: Go to the official Prometheus downloads page and get the latest download link for the Linux binary.
Step 3: Download the source using curl, untar it, and rename the extracted folder to prometheus-files.
# wget https://github.com/prometheus/prometheus/releases/download/v2.24.0/prometheus-2.24.0.linux-amd64.tar.gz
#tar -xvf prometheus-2.24.0.linux-amd64.tar.gz
#mv prometheus-2.24.0.linux-amd64 prometheus-files
Step 4: Create a Prometheus user, required directories, and make Prometheus the user as the owner of those directories.
sudo useradd — no-create-home — shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
Step 5: Copy prometheus and promtool binary from prometheus-files folder to /usr/local/bin and change the ownership to prometheus user.
sudo cp prometheus-files/prometheus /usr/local/bin/
sudo cp prometheus-files/promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
Step 6: Move the consoles and console_libraries directories from prometheus-files to /etc/prometheus folder and change the ownership to prometheus user.
sudo cp -r prometheus-files/consoles /etc/prometheus
sudo cp -r prometheus-files/console_libraries /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
=====================================
Setup Prometheus Configuration
All the prometheus configurations should be present in /etc/prometheus/prometheus.yml file.
Step 1: Create the prometheus.yml file.
sudo vi /etc/prometheus/prometheus.yml
Step 2: Copy the following contents to the prometheus.yml file.
global:
scrape_interval: 10s
scrape_configs:
— job_name: ‘prometheus’
scrape_interval: 5s
static_configs:
— targets: [‘localhost:9090’]

Step 3: Change the ownership of the file to prometheus user.
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
=====================================
Setup Prometheus Service File
Step 1: Create a prometheus service file.
sudo vi /etc/systemd/system/prometheus.service
Step 2: Copy the following content to the file.
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
— config.file /etc/prometheus/prometheus.yml \
— storage.tsdb.path /var/lib/prometheus/ \
— web.console.templates=/etc/prometheus/consoles \
— web.console.libraries=/etc/prometheus/console_libraries
[Install]
WantedBy=multi-user.target
Step 3: Reload the systemd service to register the prometheus service and start the prometheus service.
sudo systemctl daemon-reload
sudo systemctl start prometheus
Check the prometheus service status using the following command.
sudo systemctl status prometheus
sudo systemctl enable prometheus
======================================
Access Prometheus Web UI
Now you will be able to access the prometheus UI on 9090 port of the prometheus server.
http://<prometheus-ip>:9090/graph
Note : If prometheus server is in private subnet or private server . we can exposed the service via loadbalancer via assigning instance to target group on port 9090
you can refer my Grafana installation in private subnet documentation link given below
https://rohan-j-tiwari.medium.com/grafana-installation-on-amazon-private-ec2-instance-36dda72299d2
=====================================
Installation of node-exporter
Now we are going to configure node exporter agent in other server
Steps 1 : Launch one more Amazon linux ec2 instance.
SSH into that server
yum update -y
step 2 : Create user monitoring with no password
sudo su 
useradd -m -s /bin/bash monitoring 
cd /home/monitoring
Step 3 : Download the latest node-exporter tar file from official documentation
#wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
#tar -xzvf node_exporter-1.2.2.linux-amd64.tar.gz 
#mv node_exporter-1.2.2.linux-amd64.tar.gz /home/monitoring/node_exporter 
#rm -rf node_exporter-1.2.2.linux-amd64.tar.gz 
#chown -R monitoring:monitoring /home/monitoring/node_exporter
Step 3 : Create node-exporter as a service
# vi /etc/systemd/system/node_exporter.service
[Unit] 
Description=Node Exporter 
Wants=network-online.target 
After=network-online.target 
[Service] 
User=monitoring 
ExecStart=/home/monitoring/node_exporter/node_exporter 
[Install] 
WantedBy=default.target
:wq!
Step 4 : Start the service and check the status
systemctl daemon-reload 
systemctl start node_exporter 
systemctl enable node_exporter 
systemctl status node_exporter

status of node-exporter
After starting the Nodeexporter
Note : whitelist the prometheus server Security group ID into the other server Security group inbound rule with the port 9100
prometheus server : source
Node-exporter agent install in other servers : destination
source SG should be whitelist in destination server on port 9100

Configure the Server as Target on Prometheus Server
Step 1: Login to the Prometheus server and open the prometheus.yml file. 
sudo vi /etc/prometheus/prometheus.yml 
Step 2: Under the scrape config section add the node exporter target as shown below. Change 10.142.0.3 with your server IP where you have setup node exporter. Job name can be your server hostname or IP for identification purposes. 
- job_name: ‘node_exporter_metrics’ 
scrape_interval: 5s 
static_configs: 
— targets: [‘10.142.0.3:9100’]
We can add multiple ip in same prometheus.yaml file
global:
scrape_interval: 10s
scrape_configs:
— job_name: ‘prometheus’
scrape_interval: 5s
static_configs:
— targets: [‘localhost:9090’,’10.1.10.28:9100',’10.1.3.150:9100']

Step 3: Restart the prometheus service for the configuration changes to take place. 
sudo systemctl restart prometheus

Prometheus service status
Now, if you check the target in prometheus web UI (http://<prometheus-IP>:9090/targets) , you will be able to see the status as shown below.

We can use the same url as a datasource in grafana and we can configure the metric in grafana dashboard

Cpu utilization dashboard

Memory Utilization

Disk space Utilization

Congratulation we configure the successfully monitoring setup of metrics using node-exporter agent , Prometheus and Grafana
Installation of Grafana link given below

Font: https://rohan-j-tiwari.medium.com/prometheus-installation-on-amazon-ec2-and-monitor-other-server-metric-into-prometheus-and-visualize-c2e5cf4bf977  
  
