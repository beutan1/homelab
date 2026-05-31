One of the best things that I could do for myself at this point with my server was to store my own photos & videos without relying on Google Photos. This would save me the hassle of worrying about paying a subscription fee for storage, and having actual storage for things like emails.


I figured that the best way to go about this was to search for a way to store images, and I found [Immich](https://immich.app).

# Installation
As usual, the first thing that we need to do is to get a `docker-compose` from the official documentation. They also provide an `.env` file, so we will be downloading both using the provided commands. 

```bash
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

Before editing any of these, let's first make a directory to store all of our photos & videos.

```bash
justinh@thinkpad-ubuntu:~/homelab/immich$ mkdir /mnt/data/photos
```

Now, we can start to edit the `.env` file.

```env
justinh@thinkpad-ubuntu:~/homelab/immich$ cat .env
# You can find documentation for all the supported env variables at https://docs.immich.app/install/environment-variables

# The location where your uploaded files are stored
UPLOAD_LOCATION=/mnt/data/photos

# The location where your database files are stored. Network shares are not supported for the database
DB_DATA_LOCATION=./postgres

# To set a timezone, uncomment the next line and change Etc/UTC to a TZ identifier from this list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List
TZ=America/Los_Angeles

# The Immich version to use. You can pin this to a specific version like "v2.1.0"
IMMICH_VERSION=v2

# Connection secret for postgres. You should change it to a random password
# Please use only the characters `A-Za-z0-9`, without special characters or spaces
DB_PASSWORD=[secret]

# The values below this line do not need to be changed
###################################################################################
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
```
 > Where we just reference the directory that we just made, and just changed the timezone to something that is local to me. The rest can just be left untouched.

Since they made such an easy to configure set of files, we actually don't have to edit the `docker-compose` at all. Thank you Immich developers!

Then, that's about it! The next steps are what I did to move my Google Photos content all onto Immich.

# Cloud Migration
The first thing that we have to do is to go to https://takeout.google.com/.
	Deselect all
	Select Google Photos
Then, scroll to bottom, go to Next, and make these selections:
<img src="google_takeout_selection.png">
The best tool that you can use for this transfer is [immich-go](https://github.com/simulot/immich-go). Ideally, you use this, but I had too much trouble with re-downloading a large file, and making many requests for a download with google, so I'll download directly onto my laptop and transfer the large file to my server. But... 

> As the name suggests, we need to install `go` to run this. Be sure to install the right version of go, otherwise it may not work properly, I had to use this command:

```bash
sudo snap install go --classic
```


I just ran this to move the file from my laptop to server.
```Bash
rsync -P takeout-20260529T024437Z-3-001.zip thinkpad:~
```
> Where we use rsync and the `-P` flag to show progress and pause the transfer when your computer sleeps to protect against corruption. If there is a broken pipe or anything, you can also just run the same command to continue where you left off!

```bash
client_loop: send disconnect: Broken pipe
rsync(15865): error: unexpected end of file
➜  Downloads rsync -P takeout-20260529T024437Z-3-001.zip thinkpad:~
takeout-20260529T024437Z-3-001.zip
     5727802080  12%  315.29MB/s   00:02:112:34
```

Then finally, after a long transfer, we just have to run `immich-go`.

```bash
~/go/bin/immich-go --server=http://[server_ip]:2283 --api-key=[secret] upload from-google-photos takeout-20260529T024437Z-3-001.zip
```