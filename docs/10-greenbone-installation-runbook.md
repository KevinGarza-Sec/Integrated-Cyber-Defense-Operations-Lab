# 10 — Greenbone Installation Runbook

## Preconditions

Validated before install:

- Greenbone/OpenVAS VM is on VMnet2 / OPNsense LAN.
- Scanner IP: `10.10.10.100/24`.
- Default gateway: `10.10.10.254`.
- DNS server: `10.10.10.254`.
- Scanner can ping OPNsense `10.10.10.254`.
- Scanner can ping target `ubuntutarget` / `10.10.10.167`.
- DNS resolution works.
- Rollback snapshot taken: `16-greenbone-scanner-network-validation`.

## Official Install Source

Use the Greenbone Community Containers documentation:

```text
https://greenbone.github.io/docs/latest/22.4/container/index.html
```

The official compose file URL discovered from the docs is:

```text
https://greenbone.github.io/docs/latest/_static/compose.yaml
```

## Install Docker and Dependencies

On the Greenbone VM:

```bash
sudo apt update
sudo apt install -y ca-certificates curl docker.io docker-compose-v2
sudo systemctl enable --now docker
sudo usermod -aG docker "$USER"
newgrp docker
```

If `docker-compose-v2` is unavailable on this Ubuntu version, stop and use the Docker Engine official repository method from Docker's docs, then resume from the compose download step.

Verify Docker:

```bash
docker --version
docker compose version
```

## Download Greenbone Community Compose File

```bash
export DOWNLOAD_DIR="$HOME/greenbone-community-edition"
mkdir -p "$DOWNLOAD_DIR"
curl -f -L https://greenbone.github.io/docs/latest/_static/compose.yaml -o "$DOWNLOAD_DIR/compose.yaml"
```

Review the file exists:

```bash
ls -lh "$DOWNLOAD_DIR/compose.yaml"
```

## Pull and Start Containers

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" pull
docker compose -f "$DOWNLOAD_DIR/compose.yaml" up -d
```

Check containers:

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" ps
```

Watch logs while feeds initialize:

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" logs -f
```

Stop watching logs with `Ctrl+C`; this does not stop the containers.

## Change Default Admin Password

Greenbone documentation notes that a default `admin/admin` credential may exist and should be changed.

Do not store the new password in the repo.

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" exec -u gvmd gvmd gvmd --user=admin --new-password='<REDACTED_PASSWORD>'
```

Use a real password locally, but never screenshot or commit it.

## Access Web Interface

From the Greenbone VM itself:

```text
https://127.0.0.1
```

From the Windows host or another lab VM, access may require checking which address/port the container exposes:

```bash
sudo ss -ltnp | grep -E ':443|:9392'
docker compose -f "$DOWNLOAD_DIR/compose.yaml" ps
```

If the web UI is bound only to localhost, use the Greenbone VM browser/console first or adjust exposure only after understanding the binding. Do not expose Greenbone to the public internet.

## Portfolio Evidence

Next screenshots:

| Filename | Capture | Redact |
|---|---|---|
| `17-greenbone-platform-login-page.png` | Login page reached through the localhost-only SSH tunnel | Never capture credentials |
| `18-greenbone-platform-dashboard.png` | Authenticated dashboard with populated CVE/NVT data | Logged-in account name (`admin`) |
| `19-greenbone-target-scope.png` | Target/scope configured only for `10.10.10.167` | Credentials, scanner IDs if visible |
| `22b-greenbone-updated-results-14-summary.png` | First findings summary | Public IPs, account IDs, MACs |
| `23-remediation-tracker.png` | Tracker populated from findings | Real names/tickets |
| `24-validation-scan-or-remediation-proof.png` | Re-scan or manual validation | Same as findings |

## Troubleshooting Notes

- Feed sync can take a long time and can make the VM feel slow with 4 GB RAM.
- If the VM becomes sluggish, consider increasing RAM to 6–8 GB.
- If Docker install fails because the Ubuntu release is too new for a package/repository, capture the error and pivot to a supported Docker install method.
- Do not scan outside `10.10.10.167` until scope is explicitly updated.

### `no space left on device` during `docker compose up -d`

Observed failure pattern:

```text
service "pg-gvm-migrator" didn't complete successfully: exit 1
Error response from daemon: failed to create task for container: mkdir /var/lib/containerd/...: no space left on device
```

