# My SOC Home Lab

I’ve been working on building my own SOC  lab to get hands-on experience with security tools and threat detection. This repository is where I’m keeping all my notes and documentation on how I set everything up, from the firewall to the SIEM.

## What's in the Lab
I wanted a setup that gave me visibility across the network and endpoints. Here is the core tech stack I’m currently running in my virtual environment:

- **Wazuh Server:** A dedicated virtual machine serving as the brain of the whole setup (SIEM/XDR).
- **pfSense:** A dedicated virtual firewall to manage and filter traffic.
- **Windows Endpoint:** A virtual machine acting as the monitored target for log collection, Sysmon telemetry, and FIM testing.
- **Suricata:** Integrated directly into the pfSense firewall to monitor network traffic.

## Project Progress

[✓] [Wazuh Setup](docs/Wazuh-Setup.pdf)

[✓] [pfSense Integration](docs/pfSense%20Integration.pdf)

[✓] [Network Segmentation](docs/Network-Segmentation.md)

[✓] [Suricata Integration](docs/Suricata%20Integration.pdf)

[✓] [Log Collection](docs/Configuring%20log%20collection.pdf)

[✓] [File Integrity Monitoring](docs/File%20integrity%20monitoring.pdf)

[ ] Integrate SOAR platform

[ ] Deploy Honeypot

---
*Built and documented by BELASRI Ayman.*
