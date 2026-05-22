One of the main ways that I get and share music is through SoulSeek, which is a P2P application that allows users to download files from one another and share their files.

Something that I had issues with when using SoulSeek as a laptop application was that my files were unable to be shared 24/7, so anyone who is downloading the moment that I shut down my laptop would have to re-download everything from someone else. Also, I had issues with having SMB connect my laptop to my server's storage, while my SoulSeek client has to refresh and find all my shared files, since I'm sharing a network folder. Because of this, I wanted to put it on my 24/7 server.

# Installation
> As usual, we will be using Docker to install this, and using the `slskd` Docker image.

First, since we are using my new hard drive to store my music and video, we have to create the app data there as well.

in my `/mnt/data` directory, which is where my hard drive is mounted, I created these directories:

```bash
justinh@thinkpad-ubuntu:/mnt/data$ tree
.
├── lost+found  [error opening dir]
├── music
├── slskd_config
├── soulseek_downloads
└── video
    ├── movies
    └── shows
```
> Where we can just ignore `lost+found` which is a byproduct of mounting and formatting the drive.

Now, we can begin working on the `docker-compose.yml` file.

```yml
---
version: "3"
services:
  slskd:
    image: slskd/slskd:latest
    container_name: slskd
    ports:
      - "5030:5030"
      - "5031:5031"
      - "50300:50300"
    environment:
      - SLSKD_REMOTE_CONFIGURATION=true
    volumes:
      - /mnt/data/slskd_config:/app:rw
      - /mnt/data/music:/music:rw
      - /mnt/data/video:/video:rw
    user: 1000:1000
    restart: always
```
> I used the provided `docker-compose` from their [page](https://github.com/slskd/slskd/blob/master/docs/docker.md), and added my own information to it such as the directories.

This current configuration didn't work, so I likely had to give permissions to user `1000:1000` to the `slskd` directory on my new drive.

```bash
justinh@thinkpad-ubuntu:~/homelab/slskd$ sudo chown -R 1000:1000 /mnt/data/slskd_config/
justinh@thinkpad-ubuntu:~/homelab/slskd$ docker compose down
justinh@thinkpad-ubuntu:~/homelab/slskd$ vim docker-compose.yml 
justinh@thinkpad-ubuntu:~/homelab/slskd$ docker compose up -d
```
> Where I removed the top, redundant version line that I previously had:

```yml
---
services:
  slskd:
    image: slskd/slskd:latest
    container_name: slskd
    ports:
      - "5030:5030"
      - "5031:5031"
      - "50300:50300"
    environment:
      - SLSKD_REMOTE_CONFIGURATION=true
    volumes:
      - /mnt/data/slskd_config:/app:rw
      - /mnt/data/music:/music:rw
      - /mnt/data/video:/video:rw
    user: 1000:1000
    restart: always
```

One of the first issues I had was that when trying to configure something in the `slskd` settings, it was not saving my config properly and stating that my username/password were `null`. Thus, we have to create an `.env` file to store that information, and then reference it in our `docker-compose`.

```bash
justinh@thinkpad-ubuntu:~/homelab/slskd$ vim .env
justinh@thinkpad-ubuntu:~/homelab/slskd$ cat .env
ADMIN_USER=justinh
ADMIN_PASS=

SLSK_USER=
SLSK_PASS=
```

Where we can then reference this in `docker-compose`:
```bash
justinh@thinkpad-ubuntu:~/homelab/slskd$ cat docker-compose.yml 
---
services:
  slskd:
    image: slskd/slskd:latest
    container_name: slskd
    ports:
      - "5030:5030"
      - "5031:5031"
      - "50300:50300"
    environment:
       - SLSKD_REMOTE_CONFIGURATION=true
       - SLSKD_USERNAME=${ADMIN_USER}
       - SLSKD_PASSWORD=${ADMIN_PASS}
       - SLSKD_SLSK_USERNAME=${SLSK_USER}
       - SLSKD_SLSK_PASSWORD=${SLSK_PASS}
    volumes:
      - /mnt/data/slskd_config:/app:rw
      - /mnt/data/music:/music:rw
      - /mnt/data/video:/video:rw
    user: 1000:1000
    restart: always
```

Then, we have to go into the slskd app to correctly add files to share.
<img src="slskd_conf.png">
