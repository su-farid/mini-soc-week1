# Week 1 lab architecture

This diagram documents the intended logical traffic flow. Confirm interface names, addresses, and VM resources against the running VMware configuration before submission.

```mermaid
flowchart TB
    HOST["Dell host browser\n192.168.56.1"]
    subgraph LAB["VMware VMnet4 — host-only 192.168.56.0/24"]
      SIEM["siem-host · Ubuntu Server\n192.168.56.10\nWazuh manager + indexer + dashboard\nDocker Compose"]
      UB["ubuntu-web · Ubuntu\n192.168.56.20\nWazuh agent + web server"]
      WIN["win-client · Windows 10\n192.168.56.30\nWazuh agent + Sysmon"]
    end
    INTERNET["Internet / image registries"]
    HOST <-->|"HTTPS dashboard :443"| SIEM
    UB -->|"Agent telemetry :1514"| SIEM
    WIN -->|"Agent telemetry :1514"| SIEM
    SIEM -->|"NAT adapter for updates"| INTERNET
```

## Network and security notes

- The three VMs share the isolated host-only network.
- Only `siem-host` should retain a permanent NAT adapter.
- The endpoints may use temporary NAT during software installation, then that adapter should be removed.
- Wazuh agent enrollment commonly uses TCP 1515; normal agent event traffic commonly uses TCP 1514. Verify actual listening ports and firewall rules in the lab.
- The diagram describes the target design, not proof that the current services and routes are healthy.
- See [addresses.md](addresses.md) for the recorded address and resource table.
