# Troubleshooting record

[Project overview](../README.md) · [Completed setup](setup.md)

Only issues encountered in the build are included. Diagnoses from the build discussion are distinguished from confirmed outcomes; missing logs and generic service errors are not treated as proof of a particular root cause.

## Windows agent installation and service startup failed

- **Issue:** The first installation/start attempts did not produce a running agent. Later, `WazuhSvc` existed but remained stopped; reading the expected `ossec.log` path also failed.
- **Cause:** I identified a relative installer path that differed from the download location. The remaining startup failure's exact cause was not proven. A missing log did not establish that the executable or entire installation directory was missing.
- **Resolution:** Correct the installer path to the actual downloaded file, inspect the service registration, then complete the GUI configuration and manual enrollment described below. I confirmed the final service status as running. A clean reinstall was considered but not executed as a completee fix.
- **Lesson:** Separate installer-file location, service existence, service startup, and enrollment when diagnosing failure. A generic service message is not a root-cause finding.

## Windows GUI requested an authentication key

- **Issue:** The GUI required an Authentication Key after the manager address was entered, and I did not know where to obtain it.
- **Cause:** Manual GUI enrollment needs the manager-generated client key; the dashboard password is not that key. The record does not establish why the earlier automatic workflow did not finish successfully.
- **Resolution:** On the manager, use `sudo /var/ossec/bin/manage_agents` to add the Windows agent and extract its client key. Enter the complete key and manager `192.168.56.101` in the Windows Agent GUI, save/apply, and start the service. The owner reported **Running** afterward and later confirmed dashboard visibility.
- **Lesson:** Enrollment keys are generated in the manager VM only. Treat enrollment keys as secrets. Never place a real key in a screenshot, command example, or repository.

## Ubuntu agent download returned HTTP 403

- **Issue:** `wget` received **403 Forbidden** from the Wazuh package endpoint, so no usable DEB was available for installation.
- **Cause:** That download request was refused. The exact reason for refusal is unknown; there is no evidence that Ubuntu needed a different CPU architecture or that the manager was at fault.
- **Resolution:** Download the same `wazuh-agent_4.14.7-1_amd64.deb` using `curl -fLO`, check the downloaded file, install it with `WAZUH_MANAGER="192.168.56.101"`, then enable/start the service. I confirmed active/running and subsequent dashboard visibility. The full successful commands are in [setup.md](setup.md#3-ubuntu-endpoint).
- **Lesson:** Distinguish package retrieval from installation and enrollment. The `curl` workaround succeeded in this build; it is not proof of a universal `wget` problem or a guaranteed fix for every 403.
