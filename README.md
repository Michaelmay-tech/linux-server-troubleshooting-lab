# Linux Server Troubleshooting Lab

## Project Overview

Built and performed hands-on troubleshooting exercises within an Ubuntu Server virtual machine to simulate common Linux server and infrastructure issues.

This project focused on identifying service, DNS, network connectivity, and storage problems. Each issue was intentionally introduced, investigated using Linux command-line tools, resolved, and verified to ensure normal server operation was restored.

The goal of this lab was to develop practical troubleshooting skills relevant to Linux server administration and data center operations.

---

## Objectives

The objectives of this lab were to:

- Identify Linux service failures
- Troubleshoot application connectivity issues
- Differentiate between network connectivity and DNS resolution problems
- Investigate network interface and routing issues
- Recover from network outages using console access
- Inspect Linux storage and LVM configurations
- Expand a logical volume and filesystem
- Verify system functionality after repairs

---

## Lab Environment

- Ubuntu Server
- UTM Virtualization
- Nginx
- SSH
- TCP/IP
- DNS
- systemd
- systemd-resolved
- DHCP
- Linux Networking
- LVM Storage Management

---

## Baseline System Configuration

Before beginning troubleshooting scenarios, the server was verified to be operating normally.

### Server Information

- Hostname: `dc-lab-01`
- Network Interface: `enp0s1`
- IPv4 Address: `192.168.64.2/24`
- Default Gateway: `192.168.64.1`
- Operating System: Ubuntu Server

### Baseline Health Checks

The following areas were tested:

- Network connectivity using an IP address
- DNS hostname resolution
- Nginx service status
- HTTP connectivity
- Disk utilization

Commands used:

```bash
hostname
ip a
ping -c 4 8.8.8.8
ping -c 4 google.com
systemctl status nginx
df -h
```

The system was confirmed to have working network connectivity, DNS resolution, an active Nginx service, and available storage capacity before troubleshooting scenarios began.

---

# Troubleshooting Methodology

Each troubleshooting scenario followed a similar process:

1. Identify the reported problem or system failure
2. Gather information about the affected service or system component
3. Test connectivity or functionality
4. Isolate the likely cause of the issue
5. Apply the appropriate resolution
6. Verify that the system returned to normal operation

This approach emphasizes verifying the problem before making changes and confirming functionality after a repair.

---

# Scenario 1: Nginx Service Failure

## Incident Description

The Nginx web service was intentionally stopped to simulate a server-side application outage.

The objective was to identify the failed service, restore the service, and verify that the web server was responding correctly.

---

## Symptoms

The Nginx service was no longer running.

A local HTTP request to the server failed because no process was listening on TCP port 80.

---

## Investigation

The service status was checked:

```bash
systemctl status nginx
```

The output showed that the service was:

```text
inactive (dead)
```

Application connectivity was then tested:

```bash
curl -I http://localhost
```

The connection failed because the Nginx service was not running.

---

## Root Cause

The Nginx service had been stopped, resulting in no web server process listening on TCP port 80.

---

## Resolution

The service was started:

```bash
sudo systemctl start nginx
```

---

## Verification

The following commands were used to verify the repair:

```bash
systemctl status nginx
curl -I http://localhost
sudo ss -tulpn | grep :80
```

Results confirmed:

- Nginx was active and running
- The server returned an HTTP `200 OK` response
- Nginx was listening on TCP port 80

---

# Scenario 2: DNS Resolution Failure

## Incident Description

The DNS resolution service was intentionally stopped to simulate a hostname resolution failure.

The objective was to determine whether the issue was related to general network connectivity or DNS.

---

## Symptoms

The server maintained network connectivity when communicating directly with an IP address.

However, hostname resolution failed.

---

## Investigation

Network connectivity was tested using a public IP address:

```bash
ping -c 4 8.8.8.8
```

The ping completed successfully, confirming that the server still had network connectivity.

DNS resolution was then tested:

```bash
ping -c 4 google.com
```

The system returned:

```text
Temporary failure in name resolution
```

The DNS service was checked:

```bash
systemctl status systemd-resolved
```

The service was found to be inactive.

The DNS configuration was also inspected using:

```bash
resolvectl status
cat /etc/resolv.conf
```

---

## Root Cause

The `systemd-resolved` service was stopped, preventing the server from resolving hostnames.

The network connection itself remained operational.

---

## Resolution

The DNS resolver service was restarted:

```bash
sudo systemctl start systemd-resolved
```

---

## Verification

The DNS service was verified:

```bash
systemctl status systemd-resolved
```

Hostname resolution was tested again:

```bash
ping -c 4 google.com
```

DNS configuration was confirmed using:

```bash
resolvectl status
```

Results confirmed that DNS resolution was successfully restored.

---

# Scenario 3: Network Interface Outage

## Incident Description

The primary network interface was intentionally disabled to simulate a network outage.

The objective was to investigate the interface state, restore network connectivity, and verify that DHCP and routing configuration returned to normal.