Meaning:

- Docker/containerd ran out of writable storage under `/var/lib/docker` or `/var/lib/containerd`.
- This can happen even with a 40 GB virtual disk if Ubuntu allocated only part of the disk to `/`, or if Greenbone images/volumes consumed the available root filesystem.

Diagnosis commands to run before making changes:

```bash
df -h
lsblk
sudo du -xh -d1 /var/lib/docker /var/lib/containerd 2>/dev/null || true
docker system df
```

If `lsblk` shows free space in the Ubuntu volume group but `/` is smaller than the disk, expand root:

```bash
sudo lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
df -h
```

If `/` is truly full and no unallocated space remains, increase the VMware disk to 60 GB, then expand the partition/LVM/root filesystem before retrying. Avoid deleting Greenbone volumes until the disk layout is understood, because failed migrations can leave partial state.

KG lab observed diagnostics:

```text
/dev/mapper/ubuntu--vg-ubuntu--lv  19G  19G  0  100% /
sda                                   40G disk
sda3                                  38G part
ubuntu--vg-ubuntu--lv                 19G lvm /
Docker images                         11.32GB
Docker local volumes                   712MB
/var/lib/containerd                   about 11G
```

Interpretation: the VM disk is 40 GB and the LVM partition is 38 GB, but the root logical volume is only 19 GB. Expand the existing root LV into the free VG space before increasing the VMware disk.

Validated next fix for this state:

```bash
sudo lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
df -h
lsblk
```

After confirming `/` is larger, retry:

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" up -d
docker compose -f "$DOWNLOAD_DIR/compose.yaml" ps
```

Follow-up after root expansion:

```text
/dev/mapper/ubuntu--vg-ubuntu--lv  38G  19G  17G  53% /
```

Root filesystem expansion succeeded, but several containers still reported earlier disk-full state:

```text
pg-gvm-migrator: FATAL could not write lock file "postmaster.pid": No space left on device
gsa: cp: write error: No space left on device
```

Interpretation: these logs reflect containers/volumes initialized during the previous disk-full failure. Restart the Compose stack after the filesystem has free space, then re-check health.

Safe retry:

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" down
docker compose -f "$DOWNLOAD_DIR/compose.yaml" up -d
docker compose -f "$DOWNLOAD_DIR/compose.yaml" ps
```

If the same errors persist after a clean restart with free disk available, inspect logs again. If logs still reference files damaged during the disk-full run, the next likely recovery is to remove and recreate the Greenbone stack volumes from the pre-install snapshot or with `docker compose down -v`; do that only after confirming no useful scan data exists.

### Main services remain `Created` after migrations succeed

Observed after the disk fix and clean restart:

```text
cert-bund-data, data-objects, gsa, notus-data, redis-server, scap-data: Up / healthy
configure-openvas, gvm-config, gpg-data, pg-gvm-migrator: Exited (0)
nginx, gsad, gvmd, gvm-tools, ospd-openvas, openvas, openvasd, pg-gvm: Created
```

Interpretation: the earlier `docker compose up -d` run stopped after one-shot dependency/migration phases. Once `pg-gvm-migrator` and config containers exit successfully, rerun Compose to start the dependent long-running services.

Safe retry:

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" up -d
docker compose -f "$DOWNLOAD_DIR/compose.yaml" ps --format "table {{.Name}}\t{{.Service}}\t{{.Status}}\t{{.Ports}}"
sudo ss -ltnp | grep -E ':443|:9392|:80'
```

If main services remain `Created`, inspect dependency logs:

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" logs --tail=120 pg-gvm
docker compose -f "$DOWNLOAD_DIR/compose.yaml" logs --tail=120 gvmd
docker compose -f "$DOWNLOAD_DIR/compose.yaml" logs --tail=120 gsad
docker compose -f "$DOWNLOAD_DIR/compose.yaml" logs --tail=120 nginx
```

### Docker network endpoint already exists

Observed after re-running `up -d`:

```text
failed to set up container networking: endpoint with name greenbone-community-edition-vulnerability-tests-1 already exists in network greenbone-community-edition_default
```

Meaning: Docker has stale container/network endpoint state for the `vulnerability-tests` container from previous failed starts. The web ports still were not listening, and only the data/helper containers were healthy.

