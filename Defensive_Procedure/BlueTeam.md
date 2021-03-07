# Blue Team: Summary of Operations

## Table of Contents
- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic & Behavior
- Suggestions for Going Further

### Network Topology

The following machines were identified on the network:
- Name of VM 1: `Target1`
  - **Operating System**: `Linux 3.X|4.X`
  - **Purpose**: `Web Server - Vulnerable VM`
  - **IP Address**: `192.168.1.110`
- Name of VM 2: `Kali`
  - **Operating System**: `Linux 2.6.32`  
  - **Purpose**: `Pen Testing Machine`
  - **IP Address**: `192.168.1.90`
- Name of VM 3: `Capstone` 
  - **Operating System**: `No exact OS Match but appears to be Linux`
  - **Purpose**: `Alert Testing Machine`
  - **IP Address**: `192.168.1.105`
- Name of VM 4: `ELK`
  - **Operating System**: `No exact OS Match but appears to be Linux`  
  - **Purpose**: `Gathers Logs and Alerts: Elasticsearch, Logstash, Kibana`
  - **IP Address**: `192.168.1.100`
- Name of VM 5: `Target2` 
  - **Operating System**: `Linux 3.X|4.X`
  - **Purpose**: `Web Server - Vulnerable VM`
  - **IP Address**: `192.168.1.115`

### Description of Targets

The target of this attack was: `Target 1: 192.168.1.110`

Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

### Monitoring the Targets

Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

#### Name of Alert 1
Alert 1: `HTTP Errors Alert:`

Implemented as follows:
  - **Metric**: `HTTP Status Errors`
  - **Threshold**: `Above 400 over 1 minute`
  - **Vulnerability Mitigated**: `Brute Force Attacks`
  - **Reliability**: `High: Provides critical data against Brute Force Attacks`

#### Name of Alert 2
Alert 2: `Failed SSH Password Attempts:`

Implemented as follows:
  - **Metric**: `system.auth.ssh.method`
  - **Threshold**: `Above 5 over the last 5 minutes`
  - **Vulnerability Mitigated**: `Unwarranted SSH Login Attempts`
  - **Reliability**: `High: Provides data to make sure unauthorized users aren't trying to SSH into the network`

#### Name of Alert 3
Alert 3: `Port Scan:`

Implemented as follows:
  - **Metric**: `source.port`
  - **Threshold**: `Above 2000 over the last 5 minutes`
  - **Vulnerability Mitigated**: `Port Scanning - to see what ports are open on the network`
  - **Reliability**: `High: Important to know if people are scanning your network for open ports and vulnerabilities`

#### Name of Alert 4
Alert 3: `Sudo Commands Alert:`

Implemented as follows:
  - **Metric**: `system.auth.sudo.error`
  - **Threshold**: `Above 1 over the last 5 minutes`
  - **Vulnerability Mitigated**: `Catching a User trying to access restricted folders or gain root privileges`
  - **Reliability**: `High: An attacker using sudo or gaining root access can have irreversible effects. There is a slight chance of false alerts due to user accidents - but users should know their access and what they can view`

  #### Name of Alert 5
Alert 3: `CPU Usage Monitor:`

Implemented as follows:
  - **Metric**: `system.process.cpu.total`
  - **Threshold**: `Above .5 over the last 5 minutes`
  - **Vulnerability Mitigated**: `DDoS Attacks`
  - **Reliability**: `Medium - High: Keeping our network up and running by allowing normal traffic and speeds is important`


### Suggestions for Going Further (Optional)
_TODO_: 
- Each alert above pertains to a specific vulnerability/exploit. Recall that alerts only detect malicious behavior, but do not stop it. For each vulnerability/exploit identified by the alerts above, suggest a patch. E.g., implementing a blocklist is an effective tactic against brute-force attacks. It is not necessary to explain _how_ to implement each patch.

The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats, identified by the alerts above. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:
- Vulnerability 1
  - **Patch**: TODO: E.g., _install `special-security-package` with `apt-get`_
  - **Why It Works**: TODO: E.g., _`special-security-package` scans the system for viruses every day_
- Vulnerability 2
  - **Patch**: TODO: E.g., _install `special-security-package` with `apt-get`_
  - **Why It Works**: TODO: E.g., _`special-security-package` scans the system for viruses every day_
- Vulnerability 3
  - **Patch**: TODO: E.g., _install `special-security-package` with `apt-get`_
  - **Why It Works**: TODO: E.g., _`special-security-package` scans the system for viruses every day_