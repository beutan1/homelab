# Prometheus
> Now that I've installed Grafana, I can feed Prometheus metrics into it to display.

## Docker Installation
As usual, let's make the `docker-compose.yml` file.

```yml
services:
  prometheus:
    image: prom/prometheus
    user: "1000:1000"
    container_name: grafana
    restart: unless-stopped
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro # Readonly for safety
      - ./data:/prometheus # database folder for graphs to be persistent
    command:
        # These extra flags tell Prometheus where to look for its files
        - '--config.file=/etc/prometheus/prometheus.yml'
        - '--storage.tsdb.path=/prometheus'
```
> Something different to note here is that in the documentation, it specifies that prometheus needs its own `prometheus.yml` file. Thus, we need to have a path to it and tell it where to find it in the `command:` header. Same goes for storage, aka the data path.

Then, to avoid the issue we had with Grafana, with permissions, we can simply run this pre-emptively, after making the data directory within `/prometheus`

```bash
sudo chown -R 65534:65534 ~/homelab/prometheus/data
```

### Creating the config file
I used the default provided `prometheus.yml` file from their documentation, and added my own ThinkPad's address, on port `9100`. The reason that port number is chosen is that it is Prometheus's "Node Exporter" Port, for monitoring hardware information like the CPU information.

```yml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']

  - job_name: "thinkpad_stats"
    static_configs:
      - targets: ['100.81.17.16:9100']
```
> Where I just used the IP address from Tailscale, for accessibility anywhere.

<img src="img/Screenshot 2026-04-27 at 4.02.10 PM.png">
> Where we can see here that it already detected a problem! This is just that wifi is not working, but we are using ethernet, so this doesn't really matter.

We can solve this problem, by just editing this service.

```bash
sudo SYSTEMD_EDITOR=vim systemctl edit systemd-networkd-wait-online.service
```

And then we just can add these lines at the bottom

```service
[Service]
ExecStart=
ExecStart=/usr/lib/systemd/systemd-networkd-wait-online --interface=enp0s31f6
```
> Where this overrides the original settings, and only waits for my ethernet to give internet access. enp0s31f6 is my physical ethernet port, which i got using `ip a`, so I used that one.

Now simply restart the service.
```bash
sudo systemctl daemon-reload
sudo systemctl restart systemd-networkd-wait-online.service
```

What happened here was that I got an error, and the issue came from my old `.yaml` file trying to set up wifi. 

The solution for this is to just state that wifi is optional in that `.yaml` file.

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s31f6:
      dhcp4: true
  wifis:
    wlp61s0:
      optional: true
      dhcp4: true
      access-points:
        "Network-Name":
          password: "secret"
```
So in actuality, this fix is better than the service editing, and we should just undo that service editing!

```bash
sudo rm -rf /etc/systemd/system/systemd-networkd-wait-online.service.d/
sudo systemctl daemon-reload
```

And then we can just start the service again.
```bash
sudo systemctl restart systemd-networkd-wait-online.service
```

Where we can see that it is running properly in the Prometheus UI.
<img src="img/Screenshot 2026-04-27 at 4.18.43 PM.png">

Then, we can add it in our Grafana UI, to monitor it, using our ThinkPad's Tailscale IP address on port 9090.

<img src="img/Screenshot 2026-04-27 at 4.21.23 PM.png">