Conservative cleanup for just the stale container/network state:

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" stop vulnerability-tests
docker compose -f "$DOWNLOAD_DIR/compose.yaml" rm -f vulnerability-tests
docker network disconnect -f greenbone-community-edition_default greenbone-community-edition-vulnerability-tests-1 2>/dev/null || true
docker compose -f "$DOWNLOAD_DIR/compose.yaml" up -d
docker compose -f "$DOWNLOAD_DIR/compose.yaml" ps --format "table {{.Name}}\t{{.Service}}\t{{.Status}}\t{{.Ports}}"
sudo ss -ltnp | grep -E ':443|:9392|:80'
```

If multiple stale endpoints appear, use a broader but still non-volume-destroying reset:

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" down
docker network prune
docker compose -f "$DOWNLOAD_DIR/compose.yaml" up -d
```

Do not use `down -v` unless the stack remains unrecoverable and it is acceptable to recreate Greenbone volumes/feed state from scratch.

Final startup success after stale endpoint cleanup:

```text
docker compose up -d: Running 21/21
gvmd: Up / healthy
pg-gvm: Up / healthy
gsa: Up / healthy
nginx: Up
gsad: Up
ospd-openvas: Up
openvas: Up
openvasd: Up
vulnerability-tests: Up / healthy
```

Web listeners observed:

```text
127.0.0.1:443
127.0.0.1:9392
```

Interpretation: Greenbone is running successfully, but the web UI is bound to localhost on the Greenbone VM. Access it from the Greenbone VM at `https://127.0.0.1`. If remote access from the Windows host is needed, use a controlled SSH tunnel or adjust binding only after documenting the security impact; do not expose Greenbone publicly.

### Access GSA from a server VM with no browser

If the Greenbone VM is Ubuntu Server and has no GUI/browser, use an SSH local port forward from the Windows host or another trusted lab workstation instead of exposing Greenbone broadly.

On the Greenbone VM, verify or install SSH:

```bash
sudo systemctl status ssh --no-pager
```

If SSH is not installed/running:

```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
sudo ss -ltnp | grep ':22'
```

From Windows PowerShell on the host PC, keep this SSH session open:

```powershell
ssh -L 8443:127.0.0.1:443 kg@10.10.10.100
```

Then open this on the Windows host browser:

```text
https://127.0.0.1:8443
```

If port 443 does not present the login page, tunnel the alternate GSA listener:

```powershell
ssh -L 9392:127.0.0.1:9392 kg@10.10.10.100
```

Then browse:

```text
https://127.0.0.1:9392
```

Security note: this exposes Greenbone only through the authenticated SSH session to the local browser, not to the whole lab LAN or public internet.

If the browser cannot reach the tunnel, diagnose in layers:

If a new SSH session reports `open /compose.yaml: no such file or directory`, the `DOWNLOAD_DIR` shell variable is not set in that session. Re-export it or use the absolute path before running Compose commands:

```bash
export DOWNLOAD_DIR="$HOME/greenbone-community-edition"
test -f "$DOWNLOAD_DIR/compose.yaml" && echo "compose file found"
```

Equivalent absolute path form:

```bash
docker compose -f "$HOME/greenbone-community-edition/compose.yaml" ps
```

On the Greenbone VM/SSH session, verify the service responds locally:

```bash
curl -kI https://127.0.0.1:443
curl -kI https://127.0.0.1:9392
curl -I http://127.0.0.1:9392
```

On Windows PowerShell, verify the local tunnel listener exists:

```powershell
netstat -ano | findstr ":8443"
netstat -ano | findstr ":9392"
Test-NetConnection 127.0.0.1 -Port 8443
Test-NetConnection 127.0.0.1 -Port 9392
```

If no local listener appears, restart the tunnel with explicit local IPv4 binding:

```powershell
ssh -N -v -L 127.0.0.1:8443:127.0.0.1:443 kg@10.10.10.100
```

Then browse to `https://127.0.0.1:8443`. If the verbose SSH output reports `channel ... open failed` or `bind: Address already in use`, capture that exact message.

### GVMD feed/database import can exhaust a 40 GB disk

Observed during first-run feed/SCAP import:

```text
sql_copy_end: PQexec failed: ERROR: could not extend file ... No space left on device
HINT: Check free disk space.
db_copy_buffer_commit: failed to commit database copy buffer
Received Aborted signal
```