---

## Baseline Configuration

Before the outage, the network interface and routing table were inspected:

```bash
ip link show enp0s1
ip route
```

The server had:

- Interface: `enp0s1`
- IPv4 Address: `192.168.64.2`
- Default Gateway: `192.168.64.1`

---

## Symptoms

The network interface was disabled.

This resulted in:

- Loss of network connectivity
- Loss of SSH connectivity
- Inability to communicate with external IP addresses

The SSH connection was disconnected because the disabled interface was also being used for remote management.

---

## Investigation

The network interface state was inspected:

```bash
ip link show enp0s1
```

The interface was reported as:

```text
state DOWN
```

Additional network information was inspected using:

```bash
ip a
networkctl status enp0s1
```

The network service logs indicated that the interface link was down and the DHCP lease had been lost.

---

## Recovery

Because SSH access was unavailable, the UTM virtual machine console was used as an alternative management method.

The network interface was restored:

```bash
sudo ip link set enp0s1 up
```

The interface successfully returned to an operational state and reacquired its DHCP configuration.

---

## Verification

The network configuration was verified:

```bash
ip link show enp0s1
ip a show enp0s1
ip route
```

Network connectivity was tested:

```bash
ping -c 4 8.8.8.8
```

DNS functionality was also tested:

```bash
ping -c 4 google.com
```

Results confirmed:

- Network interface returned to an UP state
- DHCP restored the IPv4 address
- The default route was restored
- Internet connectivity was operational
- DNS resolution was operational

---

## Key Lesson

This scenario demonstrated the importance of having console or out-of-band access available when performing network changes on remotely managed systems.

Disabling the same network interface being used for SSH management can result in loss of remote access.

---

# Scenario 4: LVM Storage Capacity Management

## Incident Description

The server required additional storage capacity.

The objective was to inspect the existing storage configuration, identify available capacity, expand the logical volume, and verify the filesystem expansion.

---

## Storage Configuration

The server storage layout was inspected:

```bash
df -h
lsblk
```

The server contained:

- 32 GB virtual disk
- LVM physical volume on `/dev/vda3`
- Volume group named `ubuntu-vg`
- Logical volume named `ubuntu-lv`

The root filesystem initially had approximately:

- 15 GB total capacity
- 4.8 GB used
- 8.7 GB available

---

## LVM Investigation

The LVM configuration was inspected using:

```bash
sudo pvs
sudo vgs
sudo lvs
```

The investigation showed that approximately 14.47 GB of unused capacity was available within the `ubuntu-vg` volume group.

---

## Root Cause / Capacity Requirement

The logical volume was smaller than the available capacity of the volume group.

Additional storage was available but had not yet been allocated to the root logical volume.

---

## Resolution

The root logical volume was expanded using all available free space:

```bash
sudo lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv
```

The command performed two operations:

- Expanded the logical volume
- Resized the filesystem online

The server did not require downtime to complete the filesystem expansion.

---

## Verification

The root filesystem was checked:

```bash
df -h /
```

The logical volume configuration was verified:

```bash
sudo lvs
```

### Results

The root filesystem successfully expanded from approximately:

```text
15 GB
```

to:

```text
29 GB
```

Available storage increased from approximately 8.7 GB to approximately 23 GB.

---

# Final System Health Verification

After completing all troubleshooting scenarios, the system was tested to verify that normal operation had been restored.

Commands used:

```bash
hostname
ping -c 2 8.8.8.8
ping -c 2 google.com
systemctl is-active nginx
systemctl is-active systemd-resolved
df -h /
```

The final verification confirmed:

- Network connectivity was operational
- DNS resolution was operational
- Nginx was running
- The network interface was functional
- The default route was restored
- The root filesystem capacity was successfully expanded

---

# Skills Demonstrated

- Linux Server Administration
- Linux Troubleshooting
- Command-Line Diagnostics
- systemd Service Management
- Nginx Administration
- HTTP Connectivity Testing
- Network Troubleshooting
- TCP/IP
- DNS Troubleshooting
- systemd-resolved
- SSH
- Network Interface Management
- DHCP
- IP Routing
- Console-Based System Recovery
- Linux Storage Management
- Logical Volume Manager (LVM)
- Filesystem Expansion
- System Health Verification

---

# Key Takeaways

This project demonstrated the importance of a structured troubleshooting process when diagnosing Linux infrastructure issues.

Key lessons included:

- Verify service status before restarting services
- Differentiate between network connectivity failures and DNS failures
- Verify repairs using multiple methods
- Maintain console access when making remote network changes
- Inspect available storage capacity before expanding filesystems
- Verify system functionality after making configuration changes

---

# Conclusion

This project simulated common Linux server and infrastructure failures involving application services, DNS resolution, network connectivity, and storage capacity management.

Each scenario was intentionally created, investigated using Linux command-line tools, resolved, and verified.

The completed lab provided hands-on experience with troubleshooting workflows relevant to Linux server administration, data center operations, and infrastructure support.
