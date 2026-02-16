# Ansible Error: SSH Password Cannot Be Used with Host Key Checking Enabled


<br>
<br>

## Error Message

```
fatal: [hostname]: FAILED! =>
  msg: Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host.
```


- [Ansible Error: SSH Password Cannot Be Used with Host Key Checking Enabled](#ansible-error-ssh-password-cannot-be-used-with-host-key-checking-enabled)
  - [Error Message](#error-message)
  - [What This Error Means](#what-this-error-means)
  - [What Is Host Key Checking](#what-is-host-key-checking)
  - [Why This Error Happens](#why-this-error-happens)
  - [Secure Fix (Recommended Method)](#secure-fix-recommended-method)
  - [Alternative: Disable Host Key Checking (Less Secure)](#alternative-disable-host-key-checking-less-secure)
    - [Temporary (One-Time Execution)](#temporary-one-time-execution)
    - [Permanent (Inside ansible.cfg)](#permanent-inside-ansiblecfg)
  - [Best Practice Recommendation](#best-practice-recommendation)
- [Final Summary](#final-summary)


## What This Error Means

Ansible attempted to connect to a remote server using SSH password authentication.

At the same time, SSH host key checking was enabled.

Because the host was not already trusted and its fingerprint was not stored in the local `known_hosts` file, SSH expected manual confirmation.

However, Ansible was using `sshpass`.

`sshpass` is a utility that allows non-interactive SSH login using a password. It cannot respond to interactive security prompts such as accepting a new server fingerprint.

As a result, Ansible refused to connect and stopped execution.

---
<br>
<br>


## What Is Host Key Checking

Host key checking is a security feature built into SSH.

When you connect to a server for the first time using a command like:

```
ssh user@192.168.1.10
```

SSH displays the server’s unique fingerprint and asks whether you trust it.

If you type `yes`, the fingerprint is stored in a file called `known_hosts` on your local system.

On future connections, SSH compares the server’s fingerprint with the saved one.

If they match, the connection proceeds.

If they do not match, SSH blocks the connection because this could indicate:

* The server was reinstalled
* The IP address now belongs to a different system
* A possible man-in-the-middle attack attempt

In short, host key checking ensures you are connecting to the same trusted server every time.

---
<br>
<br>


## Why This Error Happens

This situation occurs when all of the following are true:

* SSH password authentication is being used
* The target host is not present in `known_hosts`
* Host key checking is enabled (default behavior)
* Ansible relies on `sshpass`

Since `sshpass` cannot accept the fingerprint interactively, the connection fails.

---
<br>
<br>


## Secure Fix (Recommended Method)

Manually connect to the server once:

```
ssh user@b1h-devlinkdocker4
```

When prompted:

```
Are you sure you want to continue connecting (yes/no)?
```

Type:

```
yes
```

This stores the server fingerprint in `known_hosts`.

After this, rerun the playbook. Ansible will connect successfully because the host is now trusted.

This is the secure and correct approach.

---
<br>
<br>


## Alternative: Disable Host Key Checking (Less Secure)

### Temporary (One-Time Execution)

```
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i myini.ini myplaybook.yml
```

This disables host key verification only for that command execution.

### Permanent (Inside ansible.cfg)

```
[defaults]
host_key_checking = False
```

This disables host verification globally for Ansible.

Security Risk:

Disabling host key checking removes protection against connecting to an unintended or malicious server.

Use this only in controlled lab or testing environments.

---
<br>
<br>


## Best Practice Recommendation

Use SSH key-based authentication instead of passwords.

With key-based authentication:

* No need for `sshpass`
* More secure than passwords
* Works smoothly with automation
* Avoids this host key prompt issue

Generate a key pair if needed:

```
ssh-keygen
```

Copy the public key to the server:

```
ssh-copy-id user@server_ip
```

After configuring key-based login and trusting the host once, Ansible connections become fully non-interactive and reliable.

---
<br>
<br>


# Final Summary

This error is not an SSH failure.

It happens because SSH host key verification requires manual confirmation, but `sshpass` cannot respond to that confirmation.

The correct solution is to trust the host once manually or move to SSH key-based authentication for secure and stable automation.