Meaning: even after the initial LVM expansion, Greenbone's feed/database import can consume nearly all of a 40 GB VM disk. If `df -h` shows `/` near full again, the proper fix is to grow the Greenbone VM disk, not repeatedly restart containers.

Read-only checks:

```bash
df -h
df -ih
lsblk
docker system df
sudo du -xh -d1 /var/lib/docker /var/lib/containerd /home 2>/dev/null | sort -h
```

Recommended lab sizing adjustment: expand the Greenbone VM disk to at least 80 GB, then grow the Linux partition/PV/LV/filesystem:

```bash
sudo apt update
sudo apt install -y cloud-guest-utils
lsblk
sudo growpart /dev/sda 3
sudo pvresize /dev/sda3
sudo lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
df -h
lsblk
```

After space is available, restart the stack and re-check web access:

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" restart gvmd gsad nginx
sleep 30
docker compose -f "$DOWNLOAD_DIR/compose.yaml" ps
curl -vk https://127.0.0.1/
```

Validated post-expansion web response:

```text
nginx ports: 127.0.0.1:443->443/tcp and 127.0.0.1:9392->9392/tcp
curl -vk https://127.0.0.1/ returned HTTP headers and OPENVAS HTML
Connection #0 to host 127.0.0.1:443 left intact
```

Interpretation: the Greenbone web UI is responding locally on the Greenbone VM. If the browser still fails, troubleshoot the Windows SSH tunnel/client side rather than Greenbone service startup.

### After pausing/resuming VMware VMs

If Chrome shows `127.0.0.1 refused to connect` for `https://127.0.0.1:8443` after pausing the lab VMs, the Windows-side SSH tunnel is usually no longer listening. Bring the stack back in layers:

1. Resume/power on OPNsense first, then the Greenbone VM.
2. On the Greenbone VM, verify the IP and services:

```bash
ip -brief addr
export DOWNLOAD_DIR="$HOME/greenbone-community-edition"
docker compose -f "$DOWNLOAD_DIR/compose.yaml" ps
sudo ss -ltnp | grep -E ':443|:9392'
curl -vk https://127.0.0.1/
```

3. If containers are not up, start them:

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" up -d
```

4. From Windows PowerShell, reopen the tunnel and keep the window open:

```powershell
ssh -N -L 127.0.0.1:8443:127.0.0.1:443 kg@10.10.10.100
```

5. In Windows browser, use exactly:

```text
https://127.0.0.1:8443
```

If the Greenbone VM IP changed from `10.10.10.100`, replace the SSH command's destination with the current IP shown by `ip -brief addr`.

If the Windows tunnel is listening but the Greenbone VM itself returns `curl: (35) Recv failure: Connection reset by peer` for `curl -vk https://127.0.0.1/`, the browser/tunnel is not the primary fault. Diagnose/restart the Greenbone web layer:

```bash
export DOWNLOAD_DIR="$HOME/greenbone-community-edition"
docker compose -f "$DOWNLOAD_DIR/compose.yaml" ps --format "table {{.Name}}\t{{.Service}}\t{{.Status}}\t{{.Ports}}"
docker compose -f "$DOWNLOAD_DIR/compose.yaml" logs --tail=120 nginx
docker compose -f "$DOWNLOAD_DIR/compose.yaml" logs --tail=120 gsad
docker compose -f "$DOWNLOAD_DIR/compose.yaml" logs --tail=120 gvmd
```

Then try a non-destructive web/backend restart:

```bash
docker compose -f "$DOWNLOAD_DIR/compose.yaml" restart gvmd gsad nginx
sleep 60
docker compose -f "$DOWNLOAD_DIR/compose.yaml" ps
curl -vk https://127.0.0.1/
```

Validated lab expansion result:

```text
sda expanded to 80G
sda3 expanded to 78G
ubuntu--vg-ubuntu--lv expanded from 38.00 GiB to 78.00 GiB
/ after resize: 77G total, 34G used, 40G available, 46% used
```

If `docker compose down -v` returns `no configuration file provided: not found`, the command was run without `-f "$DOWNLOAD_DIR/compose.yaml"` or outside the compose directory. Re-export the variable first and always include the compose file path:

```bash
export DOWNLOAD_DIR="$HOME/greenbone-community-edition"
docker compose -f "$DOWNLOAD_DIR/compose.yaml" ps
```
