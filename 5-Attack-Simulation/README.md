# 5. Attack Simulation

## Overview
Now that the alerts and dashboard are set up the next 
step was to actually test them. A Kali Linux machine 
running on the same internal network (10.0.0.140) was 
used to simulate attacks against the Ubuntu Server 
(10.0.0.26) and Windows PC (10.0.0.220). Since the 
Kali machine is internal the attacking IP will appear 
as a private address in Splunk — simulating a scenario 
where an attacker has already gained a foothold inside 
the network.

Three attacks were performed and investigated:
- SSH brute force against the Ubuntu Server
- Port scan against the Ubuntu Server  
- Brute force against the Windows PC

While the attacker perspective is briefly covered 
for context the primary focus is on what appeared 
in Splunk and how each attack was investigated.

---

## Attack 1 - SSH Brute Force Against Ubuntu Server

### Attacker Side
A wordlist of common passwords was created on the 
Kali machine with the actual account password included 
in the list

![Passwords](images/PasswordFile.png)



Hydra was then used to attempt each password against the SSH 
service on the Ubuntu Server.

    hydra -l socadmin -P passwords.txt ssh://10.0.0.26 -t 4 -V

Command breakdown:
- hydra = the brute force tool
- -l socadmin = the username being targeted
- -P passwords.txt = the password wordlist
- ssh://10.0.0.26 = the target and protocol
- -t 4 = 4 parallel threads
- -V = verbose output showing each attempt


![Attack](images/SuccessfulAttack.png)

---

### Defender Side

#### Discovery
While checking Splunk the following search was run 
against the Ubuntu Server's authentication log:

    index=main sourcetype=linux_secure "Failed password"
    | table _time, host, _raw
    | sort -_time

Setting the time range to "Today" immediately 
pulled up a stream of failed SSH login attempts all 
coming from the same IP address hitting the server 
back to back within seconds. 

![Defender Logs](images/DefenderLog.png)

#### Checking Triggered Alerts
After seeing the failed logins I navigated to Activity 
then Triggered Alerts. The brute force detection alert 
had already fired — confirming the alert was working 
as expected.

![Triggered Alerts](images/TriggeredAlerts.png)
![Triggered Alerts Table](images/BruteForceAlert.png)

#### Dashboard Overview
Pulling up the dashboard gave a quick visual of what 
was going on:

- Panel 1 had a noticeable spike on the Ubuntu Server
  line showing the sudden jump in failed logins
- Panel 2 showed the failed login count for today
  was higher than usual
- Panel 4 had the socadmin account sitting at the top
  with the most failed attempts

![Dashboard](images/Dashboard.png)

#### Investigation
The next step was checking whether any of those 
attempts actually succeeded:

    index=main sourcetype=linux_secure "Accepted"
    | rex field=_raw "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
    | where src_ip="10.0.0.140"
    | table _time, host, _raw
    | sort -_time

"Accepted password" from the attacking IP showed up. 
This confirms the attacker got in.

![Dashboard](images/BruteForceLog.png)

---

Running this search pulled up about 10 failed password 
check entries followed by a new session entry at the 
bottom. This is showing exactly where the failed attempts 
ended and the successful login began.

    index=main sourcetype=linux_secure
    | where src_ip="10.0.0.140" OR _raw LIKE "%socadmin%"
    | table _time, host, _raw
    | sort _time

![New Session](images/NewSession.png)

---

Any sudo commands run were checked:

    index=main sourcetype=linux_secure "sudo" "socadmin"
    | table _time, host, _raw
    | sort _time
    
Sudo activity from the attacker's session showed up, confirming the attacker was running privileged 
commands after gaining access.
![Sudo Command](images/SudoCommand.png)

---

New user accounts were checked to see if the attacker 
created a backdoor:

    index=main sourcetype=linux_secure "new user"
    | table _time, host, _raw
    | sort _time

A new account had been created. The attacker 
established a backdoor to maintain access even if 
the original credentials were changed.
![Backdoor](images/Backdoor.png)
