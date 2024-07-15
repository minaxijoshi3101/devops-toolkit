prometheus server - 
https://www.youtube.com/watch?v=jj38y6f6UpE

vi prometheus.yaml
global:
  scrape_interval: 1m # By default, scrape targets every 15 seconds.

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: node
    static_configs:
      - targets: ['192.168.18.11:9100']

vi docker-compose.yaml

cat docker-compose.yaml
---
version: '3.8'

volumes:
  prometheus-data: {}
  grafana-data: {}
services:
  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node_exporter
    command:
      - '--path.rootfs=/host'
    network_mode: host
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - '9090:9090'
    restart: unless-stopped
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--web.enable-lifecycle'
      - '--config.file=/etc/prometheus/prometheus.yml'
  
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - '3000:3000'
    restart: unless-stopped
    volume:
      - grafana-data:/var/lib/grafana

include new targets in prometheus.yaml and reload the docker container using docker compose restart
docker compose restart prometheus
curl -X POST localhost:9090/-/reload/


http://192.168.18.11:3000/login

admin 
admin

#curl -s -XPOST 'http://192.168.18.8:8080/view/SEH/job/${repo}/job/createItem?name=${repo}_trivy_scan_full' --data-binary @trivy_scan.xml -H "Content-Type:text/xml" -u ${USER}:1118a269715e91e16144ecae875f87060f -H "\$CRUMB"
