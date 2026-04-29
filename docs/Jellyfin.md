# Jellyfin
> Something that I struggled with was finding a good website to watch movies or TV shows that worked consistently. Thus, I figured I could host my own collection, and not rely on someone else's servers, with Jellyfin!

Using the given `docker-compose-yml` on their Docker documentation, I added my own shows & movies directories to the `volumes:` section

```yml
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    # Optional - specify the uid and gid you would like Jellyfin to use instead of root
    user: "1000:1000"
    ports:
      - 8096:8096/tcp
      - 7359:7359/udp
    volumes:
      - /jellyfin/config:/config
      - /jellyfin/cache:/cache
      # Separate directories for movies & shows, both are readonly
      - /home/justinh/video/movies:/media/movies:ro
      - /home/justinh/video/shows:/media/shows:ro
    restart: 'unless-stopped'
    # Optional - alternative address used for autodiscovery
    environment:
      - JELLYFIN_PublishedServerUrl=http://example.com
    # Optional - may be necessary for docker healthcheck to pass if running in host network mode
    extra_hosts:
      - 'host.docker.internal:host-gateway'
```
> Some key mistakes I made was that I didn't define user "1000:1000" and that I didnt put `/media/movies` or `/media/shows` in the given paths. Furthermore I had to define a proper config & cache directory within the jellyfin directory.

Similar to Grafana, there is a permission issue that we must fix with user `1000:1000`.

After making the config & cache directories, I had to state that user `1000:1000` had the correct permissions.

```bash
sudo chown -R 1000:1000 /jellyfin/config /jellyfin/cache
```
> Which I accidentally thought that the config and cache directories had to be explicitly made! But, when running this on its own, it fixes that problem. That's because the compose file uses root level paths, which I didn't reference properly.