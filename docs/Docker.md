# Docker
Something that I knew that I wanted to do was to have an easy way of installing and running programs on my server. One of which that I wanted to get up and running was AdGuard DNS, and I heard that one of the easiest ways of doing that was through Docker. Because of my exposure to Docker in my full-stack development class, I decided to give it a go.

I had to run these series of commands that were fed to me by Gemini for the installation:

```bash
# Update your existing list of packages
sudo apt-get update
sudo apt-get install ca-certificates curl

# Create a directory for the security keys and download Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the official Docker repository to your server's sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
> Besides the last one, these mostly do make a lot of sense

Then, we can simply install the Docker Engine

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

And following this, we can always set docker to have root privileges for quality of life.

```bash
sudo usermod -aG docker $USER
```

And apply the change immediately.
```bash
newgrp docker
```