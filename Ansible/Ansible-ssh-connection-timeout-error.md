# Ansible Error: SSH Connection Timed Out (UNREACHABLE)

## Error Message

```bash
fatal: [hostname]: UNREACHABLE! => changed=false
  msg: |-
    Failed to connect to the host via ssh: Connection timed out during banner exchange
    Connection to 192.168.x.x port 22 timed out
  unreachable: true
```

- [Ansible Error: SSH Connection Timed Out (UNREACHABLE)](#ansible-error-ssh-connection-timed-out-unreachable)
  - [Error Message](#error-message)
  - [What This Error Means](#what-this-error-means)
  - [What Is "Banner Exchange"](#what-is-banner-exchange)
  - [Most Common Causes](#most-common-causes)
  - [Quick Connectivity Check](#quick-connectivity-check)
  - [What to Verify Step by Step](#what-to-verify-step-by-step)
    - [1. Confirm Server Is Reachable](#1-confirm-server-is-reachable)
    - [2. Check SSH Service on Target](#2-check-ssh-service-on-target)
    - [3. Check Firewall Rules](#3-check-firewall-rules)
    - [4. Check If SSH Uses a Different Port](#4-check-if-ssh-uses-a-different-port)
    - [5. Verify Inventory Configuration](#5-verify-inventory-configuration)
- [Best Practice Recommendations](#best-practice-recommendations)
- [Final Summary](#final-summary)


<br>
<br>

## What This Error Means

- Ansible attempted to connect to the remote server using SSH.
- The connection could not be established.
- The message "Connection timed out during banner exchange" indicates that the client tried to open a connection to port 22, but the server did not respond.

<br>
<br>

- This is not an authentication problem.
- This is not a privilege escalation issue.
- This is a network-level connectivity problem.

In simple terms: Ansible could not reach the SSH service on the target server.

---


<br>
<br>


## What Is "Banner Exchange"

- When an SSH connection starts, the client and server exchange identification information before authentication begins.
- This initial handshake is called the banner exchange.
- If the server does not respond during this phase, the connection times out.

<br>

That usually means:

* The SSH service is not reachable
* The port is blocked
* The server is down

---


<br>
<br>


## Most Common Causes

1. SSH service is not running on the target server
2. Port 22 is blocked by a firewall or security rule
3. Incorrect IP address in inventory
4. Server is powered off or unreachable
5. SSH is running on a different port
6. Network routing issue between control node and target

---


<br>
<br>


## Quick Connectivity Check

From the control node (where Ansible runs), test port access:

```bash
nc -zv 192.168.x.x 22
```

If it shows timeout → port 22 is not reachable.

You can also test with:

```bash
ssh user@192.168.x.x
```

If this hangs or times out, the issue is outside Ansible.

---


<br>
<br>


## What to Verify Step by Step

### 1. Confirm Server Is Reachable

Ping the server:

```bash
ping 192.168.x.x
```

If ping fails, check:

* Network configuration
* VPN connectivity
* Routing

---


<br>
<br>


### 2. Check SSH Service on Target

On the target server (via console or direct access), verify SSH service:

```bash
systemctl status sshd
```

If not running, start it:

```bash
systemctl start sshd
```

Enable it at boot:

```bash
systemctl enable sshd
```

---


<br>
<br>


### 3. Check Firewall Rules

On the target server, verify firewall allows port 22:

```bash
firewall-cmd --list-all
```

or check iptables rules if applicable.

If port 22 is blocked, allow it.

---


<br>
<br>


### 4. Check If SSH Uses a Different Port

In `/etc/ssh/sshd_config`, look for:

```
Port 2222
```

If SSH runs on a different port, update your inventory:

```
hostname ansible_host=192.168.x.x ansible_port=2222
```

---


<br>
<br>


### 5. Verify Inventory Configuration

Ensure the IP address is correct and not outdated.

Incorrect IP addresses are a common cause in dynamic environments.

---


<br>
<br>


# Best Practice Recommendations

* Always verify SSH connectivity manually before running playbooks
* Use monitoring to ensure SSH service is running
* Restrict firewall rules properly but allow required automation access
* Keep inventory updated and accurate

---


<br>
<br>


# Final Summary

- This error means Ansible could not establish an SSH connection to the target server.
- The timeout occurred before authentication.
- This is a connectivity or network issue, not an Ansible configuration issue.
- Fixing SSH availability, firewall rules, network routing, or correct port configuration resolves the problem.
