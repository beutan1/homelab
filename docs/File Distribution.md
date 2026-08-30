When using my laptop to perform file distribution, there was always downtime for both downstream and upstream traffic whenever my computer was to turn off. Furthermore, having to connect to my network storage using SMB constantly was a pain. The only logical solution here was to simply forego the middle-man, and put the application on my server to be run 24/7.
# Installation
> As usual, we will be using Docker to install this, which can be any file-sharing app of your choosing.

First, since we are using my new hard drive to store my music and video, we have to create the app data there as well.

in my `/mnt/data` directory, which is where my hard drive is mounted, I created these directories:

```bash
justinh@thinkpad-ubuntu:/mnt/data$ tree
.
├── lost+found  [error opening dir]
├── music
├── gateway_config
├── downloads
└── video
    ├── movies
    └── shows
```
> Where we can just ignore `lost+found` which is a byproduct of mounting and formatting the drive.

Then, we can just create a directory `sync_gateway` to put our new docker-compose and `.env`

Now, we can begin working on the `docker-compose.yml` file.

```yml
justinh@thinkpad-ubuntu:~/homelab/sync_gateway$ cat docker-compose.yml 
---
services:
  sync_gateway:
    image: ${APP_IMAGE_REPO}
    container_name: file_sync
    ports:
      - "5030:5030"
      - "5031:5031"
      # Share ports
      - "${LOCAL_IP}:50300:50300"
      - "${LOCAL_IP}:50301:50301"
    environment:
      - ${ENV_KEY_REMOTE}=true
      - ${ENV_KEY_USER}=${ADMIN_USER}
      - ${ENV_KEY_PASS}=${ADMIN_PASS}
      - ${ENV_KEY_CLIENT_USER}=${USER}
      - ${ENV_KEY_CLIENT_PASS}=${PASS}
    volumes:
      - /mnt/data/downloads:/app/downloads:rw
      - /mnt/data/gateway_config:/app:rw
      - /mnt/data/music:/music:rw
      - /mnt/data/video:/video:rw
    user: 1000:1000
    restart: always
```
This current configuration didn't work, so I likely had to give permissions to user `1000:1000` to the `sync_gateway` directory on my new drive.

```bash
justinh@thinkpad-ubuntu:~/homelab/sync_gateway$ sudo chown -R 1000:1000 /mnt/data/gateway_config/
justinh@thinkpad-ubuntu:~/homelab/sync_gateway$ docker compose down
justinh@thinkpad-ubuntu:~/homelab/sync_gateway$ docker compose up -d
```

One of the first issues I had was that when trying to configure something in the `sync_gateway` application settings, it was not saving my config properly and stating that my username/password were `null`. Thus, we have to create an `.env` file to store that information, and then reference it in our `docker-compose`.

```bash
justinh@thinkpad-ubuntu:~/homelab/sync_gateway$ cat env_example.txt 
LOCAL_IP=127.0.0.1
APP_IMAGE_REPO=local-registry/data-sync-gateway:stable

# Internal Key Bindings
ENV_KEY_REMOTE=CORE_REMOTE_ENABLED
ENV_KEY_USER=GATEWAY_ADMIN
ENV_KEY_PASS=GATEWAY_TOKEN
ENV_KEY_CLIENT_USER=NODE_CLIENT_ID
ENV_KEY_CLIENT_PASS=NODE_CLIENT_SECRET

# Dummy Credentials
ADMIN_USER=admin
ADMIN_PASS=super_secure_password_placeholder
APP_USER=node_user_01
APP_PASS=node_secret_token_placeholder
```

# Port forwarding
Then now, we need to `port forward` ports 50300 and 50301 so that people can download from the server, bypassing the zero-trust.

# Extra changes
I moved the `.config` files from the external hard drive to the NVME, as it would make more sense for it to be there. Here is the adjusted `docker-compose.yml`

```bash
justinh@thinkpad-ubuntu:~/homelab/sync_gateway$ cat docker-compose.yml 
---
services:
  sync_gateway:
    image: ${APP_IMAGE_REPO}
    container_name: file_sync
    ports:
      - "5030:5030"
      - "5031:5031"
      # Share ports
      - "${LOCAL_IP}:50300:50300"
      - "${LOCAL_IP}:50301:50301"
    environment:
      - ${ENV_KEY_REMOTE}=true
      - ${ENV_KEY_USER}=${ADMIN_USER}
      - ${ENV_KEY_PASS}=${ADMIN_PASS}
      - ${ENV_KEY_CLIENT_USER}=${USER}
      - ${ENV_KEY_CLIENT_PASS}=${PASS}
      - ${ENV_KEY_CLIENT_DESC}=${DESC}
    volumes:
      - /mnt/data/downloads:/app/downloads:rw
      - ./config:/app/config:rw
      - /mnt/data/music:/music:rw
      - /mnt/data/video:/video:rw
    restart: always
```
> Note that I had to remove the last line `user: 1000:1000` due to there being permission errors with the new release of this docker image. The above updated docker-compose is working.
