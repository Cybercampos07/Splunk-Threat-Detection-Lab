# 3. Log Collection Configuration

## Overview
This section covers the configuration of log collection from two endpoints, a Windows PC and an Ubuntu Server. The Splunk Universal Forwarder was installed on both machines to ship logs to the Splunk SIEM for analysis.

## What is a Universal Forwarder
The Splunk Universal Forwarder is a lightweight agent installed on monitored endpoints. It collects specified log files and forwards them to the Splunk indexer over port 9997. This is how enterprise Splunk deployments work, every endpoint has a forwarder sending logs back to a central Splunk server.

## Log Sources Configured

| Source | Machine | Log Type |
|---|---|---|
| WinEventLog Security | Windows PC | Logins, failures, privilege use |
| WinEventLog System | Windows PC | System events and errors |
| WinEventLog Application | Windows PC | Application events |
| /var/log/syslog | Ubuntu Server | General system activity |
| /var/log/auth.log | Ubuntu Server | SSH logins and sudo commands |
| /var/log/ufw.log | Ubuntu Server | Firewall activity |

## Part 1 - Windows Log Collection

### What are Windows Event Logs
Windows Event Logs are the built in logging system for 
Windows operating systems. Every significant action that 
occurs on a Windows machine is recorded as an event with 
a unique EventCode. These logs are stored in three main 
categories:

- Security = authentication events, privilege use, policy changes
- System = Windows system events, driver issues, service starts
- Application = software events, errors, and crashes

In a SOC environment Windows Event Logs are one of the 
most critical data sources for detecting threats.

### Step 1 - Download Universal Forwarder
Downloaded the Splunk Universal Forwarder installer from:

    https://www.splunk.com/en_us/download/universal-forwarder.html

Selected Windows 64-bit .msi installer.

### Step 2 - Install Universal Forwarder
Ran the .msi installer and configured the following 
during setup:
- Created forwarder admin credentials
- Set Receiving Indexer to 10.0.0.x on port 9997

This tells the forwarder where to send collected logs 
once collection is configured.

### Step 3 - Verify Forwarder Connection
After installation confirmed the forwarder was actively 
connected to the Splunk server:

    "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" list forward-server

Output confirmed:

    Active forwards:
        10.0.0.x:9997

The forwarder was connected but no logs were appearing 
in Splunk yet.

### Step 4 - Troubleshooting - No Logs in Splunk
After installation ran this search in Splunk to verify 
logs were coming in:

    index=main source="WinEventLog:Security"

No results were returned. The forwarder was connected 
to Splunk but nothing was being collected or sent.

The reason: The Universal Forwarder does not 
automatically know what logs to collect. It requires 
a configuration file called inputs.conf that explicitly 
tells it which log sources to monitor. Without this 
file the forwarder sits idle — connected but collecting 
nothing.

This is by design. Splunk leaves collection blank 
because every environment needs different logs. A web 
server needs different logs than a workstation. The 
administrator decides exactly what gets collected.

### Step 5 - Create inputs.conf
To fix the issue manually created the inputs.conf file 
at this location:

    C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf

Why this specific folder:
- etc\system\local = where all custom configurations go
- Files here override Splunk default settings
- This is the standard location for manual forwarder configurations

The file did not exist before this step — it was 
created from scratch.

