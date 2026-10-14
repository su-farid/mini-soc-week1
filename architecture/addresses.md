# Lab addresses

| Machine | Role | OS | IP (host-only) | RAM | vCPU | Disk |
|---|---|---|---|---|---|---|
| siem-host | Wazuh in Docker | Ubuntu Server | 192.168.56.10 | 16 GB | 2 | 200 GB |
| ubuntu-web | Linux endpoint + web server | Ubuntu | 192.168.56.20 | 4 GB | 4 | 30 GB |
| win-client | Windows endpoint | Windows 10 | 192.168.56.30 | 4 GB | 2 | 60 GB |
| Dell (host) | Browser access | Windows | 192.168.56.1 (VMnet4) | 32 | - | 1 TB |

Network: VMware VMnet4 (host-only), 192.168.56.0/24, static IPs.
NAT adapter: siem-host only, for image downloads. Removed from endpoints after setup.