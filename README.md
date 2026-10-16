# Mini-SOC in a Box — Week 1

Week 1 of a 12-week Blue Team capstone. The goal is to build an isolated virtual lab, deploy Wazuh, enrol Linux and Windows agents, and verify that endpoint events can be investigated from the dashboard.

## Lab overview

- **Wazuh version recorded during setup:** 4.14.7 (single-node Docker Compose)
- **Hypervisor:** VMware Workstation
- **Host-only network:** VMnet4, `192.168.56.0/24`
- **VM addresses:** `siem-host` — `192.168.56.10`; `ubuntu-web` — `192.168.56.20`; `win-client` — `192.168.56.30`
- **Architecture/address table:** [architecture/addresses.md](architecture/addresses.md)
- **Evidence:** [evidence/](evidence/)
- **Build journal:** [journal.md](journal.md)

> **Current status:** The repository contains screenshots and notes for the initial build, agent setup, and five verification events. The journal records a Filebeat-to-indexer `401 Unauthorized` problem. Until that is resolved and re-tested, do not treat dashboard ingestion as currently healthy. The Week 1 persistence test (W4) is also not yet documented as passed.

## Start the lab

1. Power on `siem-host` and log in.
2. In the Wazuh Compose directory, run:
   ```bash
   docker compose start
   docker compose ps
   ```
3. Confirm all three Wazuh containers are running. If they are not, inspect `docker compose logs --tail=100` before proceeding.
4. Open `https://192.168.56.10` from the host computer. A browser certificate warning can be expected for a lab/self-signed certificate; verify that you are connecting to your own lab IP before proceeding. Do not disable certificate validation globally.
5. Start `ubuntu-web` and `win-client`.
6. Check Agents in the dashboard and confirm both agents show Active.
7. Generate a test event and verify that it is searchable in the dashboard. If events exist in `alerts.json` but not in the dashboard, follow the open issue in [journal.md](journal.md).

## Stop the lab

1. Shut down `win-client`.
2. Shut down `ubuntu-web`.
3. On `siem-host`, from the Compose directory, run:
   ```bash
   docker compose stop
   ```
4. Shut down `siem-host`.

**Do not run `docker compose down -v`.** Removing volumes can delete persisted index and alert data. Keep secrets out of shell history, screenshots, and Git.

## Wazuh platform (W1–W5)

The build notes identify Wazuh image tags as 4.14.7 and show a screenshot of the Compose container list. Re-check the live deployment before claiming it is healthy:

```bash
docker compose ps
docker compose images
docker stats --no-stream
```

Record the actual image tags and idle memory values from the current run. The existing values below are historical measurements from the repository, not a guarantee of the current run.

| Container | Previously recorded memory | Previously recorded memory limit % | Previously recorded CPU |
|---|---:|---:|---:|
| `wazuh.indexer` | 1.355 GiB | 35.88% | 1.18% |
| `wazuh.manager` | 407.4 MiB | 10.53% | 1.05% |
| `wazuh.dashboard` | 207.9 MiB | 5.37% | 0.30% |

Evidence: [container list](evidence/03-docker-compose-ps.png), [memory stats](evidence/12-docker-stats.png), [HTTPS dashboard](evidence/04-dashboard-https.png).

The browser certificate warning is expected if the deployment uses a self-signed or otherwise locally untrusted certificate. Confirm the certificate details and lab IP; do not claim the certificate is publicly trusted.

## Agents and verification events

The repository includes screenshots named for the following events. Re-open them and confirm the agent, rule description, and rule level are legible before submission.

| # | Event | Agent | Recorded rule description | Recorded level | Evidence |
|---|---|---|---|---:|---|
| 1 | Failed SSH login | `ubuntu-web` | sshd: Attempt to login using a non-existent user (rule 5710) | 5 | [screenshot](evidence/07-event1-ssh.png) |
| 2 | Command run with sudo | `ubuntu-web` | Successful sudo to ROOT executed (rule 5402) | 3 | [screenshot](evidence/08-event2-sudo.png) |
| 3 | New local Linux user | `ubuntu-web` | New user added to the system (rule 5902) | 8 | [screenshot](evidence/09-event3-adduser.png) |
| 4 | Windows program start via Sysmon | `win-client` | Discovery activity executed | 3 | [screenshot](evidence/10-event4-sysmon.png) |
| 5 | Failed Windows logon | `win-client` | Logon Failure — Unknown user or bad password (rule 60122) | 5 | [screenshot](evidence/11-event5-winlogon.png) |

## Event path example (V2)

Example event: `sudo ls /root` on `ubuntu-web`.

1. Linux records the authentication/sudo event in its authentication log.
2. The Wazuh agent's logcollector reads the configured log file.
3. The agent sends the event to the manager over the configured agent connection (normally TCP 1514).
4. The manager decodes the event and evaluates Wazuh rules.
5. The manager writes matching alerts to `alerts.json`.
6. Filebeat forwards alerts to the indexer.
7. The indexer stores the alert in the Wazuh alerts index.
8. The dashboard queries the indexer and presents the event.

The journal records that this pipeline previously failed at the Filebeat-to-indexer step with `401 Unauthorized`. The root cause and fix must be verified using the actual deployment configuration and official Wazuh 4.14 documentation; do not assume the dashboard admin password is interchangeable with internal service credentials.

## Network isolation and security

- The intended design is a host-only lab network for all three VMs.
- Only `siem-host` should retain a permanent NAT adapter. Remove temporary NAT from both endpoints after software installation.
- Verify the adapter configuration in VMware and confirm endpoints do not have a permanent Internet route.
- `.gitignore` helps prevent accidental additions but does not remove files already tracked or prove that the Git history is free of secrets. Review tracked files and repository history before submission.
- Keep passwords, private keys, certificates, tokens, and internal service credentials out of the repository.

## VM resources — verify against the assignment

The current [address table](architecture/addresses.md) records these configured resources: `siem-host` 16 GB RAM / 2 vCPU / 200 GB disk; `ubuntu-web` 4 GB / 4 vCPU / 30 GB; `win-client` 4 GB / 2 vCPU / 60 GB. These values differ from the Week 1 brief's target table. Confirm the real VM settings and either change the VMs to match the brief or document the deviation with the instructor; do not silently represent them as compliant.

## Persistence test (W4) — pending

Do not mark this requirement passed until the actual lab has been tested.

1. Record the status of both agents and identify an existing alert.
2. Run `docker compose stop`, then `docker compose start`; wait for the stack to become healthy.
3. Confirm all containers are running, both agents return to Active, and the old alert remains searchable.
4. Reboot `siem-host` and repeat the checks.
5. Add actual results and screenshots to [journal.md](journal.md) and [evidence/](evidence/).

## Windows evaluation expiry

The previous note says “89 days” but does not specify the start date. Check the actual Windows evaluation expiry on `win-client` and record the exact date. Do not treat a remaining-day count as a calendar expiry date.
