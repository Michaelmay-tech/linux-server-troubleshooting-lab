# Linux Server Troubleshooting Lab

## Project Overview

Built and performed hands-on troubleshooting exercises within an Ubuntu Server virtual machine to simulate common Linux server and infrastructure issues.

The lab focused on identifying service, DNS, network connectivity, and storage problems and verifying system functionality after each repair.

## Environment

- Ubuntu Server
- UTM Virtualization
- Nginx
- SSH
- TCP/IP
- DNS
- systemd
- systemd-resolved
- Linux networking
- LVM storage management

---

# Scenario 1: Nginx Service Failure

## Problem

The Nginx web service was intentionally stopped to simulate a service outage.

## Troubleshooting

The following commands were used to investigate the issue:

```bash
systemctl status nginx
curl -I http://localhost
