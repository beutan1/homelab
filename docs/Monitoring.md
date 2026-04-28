# Monitoring
> Something that I wanted to do was monitor my server's hardware status from anywhere, and Grafana/Prometheus seemed like the right way to do it.

An issue that I had prior to this was that they were running in separate containers, which doesn't really make sense, considering that these two are interacting with one another. Thus, we will make it in one container using Docker.

## Docker Installation
First, let's make the directory and a `docker-compose.yml` file to run everything.

```bash
mkdir monitoring && cd monitoring
vim docker-compose.yml
```

```yml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus/config:/etc/prometheus
      - ./prometheus/data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    # We keep the port open so you can check the Prometheus UI if needed
    ports:
      - "9090:9090"

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    user: "1000:1000"
    ports:
      - "3001:3000"
    volumes:
      - ./grafana/data:/var/lib/grafana
    depends_on:
      - prometheus
```
> Where this time, we have another service called node-exporter, which is also created by Prometheus. I previously did not have this created, thus I had no way of actually monitoring my hardware information.

Then, we can just make this `prometheus.yml` in the same directory like before.

```bash
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```
> Where this time, we don't have to specify a specific ip address, and just the node exporter at the node exporter port since we are working in the same container.

Furthermore, there is a line that states that grafana `depends_on:` Prometheus.  This just means that it can't launch without Prometheus launching first.

Now, we can go into each directory of the old `/grafana` and the old `/prometheus` and use
```bash
docker compose down
```
 to stop those containers, and then remove their directories.

```bash
rm -rf grafana/ && rm -rf prometheus/
```
Now, we can start the container with all 3 services that we specified. Here, I ran into an issue of port 9090 being occupied, and when I checked, it was by a process called `cockpit` that I had installed when I first set up my server. 

```bash
justinh@thinkpad-ubuntu:~/homelab/monitoring$ sudo lsof -i :9090
[sudo] password for justinh: 
COMMAND     PID       USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd       1       root  164u  IPv6   7033      0t0  TCP *:9090 (LISTEN)
cockpit-t 80062 cockpit-ws    3u  IPv6   7033      0t0  TCP *:9090 (LISTEN)
cockpit-t 80062 cockpit-ws   19u  IPv6 765096      0t0  TCP thinkpad-ubuntu.tail26851a.ts.net:9090->justins-laptop.tail26851a.ts.net:63728 (ESTABLISHED)
```

So, I just removed it.
```bash
sudo apt remove cockpit
```

Then, I made sure to stop the process, since it can still be running.
```bash
sudo systemctl stop cockpit.socket
```

### Important fix
Then, a big issue that I had was that I had to put `prometheus.yml` in the right directory, being `/prometheus/config`, and then using this command to grant access to the file:

```bash
sudo chown -R 1000:1000 ./grafana/
sudo chown -R 1000:1000 ./prometheus/config/
```
> For both of them.

Now, we can just use Grafana as the UI and Prometheus funnels information into it!