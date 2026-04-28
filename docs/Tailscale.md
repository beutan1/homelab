# Tailscale
> I needed some way to access my server's contents safely, so I just installed Tailscale, which is basically WireGuard for end-end encryption & zero trust VPN

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```
I just installed it with that top command, and then got it running with the second. 

Then, to bypass any blocking in places like school, or on airport wifi, I used my home server as an Exit Node, to access the internet as if I was at my apartment/home.

<img src="img/Screenshot 2026-04-27 at 3.56.30 PM.png">
