# Grafana
> I wanted to get a way to view my hardware stats, and this seemed like the way to do it in a visually appealing and digestible manner. 

## Docker installation
As usual, I opted for an installation though its docker image. Start by making a directory for Grafana, and creating its `docker-compose.yml` file within that directory

```bash
mkdir grafana && cd grafana
vim docker-compose.yml
```
and within that file, I used Grafana's documentation to write this `yml` file.

```yml
services:
  grafana:
    image: grafana/grafana
    user: "1000:1000"
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - "./data:/var/lib/grafana"
```
> Something to note here which I made a mistake on earlier is that "1000:1000" as a port number should be in quotations. Furthermore, the port number that this is running on actually has an existing conflict with AdGuard Home. Because of that, I have to change the port number!

```yml
services:
  grafana:
    image: grafana/grafana
    user: "1000:1000"
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3001:3000"
    volumes:
      - "./data:/var/lib/grafana"
```
> Now the external port number of Grafana is 3001, which prevents conflicts.

Since I ran it before, I have to stop the Docker container, and then run it again, within the `/grafana` directory

```bash
docker compose down
docker compose up -d
```

Where there should not be any conflict now.

### Permission issues
```bash
**justinh@thinkpad-ubuntu**:**~/homelab/grafana**$ docker logs grafana

GF_PATHS_DATA='/var/lib/grafana' is not writable.

You may have issues with file permissions, more information here: http://docs.grafana.org/installation/docker/#migrate-to-v51-or-later

mkdir: can't create directory '/var/lib/grafana/plugins': Permission denied

GF_PATHS_DATA='/var/lib/grafana' is not writable.

You may have issues with file permissions, more information here: http://docs.grafana.org/installation/docker/#migrate-to-v51-or-later

mkdir: can't create directory '/var/lib/grafana/plugins': Permission denied

GF_PATHS_DATA='/var/lib/grafana' is not writable.

You may have issues with file permissions, more information here: http://docs.grafana.org/installation/docker/#migrate-to-v51-or-later

mkdir: can't create directory '/var/lib/grafana/plugins': Permission denied

GF_PATHS_DATA='/var/lib/grafana' is not writable.

You may have issues with file permissions, more information here: http://docs.grafana.org/installation/docker/#migrate-to-v51-or-later

mkdir: can't create directory '/var/lib/grafana/plugins': Permission denied

GF_PATHS_DATA='/var/lib/grafana' is not writable.

You may have issues with file permissions, more information here: http://docs.grafana.org/installation/docker/#migrate-to-v51-or-later

mkdir: can't create directory '/var/lib/grafana/plugins': Permission denied

GF_PATHS_DATA='/var/lib/grafana' is not writable.

You may have issues with file permissions, more information here: http://docs.grafana.org/installation/docker/#migrate-to-v51-or-later

mkdir: can't create directory '/var/lib/grafana/plugins': Permission denied

GF_PATHS_DATA='/var/lib/grafana' is not writable.

You may have issues with file permissions, more information here: http://docs.grafana.org/installation/docker/#migrate-to-v51-or-later

mkdir: can't create directory '/var/lib/grafana/plugins': Permission denied

GF_PATHS_DATA='/var/lib/grafana' is not writable.

You may have issues with file permissions, more information here: http://docs.grafana.org/installation/docker/#migrate-to-v51-or-later

mkdir: can't create directory '/var/lib/grafana/plugins': Permission denied

**justinh@thinkpad-ubuntu**:**~/homelab/grafana**$ sudo chown -R 1000:1000 ~/homelab/grafana/data

[sudo] password for justinh:
```
> Where we can see here that there is issues with the grafana directory not being writable. Thus, we have to fix the permissions for that. This just gives permissions to user= "1000:1000" which we specified in the `yml`file in `docker-config`, and allows us write permissions into that directory that had previously denied permissions
