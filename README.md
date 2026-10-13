# Wazuh-SIEM-Lab
A practical cybersecurity engineering homelab simulating an enterprise Local Area Network (LAN) monitored by a centralized **Wazuh SIEM** platform. This project demonstrates full-cycle defensive operations: virtual infrastructure provisioning, network bridging, multi-platform agent enrollment, real-time log analysis, and automated threat mitigation via custom **Active Response** rules.

*Author: Facundo Maidana*

## Architecture and Topology

The lab simulates an enterprise network where all endpoints and the centralized SIEM manager reside on the same LAN subnet via physical **Ethernet Bridging**, allowing unrestricted bi-directional communication, telemetry ingestion, and active containment.

## Technologies and Environment

| Component          | Technology                              | Role                                                                |
| ------------------ | --------------------------------------- | ------------------------------------------------------------------- |
| **SIEM Platform**  | Wazuh (Manager, Indexer, Dashboard)     | Centralized telemetry, log aggregation, correlation and visualization |
| **Virtualization** | Oracle VM VirtualBox and QEMU/KVM       | Type-2 Hypervisor managing virtual machines and virtual networks    |
| **SIEM Host OS**   | Ubuntu Server LTS                       | Dedicated Linux server hosting the core SIEM stack                  |
| **Endpoints**      | Windows 11, Ubuntu Desktop, Host Laptop | Multi-platform endpoints running lightweight Wazuh agents           |
| **Networking**     | Bridged Adapter (Gigabit Ethernet)      | Provides unique physical LAN IP leases for all nodes                |
| **Threat Defense** | Wazuh Active Response (firewall-drop)   | Automated IP blocking and incident remediation                      |
