# Splunk Threat Detection Lab

## Overview
A hands-on SOC home lab built from scratch to develop 
real world skills in threat detection, log analysis, 
and incident investigation. This lab simulates an 
enterprise SOC environment using industry standard tools.

## Lab Architecture
| Component | Role |
|---|---|
| Ubuntu Server 24.04 | Hosts Splunk Enterprise SIEM |
| Windows 11 PC | Monitored endpoint |
| Kali Linux | Attack simulation machine |
| Splunk Enterprise | SIEM platform |
| Splunk Universal Forwarder | Log collection agent |

## What Was Built
- Deployed Splunk Enterprise on Ubuntu Server
- Configured log collection from Windows and Linux endpoints
- Built brute force detection alert
- Created SOC monitoring dashboard
- Simulated real attacks using Kali Linux and Hydra
- Investigated full attack chain end to end in Splunk

## Tools Used
- Splunk Enterprise
- Ubuntu Server 24.04
- Kali Linux
- Nmap
- Hydra
- Wireshark
- UFW Firewall

## Documentation
- [1. Infrastructure Setup](#)
- [2. Splunk SIEM Deployment](#)
- [3. Log Collection Configuration](#)
- [4. Detection Engineering](#)
- [5. Attack Simulation](#)
- [6. Incident Investigation](#)

## Skills Demonstrated
- SIEM deployment and configuration
- Log analysis and threat detection
- Incident investigation and documentation
- Linux server administration
- Network security monitoring
- Attack simulation and detection
