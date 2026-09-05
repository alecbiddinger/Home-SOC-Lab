# Completed infrastructure and enrollment

[Project overview](../README.md) · [Architecture](../architecture/architecture.md) · [Troubleshooting](troubleshooting.md)

**Scope:** a retrospective record of the completed infrastructure milestone. Commands below preserve the successful enrollment workflow from this build; they were not executed against the VMs while preparing this repository. This is not a full installation automation guide. Detection exercises and unverified configuration changes are excluded.

## 1. Host, VMs, and networking

The Windows desktop was set up with Oracle VirtualBox and three VMs. Each uses NAT for outbound access and the common host-only segment `192.168.56.0/24` for lab communication. The build used NAT on Adapter 1 and host-only on Adapter 2.

| VM | Host-only IP address |
|---|---|
| `SOC-WAZUH-01` | `192.168.56.101` |
| `WIN11-LAB` | `192.168.56.102` |
| `UBUNTU-LAB` | `192.168.56.103` |

The Wazuh all-in-one VM was brought online and its dashboard became accessible from the host. The manager's host-only address was confirmed as **`192.168.56.101`** and used during endpoint enrollment. Exact server installation commands and final resource allocations were not retained as confirmed steps, so they are not reconstructed here.

## 2. Windows endpoint

Windows 11 Pro was installed in `WIN11-LAB` at `192.168.56.102`. The initial Windows boot used the x64 Windows installation image. The build subsequently progressed through Windows setup to a working endpoint.

The Windows Wazuh Agent was installed using the dashboard's **Agents management → Summary → Deploy new agent** workflow, with `192.168.56.101` as the manager. An early installer command referenced a relative MSI filename that did not match the actual download location. The recorded corrected invocation targeted the downloaded file:

```powershell
msiexec.exe /i "$env:TEMP\wazuh-agent" /q WAZUH_MANAGER="192.168.56.101"
```

That temporary path is specific to this build; it is not a permanent installer location. The service subsequently existed but startup still failed. The successful final configuration used manual client-key enrollment.

### Successful manual enrollment

On the Wazuh manager:

```bash
sudo /var/ossec/bin/manage_agents
```

The recorded workflow was to add `WIN11-LAB` with **A**, use `any` for its address constraint, then extract its client key with **E** using the agent's assigned ID. `any` avoids binding that entry to a changing endpoint address; the client key is still required. No agent ID or real key is reproduced here.

In the Windows Wazuh Agent GUI, the manager address was set to `192.168.56.101`, the entire extracted key was entered in **Authentication Key**, and the configuration was saved/applied before starting the service. Status was then confirmed as **Running**. Both endpoint agents were later confirmed visible in the dashboard.

The service check used during the build was:

```powershell
Get-Service WazuhSvc
```

The client key is manager-generated enrollment material, not the dashboard password. The underlying utility is described in [Wazuh's manage_agents reference](https://documentation.wazuh.com/current/user-manual/reference/tools/manage-agents.html).

### Sysmon installation boundary

Sysmon was downloaded from Microsoft and is part of the reported completed Windows endpoint. The build guidance used the x64 executable with a default installation.

Sysmon writes to the `Microsoft-Windows-Sysmon/Operational` event channel. Collection of that channel must be validated separately from agent enrollment. See [Microsoft's Sysmon documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon).

## 3. Ubuntu endpoint

Ubuntu Server was installed in `UBUNTU-LAB` at `192.168.56.103` with OpenSSH enabled. The Wazuh dashboard's deployment page was used to select the **DEB amd64** package for this x86-64 Ubuntu endpoint.

The initial `wget` download returned **403 Forbidden**, preventing the package from being downloaded. Switching to `curl` succeeded. The exact successful sequence recorded in the build was:

```bash
curl -fLO https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.7-1_amd64.deb
ls -lh wazuh-agent_4.14.7-1_amd64.deb
sudo WAZUH_MANAGER="192.168.56.101" dpkg -i ./wazuh-agent_4.14.7-1_amd64.deb
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
sudo systemctl status wazuh-agent
```

I confirmed **active (running)** after these steps, then confirmed the endpoint was visible in the dashboard. The file listing checks that a download exists; it is not a package integrity or signature check. The 403's server-side cause was not established.

`4.14.7-1` is the package recorded in this build, not a claim about the latest release or a future installation recommendation. For future rebuilds, consult [Wazuh's Linux agent installation documentation](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html) and use a manager-compatible package.

## 4. Completed milestone

- Three VMs are set up and running.
- The Wazuh dashboard is accessible.
- Windows Wazuh Agent enrollment succeeded and its service reported running.
- Ubuntu Wazuh Agent installation succeeded and its service reported active/running.
- Both endpoint agents are visible in the dashboard.
- Sysmon and OpenSSH are installed on their respective endpoints.

These are self-reported outcomes from the build. Screenshots, exported event evidence, and a verified version/configuration inventory remain to be added. This record does not claim that snapshots were taken, custom Sysmon collection was configured, SSH detections were tested, or incident cases were completed. Those next steps are tracked in the [project roadmap](../README.md#roadmap-and-completion-criteria).
