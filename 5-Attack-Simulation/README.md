# 5. Attack Simulation

## Overview
Attack simulation is a critical part of validating a 
SOC environment. Building alerts and dashboards means 
nothing if they have not been tested against real attack 
activity. This section covers three simulated attacks 
performed from a Kali Linux machine against the lab 
environment — a brute force SSH attack against the 
Ubuntu Server, a port scan reconnaissance against the 
Ubuntu Server, and a brute force attack against the 
Windows PC.

Note: While the attacker perspective is briefly covered 
for context, the primary focus is on how each attack was 
detected, investigated, and analyzed from the defender's 
perspective using Splunk.

## Lab Setup

| Machine | Role | IP Address |
|---|---|---|
| Kali Linux | Attack machine | 10.0.0.147 |
| Ubuntu Server | Target / Splunk SIEM | 10.0.0.26 |
| Windows 11 PC | Monitored endpoint | 10.0.0.220 |


Both the Ubuntu Server and Windows PC were actively 
sending logs to Splunk during the simulation — meaning 
every attack action was being captured and available 
for investigation.


---

## Attack 1 - SSH Brute Force Against Ubuntu Server

### Attacker Side
A brute force attack was simulated using Hydra against 
the SSH service on the Ubuntu Server. A wordlist of 
10 common passwords was created on the Kali Linux machine and Hydra was configured 
to attempt each one in parallel.

Command used:

    hydra -l socadmin -P passwords.txt ssh://10.0.0.26 -t 4 -V

Command breakdown:
- hydra = the brute force tool
- -l socadmin = the username being targeted
- -P passwords.txt = the password wordlist
- ssh://10.0.0.26 = the target and protocol
- -t 4 = 4 parallel threads
- -V = verbose output showing each attempt

Hydra tried all 10 passwords within seconds. None 
matched the actual password so the attack was 
unsuccessful from the attacker's perspective.
