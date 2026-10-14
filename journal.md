# Journal: Week 1

## 2026-10-?? Invalid YAML in netplan

**What I tried:** Gave siem-host a static IP on the new host-only adapter by editing `/etc/netplan/50-cloud-init.yaml`.
**What failed:** `netplan apply` returned `Invalid YAML: inconsistent indentation`.
**What fixed it:** `ens33` and `ens37` were indented differently. YAML needs every key at the same level to start in the same column, and tabs are not allowed. I aligned them and tested with `netplan try`. I also learned the adapter names are `ens33`/`ens37` on VMware, not `enp0s3`.

## 2026-10-?? The Dell could not open the dashboard

**What I tried:** Opened `https://192.168.56.10` from the Dell. It worked inside win-client but not on the Dell.
**What failed:** The Dell had two host-only adapters on 192.168.56.x (VirtualBox and VMware), and the VMware VMnet4 adapter only had a 169.254.x.x address.
**What fixed it:** I disabled the VirtualBox host-only adapter and gave the VMnet4 adapter 192.168.56.1/24 with no gateway. Then ping and the dashboard worked from the Dell.

## 2026-10-?? Docker permission denied

**What I tried:** Ran `docker compose -f generate-indexer-certs.yml run --rm generator`.
**What failed:** `permission denied while trying to connect to the docker API at unix:///var/run/docker.sock`.
**What fixed it:** My user was not in the `docker` group. `sudo usermod -aG docker $USER` and a new login fixed it.

## 2026-10-?? Wrong agent package architecture

**What I tried:** Installed the Linux agent with the command from the dashboard.
**What failed:** `package architecture (arm64) does not match system (amd64)`.
**What fixed it:** I had picked ARM64 in the Deploy new agent form. I downloaded the amd64 package of the same version (4.14.7) and installed that.

## 2026-10-14 Duplicate agent name after restarting the stack

**What I tried:** Restarted the stack while changing the admin password.
**What failed:** ubuntu-web showed disconnected. `ossec.log` said `Duplicate agent name: ubuntu-web` and `Unable to connect to any server`.
**What fixed it:** I removed the old registration on the manager with `manage_agents -r <id>`, deleted `/var/ossec/etc/client.keys` on the agent and started it again so it enrolled with a new key.

## 2026-10-14 ubuntu-web lost its host-only IP

**What I tried:** SSH from the Dell to ubuntu-web (192.168.56.20).
**What failed:** Connection timed out. `ip a` showed `ens33` had no address and `ens37` (NAT) had a DHCP address. Ubuntu Desktop uses NetworkManager, so a netplan file did not help.
**What fixed it:** I created a NetworkManager profile with a static address using `nmcli con add ... ipv4.method manual ipv4.addresses 192.168.56.20/24` and brought it up.

## 2026-10-14 SSH and sudo events never reached the dashboard

**What I tried:** Ran `sudo ls /root` and failed SSH logins and looked for them in Wazuh.
**What failed:** The agent was Active but nothing arrived. `ossec.log` showed the logcollector analysing `dpkg.log` and the journal, but not `/var/log/auth.log`.
**What fixed it:** I added a `<localfile>` block (`log_format` syslog, location `/var/log/auth.log`) to the agent's `ossec.conf` and restarted the agent. The log then showed `Analyzing file: '/var/log/auth.log'`.

## 2026-10-14 Agent stopped after I restarted it by hand

**What I tried:** Restarted the agent with `wazuh-control restart` after editing `ossec.conf`.
**What failed:** The agent went `inactive (dead)` and the dashboard showed disconnected. systemd did not know about the restart.
**What fixed it:** Starting it with `sudo systemctl start wazuh-agent`. Now I always use `systemctl` for the agent service.

## 2026-10-14 Alerts in alerts.json but empty dashboard

**What I tried:** Checked each step of the path. The sudo alert (rule 5402, agent 004) was in `/var/ossec/logs/alerts/alerts.json` on the manager.
**What failed:** `filebeat test output` returned `401 Unauthorized`. The manager still used the old indexer password after I changed it.
**What fixed it:** TODO (fill in after the fix: update `INDEXER_PASSWORD` for the manager in `docker-compose.yml`, recreate the manager container, check `filebeat test output` again).

## Persistence test (W4)

TODO: write what I did (`docker compose stop` / `start`, reboot of siem-host) and what I observed (agents still enrolled? old alerts still visible?).
