# Anti-DDoS Platform — VPS test installer

One-line install of the Anti-DDoS control plane + XDP data plane onto a single
Debian/Ubuntu **KVM** VPS, for evaluation.

```sh
curl -fsSL https://github.com/Spiuwirkid/anti-ddos-release/releases/latest/download/install.sh | sudo bash
```

## What it installs

- **api-server** + web console (HTTPS on `:8443`)
- **xdp-agent** — attaches generic XDP to the VPS's public NIC and protects this host
- **flow-collector** — detects volumetric attacks from the xdp-agent's inline samples
- **log-forwarder**, **PostgreSQL**

On a single VPS these are wired into the autonomous loop: *xdp-agent samples traffic →
flow-collector detects → api-server decides → xdp-agent drops*. The installer prints the
console URL and a temporary admin password when it finishes.

## Requirements

- A **KVM** VPS — **not** OpenVZ/LXC, which share the host kernel and cannot load XDP.
- Debian 12/13 or Ubuntu 22.04/24.04, **amd64**.
- Root (the one-liner uses `sudo`). ~1 GB RAM is enough for a test.

## Not covered on one VPS

- **bgp-controller** (RTBH/Flowspec escalation) needs an ISP BGP peer — installed but left disabled.
- **l7-proxy / l7-agent** (application-layer defence) need a protected origin — installed but left disabled.

Enable those once you have the matching infrastructure; see the end-of-install notes.

## Notes

- This is a plain software install for a throwaway test box — **not** the hardened appliance
  image (no full-disk encryption, no lockdown).
- Release binaries are obfuscated builds; the platform source is not part of this repository.
- Verify the download with the published `SHA256SUMS` (the installer does this automatically).

## Uninstall

```sh
for s in xdp-agent api-server flow-collector log-forwarder bgp-controller l7-agent l7-proxy orchestrator; do
  sudo systemctl disable --now "$s" 2>/dev/null
done
sudo rm -rf /opt/anti-ddos /etc/anti-ddos /var/lib/anti-ddos /usr/lib/anti-ddos
```
