# Home SOC Lab

A self-directed home lab for practicing SOC analyst and detection-engineering skills end-to-end: build a monitored Active Directory + SIEM environment, simulate attacks, write and tune detections, and document the whole process.

## Overview

Built in VirtualBox on an isolated host-only + NAT network. The lab centers on a Wazuh manager (with Splunk also running on the same host) collecting logs from a domain controller and a Windows workstation, with a Kali box used to generate real attack traffic to validate detections against.

Documentation is generated session-by-session with LabScribe — a Claude-powered desktop tool built specifically for this project — from real captured terminal transcripts, screenshots, and notes. `README.md` (this file) is a hand-maintained overview of the whole project; the full per-session write-ups (build steps, troubleshooting logs, exact commands and errors) live in [`docs/`](./docs), and [`CHANGELOG.md`](./CHANGELOG.md) is the append-only session log.

## Environment

| Host | OS | Role | IP |
|---|---|---|---|
| DC01 | Windows Server 2022 | Domain Controller + DNS | `10.10.10.10` |
| WKS01 | Windows 11 | Domain-joined workstation, Splunk Universal Forwarder + Sysmon | `10.10.10.20` |
| SIEM01 (`soc-lab-ubuntu`) | Ubuntu | Wazuh manager + Splunk | `10.10.10.30` |
| KALI01 | Kali Linux | Attacker box | `10.10.10.40` |

Note: several sessions' captured transcripts show activity on a `192.168.192.0/24` host-only segment rather than the `10.10.10.0/24` addressing above — this reflects real network changes made over the course of the project (see individual session docs for exact IPs observed at the time).

## Timeline

| Date | Session | Summary |
|---|---|---|
| 2026-08-02 | [Ubuntu Setup — Disk, LVM & Wazuh Install](./docs/2026-08-02-ubuntu-setup-disk-lvm-wazuh-install.md) | Initial SIEM01 build: disk partitioning, LVM, and first Wazuh all-in-one install. |
| 2026-08-05 | [Splunk Installation & HTTPS Setup](./docs/2026-08-05-splunk-installation-https-setup.md) | Brought SIEM01 online as a Wazuh manager, enrolled the Windows 11 host as an agent, and began Splunk install/HTTPS setup. |
| 2026-08-05 | [Windows VM Capture Agent Setup](./docs/2026-08-05-windows-vm-capture-agent-setup.md) | Installed and configured the Splunk Universal Forwarder on WKS01 to forward Security/System/Sysmon logs to the SIEM. |
| 2026-08-06 | [Kali Capture Agent — Initial Setup](./docs/2026-08-06-kali-capture-agent-initial-setup.md) | Configured LabScribe's auto-capture agent on KALI01 so future attack sessions are recorded automatically. |
| 2026-08-06 | Simulating Attacks and Detection | *No transcript captured for this session — see [Known Gaps](#known-gaps) below.* |
| 2026-08-06 | [Simulating Attacks and Wazuh Filtering](./docs/2026-08-06-simulating-attacks-and-wazuh-filtering.md) | First SSH brute-force attack from KALI01 against SIEM01; confirmed the attempts landed in `auth.log`. |
| 2026-08-06 | [Adding Wazuh Correlation Rules for Brute Force](./docs/2026-08-06-adding-wazuh-correlation-rules-for-brute-force.md) | Began writing a custom Wazuh correlation rule for SSH brute-force detection (`local_rules.xml`). |
| 2026-08-07 | [Rule Adding and Conditions](./docs/2026-08-07-rule-adding-and-conditions.md) | SIEM01 maintenance and Guest Additions build tooling; follow-on detection-rule work carried into later sessions. |
| 2026-08-08 | [Adding Custom Rules to Wazuh](./docs/2026-08-08-adding-custom-rules-to-wazuh.md) | Finished the custom SSH brute-force detection rule and validated it against a live Hydra attack; debugged a manager-breaking XML error along the way. |
| 2026-08-09 | [Port Scan Detection](./docs/2026-08-09-port-scan-detection.md) | Built a UFW-based port scan / brute-force detection pipeline: custom decoder, rules mapped to MITRE T1046, validated with Hydra and repeated Nmap scans. |

For the full commit-by-commit history (including a couple of duplicate re-generations), see [`CHANGELOG.md`](./CHANGELOG.md).

## Detection Engineering

Custom Wazuh detections written and validated against live attack traffic from KALI01:

- **SSH brute-force detection** — correlation rule against repeated `sshd` auth failures (MITRE T1110.001), plus a follow-on rule for a successful login after a brute-force pattern.
- **Port scan detection** — custom `ufw-block` decoder plus a frequency-based rule (MITRE T1046) triggered by repeated UFW-blocked connection attempts from the same source.

Both were validated end-to-end: attack traffic generated from Kali (Hydra for brute-force, Nmap for port scans), and the resulting Wazuh alerts confirmed via `wazuh-logtest` and the dashboard.

## Known Gaps

- **Simulating Attacks and Detection (2026-08-06, 10:52–11:08)** has no documentation. The transcript file LabScribe originally associated with this session had actually stopped being written five minutes *before* the session began — a symptom of a `script` write-buffering bug (fixed later by adding `--flush` to the capture agent). Its real content belongs to the prior "Kali Capture Agent — Initial Setup" session and has been correctly attributed there instead. There is no recoverable transcript for this session.
- A "Full Report (Word)" link previously in this README pointed at a `.docx` file that doesn't exist anywhere in this repo's history — it's been removed rather than recreated. If a consolidated Word-format report (like the one in [`Vulnerability-Management-Workflow`](https://github.com/DevinCodes13/Vulnerability-Management-Workflow)) is wanted for this project too, that's a separate task.

## Tools Used

| Category | Tools |
|---|---|
| SIEM / Detection | Wazuh, Splunk, Sysmon |
| Attacker Simulation | Kali Linux, Hydra, Nmap |
| Documentation | LabScribe (Claude-powered auto-documentation from captured terminal sessions) |
| Hypervisor / Networking | VirtualBox (host-only + NAT) |
