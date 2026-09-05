# Architecture

[Project overview](../README.md) · [Completed setup](../docs/setup.md)

**Work in progress:** infrastructure and endpoint enrollment are complete. Detailed telemetry validation and detection coverage remain future milestones.

![Virtual home SOC network diagram](network-diagram.png)

## VM and host design

A Windows desktop runs Oracle VirtualBox and all three VMs. Keeping the initial environment on one host makes the lab easier to operate and reason about. It also creates a single point of failure and shared resource contention; this is a learning deployment with no high-availability claim.

| System | Function | Installed components | Host-only address |
|---|---|---|---|
| Windows desktop | Virtualization host and analyst dashboard access | Oracle VirtualBox and a browser | Not recorded; no host address is assumed here. |
| `SOC-WAZUH-01` | Central monitoring VM | Wazuh manager, indexer, dashboard | `192.168.56.101` |
| `WIN11-LAB` | Windows endpoint | Windows 11 Pro, Wazuh Agent, Sysmon | `192.168.56.102` |
| `UBUNTU-LAB` | Linux endpoint | Ubuntu Server, Wazuh Agent, OpenSSH | `192.168.56.103` |

VM names are the lab labels used in the build. Final vCPU, RAM, disk allocations, and exact software versions are manually configured in accordance to performance on my PC.

The all-in-one Wazuh VM places central processing, indexing, and the analyst interface together. Windows and Ubuntu provide different endpoint activity for later investigations. Sysmon supplies Windows event telemetry; OpenSSH supplies an authentication surface on Ubuntu.

## Two network paths

Each VM uses a NAT adapter and a host-only adapter. The build used Adapter 1 for NAT and Adapter 2 for host-only networking; guest interface names are intentionally not assumed.

**Host-only lab path — `192.168.56.0/24`.** All three VMs attach to the same VirtualBox host-only network: the manager at `192.168.56.101`, Windows at `192.168.56.102`, and Ubuntu at `192.168.56.103`. This path supports VM-to-VM communication, endpoint-to-manager communication, and dashboard access from the desktop. Both endpoint agents point to `192.168.56.101`, the manager's host-only address.

**NAT path — outbound access.** Separate NAT adapters allow VM downloads and updates using the host's network connectivity. NAT is not placed between the endpoints and Wazuh, and the manager is not an internet gateway for the endpoints. The diagram groups the outbound paths for readability; it does not imply a shared VirtualBox “NAT Network” service.

Host-only networking allows host/guest communication without directly attaching that virtual segment to the physical LAN. NAT provides a separate outbound route and is not an egress allowlist. These mode semantics are described in [Oracle's Virtual Networking documentation](https://docs.oracle.com/en/virtualization/virtualbox/7.2/user/networkingdetails.html).

## Data flow and what is verified

1. Windows and Ubuntu agents have enrolled with the manager; the owner reports both visible in the Wazuh dashboard.
2. The intended Windows path is activity → Sysmon/Windows Event Log → Wazuh Agent → central Wazuh analysis and indexing → dashboard.
3. The intended Ubuntu path is SSH activity → local authentication records → Wazuh Agent → central Wazuh analysis and indexing → dashboard.
4. Source-specific collection and alert behavior need explicit end-to-end validation. The diagram's solid network links describe connectivity, not verified detection coverage.

Windows enrollment used a manager-generated client key imported into the agent GUI. Ubuntu installation supplied the manager address and was followed by starting the agent service. Enrollment material is deliberately excluded from this repository.

## Boundaries and limitations

- The Windows desktop participates in the host-only segment; it is not isolated from the lab VMs.
- With NAT enabled, the VMs can make outbound connections through the host. The environment is not air-gapped, and this design is not presented as malware containment.
- Bridged networking and inbound port forwarding are not part of the documented design. Their absence has not been independently audited on the running host.
- No dedicated firewall/router VM, network sensor, Active Directory domain, or automated response workflow is included in this baseline.
- Dashboard access and connected agents establish infrastructure readiness, not completeness of logging, detection accuracy, or production security.

Future exercises should remain scoped to these owned lab addresses and benign test activity.