File contents added:

    [WinEventLog://Security]
    index = main
    disabled = 0

    [WinEventLog://System]
    index = main
    disabled = 0

    [WinEventLog://Application]
    index = main
    disabled = 0

Breaking down each line:
- [WinEventLog://Security] = the specific Windows log to monitor
- index = main = which Splunk index to send logs to
- disabled = 0 = enabled (0 = on, 1 = off)

### Why Each Log Was Selected

**Security Log:**
The most important log for SOC work. Contains:
- EventCode 4624 = Successful logins
- EventCode 4625 = Failed login attempts
- EventCode 4672 = Admin privileges assigned
- EventCode 4688 = New processes created
- EventCode 4720 = New user accounts created

These events are directly used for detecting brute 
force attacks, privilege escalation, and unauthorized 
access.

**System Log:**
Contains Windows system level events including:
- Service starts and stops
- Driver failures
- System errors and warnings

Useful for detecting persistence mechanisms where 
malware installs itself as a Windows service.

**Application Log:**
Contains events from installed applications including:
- Software installations
- Application crashes
- Database events

Useful for detecting suspicious software activity 
and unauthorized installations.

### Step 6 - Common Issue During Setup
When creating inputs.conf Windows automatically saved 
it as inputs.conf.txt instead of inputs.conf because 
file extensions were hidden by default.

Fix applied:
1. Opened File Explorer
2. Enabled View > Show > File name extensions
3. Renamed inputs.conf.txt to inputs.conf
4. Restarted the forwarder

This is a good example of why understanding file 
extensions matters — incorrect file types can cause 
configurations to silently fail with no obvious error.

### Step 7 - Restart Forwarder
Applied the new configuration by restarting the forwarder:

    "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" restart

### Step 8 - Verify Logs Now Flowing
Ran the same search again in Splunk after restarting:

    index=main source="WinEventLog:Security"
    | stats count by EventCode
    | sort -count

This time results appeared confirming Windows Security 
logs were successfully flowing into Splunk.

The troubleshooting process used here mirrors real SOC 
work — identify the problem, understand the root cause, 
apply the fix, and verify it worked.

## Part 2 - Ubuntu Server Log Collection

### Step 1 - Understanding Why the Server Needs a Forwarder

After installing Splunk Enterprise on the Ubuntu Server, 
Splunk was capable of receiving logs from other machines 
but was not collecting any of its own system logs. This 
created a blind spot — the SIEM server itself was 
unmonitored.

To understand why this matters:
- The Ubuntu Server runs SSH which could be targeted by 
  brute force attacks
- Every sudo command run on the server is a security event
- The UFW firewall generates logs showing blocked connections
- Without a forwarder none of this activity would appear in Splunk

The decision was made to install the Universal Forwarder 
on the Ubuntu Server itself so that the SIEM has full 
visibility — including into its own activity. This 
eliminates a potential blind spot and mirrors how 
enterprise SOC environments are configured.

### Step 2 - Download Universal Forwarder on Ubuntu Server
Downloaded the Linux version of the Universal Forwarder 
directly to the Ubuntu Server using wget. First confirmed 
the server architecture:

    uname -m

Output confirmed x86_64 architecture. Then downloaded 
the matching forwarder:

    wget -O splunkforwarder-9.4.1-e3bdab203ac8-linux-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/9.4.1/linux/splunkforwarder-9.4.1-e3bdab203ac8-linux-amd64.deb"

### Step 3 - Install Universal Forwarder

    sudo dpkg -i splunkforwarder-9.4.1-e3bdab203ac8-linux-amd64.deb

The forwarder installs to /opt/splunkforwarder — separate 
from Splunk Enterprise which lives at /opt/splunk. Both 
can run on the same machine without conflict because they 
use different directories and different ports.

### Step 4 - Start and Accept License

    sudo /opt/splunkforwarder/bin/splunk start --accept-license

During startup:
- Accepted the license agreement
- Created forwarder admin credentials
- The forwarder prompted to change the management port 
  from 8089 to 8090 because Splunk Enterprise was already 
  using port 8089 on the same machine

This port conflict is a home lab specific issue. In 
enterprise environments the forwarder and indexer run 
on separate machines so port conflicts do not occur.

### Step 5 - Point Forwarder at Splunk Server
Configured the forwarder to send logs to Splunk Enterprise:

    sudo /opt/splunkforwarder/bin/splunk add forward-server 10.0.0.x:9997 -auth admin:password

### Step 6 - Add Ubuntu Log Files to Monitor
Told the forwarder which specific log files to collect 
and send to Splunk:

    sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog -index main -sourcetype syslog

    sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log -index main -sourcetype linux_secure

    sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/ufw.log -index main -sourcetype ufw

Why each log was selected:

- /var/log/syslog = general system activity, service 
  starts and stops, background processes
- /var/log/auth.log = every SSH login attempt, sudo 
  command, and authentication event on the server
- /var/log/ufw.log = every connection the firewall 
  blocked, including port scan attempts

Unlike Windows which uses inputs.conf, the Linux 
forwarder accepts log sources directly through the 
add monitor command. Each command immediately registers 
that log file for collection.

### Step 7 - Configure Boot Start

    sudo /opt/splunkforwarder/bin/splunk stop
    sudo /opt/splunkforwarder/bin/splunk enable boot-start -user root
    sudo /opt/splunkforwarder/bin/splunk start

### Step 8 - Verify Forwarder is Running

    sudo /opt/splunkforwarder/bin/splunk status

Expected output:
- splunkd is running
- splunk helpers are running

### Step 9 - Verify Ubuntu Logs Flowing in Splunk

    index=main sourcetype=syslog
    | table _time, host, _raw
    | sort -_time

Results showed ubuntu-server as the host confirming 
the forwarder was working correctly.

## Key Concepts Learned
- Universal Forwarder architecture and purpose
- Windows Event Log collection using inputs.conf
- Linux log file monitoring using add monitor command
- Difference between Windows and Linux logging systems
- Verifying forwarder connectivity to Splunk indexer
- Importance of monitoring the SIEM server itself
- Troubleshooting silent configuration failures

## Next Section
[4. Detection Engineering](../4-Detection-Engineering/README.md)
