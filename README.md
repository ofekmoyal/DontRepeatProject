++ /workspaces/DontRepeatProject/README.md
## DontRepeatProject
Exactly as the title says: not because you can't, but because it will consume your time endlessly.

### Home lab overview

This repo notes an approach I used to expose services from a physical home server to my devices using Tailscale. The server is an older desktop with the following specs:

- CPU: Intel i7-6700 (4 cores / 8 threads)
- GPU: NVIDIA GTX 980 (for optional hardware transcoding)
- RAM: 8 GB
- Role: host for Vaultwarden (Bitwarden-compatible), Plex/Jellyfin, Deluge, Minecraft servers, and other media/game services

### Services hosted

- Vaultwarden (self-hosted Bitwarden implementation) — stores only end-to-end encrypted vaults
- Exit node (Tailscale) — optional egress for remote devices
- Plex / Jellyfin — media streaming (local files + remote streaming when needed)
- Deluge (torrent client) — automatic downloads and seeding
- Minecraft servers — game servers reachable to friends/devices on the tailnet

### How I used Tailscale

- Tailnet mesh: I installed Tailscale on the home server and on client devices so they all join the same private tailnet. This removes the need to punch holes in my home router or expose public ports.
- Service access: instead of opening ports, I access services over Tailscale IPs / MagicDNS names (stable hostnames provided by Tailscale). For example `vaultwarden.home.tail` or the server's Tailscale IP.
- Vaultwarden + E2EE: Vaultwarden stores only the encrypted vault blobs. Clients perform encryption/decryption locally, so hosting it on a machine reachable only via Tailscale preserves confidentiality while avoiding public exposure.
- Exit node: I configured the home server as a Tailscale exit node for optional internet egress from mobile devices. This lets devices route internet traffic through the home connection when needed without adding VPN server complexity.
- Subnet routes / LAN access: when I needed devices on the tailnet to reach services on the home LAN (or other machines), I enabled subnet routing on the server so the tailnet can reach specified subnets.
- NAT and no port-forward: services that normally required public ports (Minecraft, Plex remote access) are reachable via Tailscale directly. For friends without Tailscale, I selectively enable port-forwarding or use Tailscale's sharing features.
- Reverse proxy / HTTPS: I keep a small reverse-proxy (Caddy / nginx) for local host-routing and TLS termination when exposing any service; otherwise I use Tailscale's MagicDNS + direct connections and rely on app-level TLS/E2EE.

### Practical notes and trade-offs

- Performance: the i7-6700 + GTX 980 is adequate for light-to-moderate Plex / Jellyfin streaming. For heavy concurrent transcoding, consider upgrading RAM and GPU or using direct-play where possible.
- Hardware transcoding: older GTX 980 supports NVENC — enable NVIDIA drivers and the container runtime (if using Docker) to allow Plex/Jellyfin to use GPU accel.
- Vaultwarden is E2EE by design, but still keep regular backups of the database and attachments (encrypted). Also enable rate-limits, strong admin password, and keep Vaultwarden updated.
- Security: Tailscale reduces attack surface by avoiding public ports, but keep the server OS, Docker images, and services patched. Use Tailscale ACLs and device authorization to limit who can join the tailnet.

### Example workflow (high level)

1. Install Tailscale on the home server and join your tailnet.
2. Install services (Vaultwarden, Plex/Jellyfin, Deluge, Minecraft) locally or via Docker and bind them to localhost or the LAN interface.
3. Use MagicDNS or the server's Tailscale IP to connect from remote clients; for services that require internet egress, optionally enable the server as an exit node.
4. Configure hardware accel (NVIDIA drivers) in Plex/Jellyfin containers if you need transcoding performance from the GTX 980.

If you'd like, I can add concrete examples: Docker compose snippets for Vaultwarden, Plex/Jellyfin with NVIDIA runtime, Tailscale ACL samples, or a short post-ready paragraph describing the setup for a blog.
