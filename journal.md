# Journal: Week 1

This journal records troubleshooting from the lab build. Where the original notes did not include a date, no date has been invented.

## Date not recorded — Invalid YAML in netplan
**What I tried:** Set a static IP on the siem-host host-only adapter by editing `/etc/netplan/50-cloud-init.yaml`.
**What failed:** `netplan apply` returned `Invalid YAML: inconsistent indentation`.
**What fixed it:** Aligned the `ens33` and `ens37` keys and tested with `netplan try`. VMware used `ens33`/`ens37`, not `enp0s3`.

## Date not recorded — Dell could not open the dashboard
**What I tried:** Opened `https://192.168.56.10` from the Dell.
**What failed:** The Dell had two host-only adapters on the same subnet, and VMware VMnet4 had a 169.254.x.x address.
**What fixed it:** Disabled the conflicting VirtualBox host-only adapter and assigned VMnet4 `192.168.56.1/24` with no gateway. Ping and dashboard access then worked.

## Date not recorded — Docker permission denied
**What I tried:** Ran the certificate-generator Compose command.
**What failed:** Permission was denied for `/var/run/docker.sock`.
**What fixed it:** Added my account to the `docker` group and logged in again. Note: membership in this group grants root-equivalent privileges.

## Date not recorded — Wrong agent package architecture
**What I tried:** Installed the Linux agent using the command from the dashboard.
**What failed:** The package was ARM64 but the system was AMD64.
**What fixed it:** Selected the AMD64 package matching Wazuh version 4.14.7.

## 2026-10-14 — Duplicate agent name
**What I tried:** Restarted the stack while changing the admin password.
**What failed:** `ubuntu-web` disconnected and `ossec.log` reported a duplicate agent name.
**What fixed it:** Removed the stale registration, removed the old agent key on the endpoint, and enrolled it again.

## 2026-10-14 — ubuntu-web lost its host-only IP
**What I tried:** Connected from the Dell to `192.168.56.20`.
**What failed:** SSH timed out; `ens33` had no address while `ens37` had a NAT DHCP address.
**What fixed it:** Created a NetworkManager profile with a static address and brought it up.

## 2026-10-14 — SSH and sudo events did not reach Wazuh
**What I tried:** Ran `sudo ls /root` and generated failed SSH logins.
**What failed:** The agent was Active, but the auth log was not being collected.
**What fixed it:** Added a `<localfile>` entry for `/var/log/auth.log` in the agent configuration and restarted the agent using systemd.

## 2026-10-14 — Agent stopped after a manual restart
**What I tried:** Restarted the agent with `wazuh-control restart`.
**What failed:** The service became inactive and the dashboard showed it disconnected.
**What fixed it:** Started the service with `sudo systemctl start wazuh-agent`.

## 2026-10-14 — Alerts exist in alerts.json but dashboard is empty
**Status: OPEN — the fix has not yet been verified.**

**Observed:** A sudo alert (rule 5402, agent 004) existed in `/var/ossec/logs/alerts/alerts.json`, but `filebeat test output` returned `401 Unauthorized`.

**Next steps:**
1. Follow the official Wazuh 4.14 Docker password-change procedure and identify the correct Filebeat-to-indexer credential setting for this deployment. Do not assume the dashboard administrator password is also the internal service password.
2. Back up configuration securely; do not commit private values.
3. Apply the documented update and restart/recreate only the required service, preserving volumes. Never use `docker compose down -v`.
4. Re-run `filebeat test output` and confirm success.
5. Confirm a recent alert appears in the dashboard, save redacted evidence, and replace this note with the commands actually used and observed result.

## Persistence test (W4)
**Status: NOT YET VERIFIED. Do not claim this requirement is complete until tested.**

1. Record current agent status and identify an existing alert in the dashboard.
2. From the Compose directory, run `docker compose stop`, then `docker compose start`; wait for services to become healthy.
3. Confirm all three containers are running, both agents return to Active, and the existing alert is still searchable.
4. Reboot `siem-host` and repeat those checks.
5. Save command output/screenshots under `evidence/` and record the actual results here.

Do not mark W4 passed unless both the stack restart and host reboot checks pass.
