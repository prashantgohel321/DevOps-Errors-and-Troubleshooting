# Ansible Error: Invalid or Incorrect SSH Password (UNREACHABLE)


## Error Message

```
fatal: [hostname]: UNREACHABLE! => changed=false
  msg: |-
    Invalid/incorrect password: Warning: Permanently added '192.168.111.189' (ED25519) to the list of known hosts.
    Permission denied, please try again.
  unreachable: true
```

<br>
<br>

- [Ansible Error: Invalid or Incorrect SSH Password (UNREACHABLE)](#ansible-error-invalid-or-incorrect-ssh-password-unreachable)
  - [Error Message](#error-message)
  - [What This Error Means](#what-this-error-means)
  - [What Is Happening in the Background](#what-is-happening-in-the-background)
  - [Possible Causes](#possible-causes)
  - [What to Check](#what-to-check)
    - [1️⃣ Verify Manually](#1️⃣-verify-manually)
    - [2️⃣ Check Inventory Configuration](#2️⃣-check-inventory-configuration)
    - [3️⃣ Check If Password Authentication Is Disabled](#3️⃣-check-if-password-authentication-is-disabled)
    - [4️⃣ Check Account Status](#4️⃣-check-account-status)
- [Secure and Recommended Solution](#secure-and-recommended-solution)
  - [Use SSH Key-Based Authentication](#use-ssh-key-based-authentication)
- [Final Summary](#final-summary)


<br>
<br>

## What This Error Means

Ansible successfully reached the remote server over the network.

It even added the server’s fingerprint to the local `known_hosts` file, which means SSH connectivity worked.

However, authentication failed.

The message clearly shows:

Invalid/incorrect password
Permission denied, please try again.

This means the SSH credentials provided to Ansible were incorrect or not allowed.

In simple terms: the server was reachable, but login failed.

---

<br>
<br>


## What Is Happening in the Background

When Ansible connects using SSH with a password, it attempts:

* Network connection to the server
* Host verification (fingerprint check)
* User authentication using provided credentials

In this case:

* Network connection succeeded
* Host key was accepted and stored
* Password authentication failed

Since authentication failed, Ansible marks the host as `UNREACHABLE`.

`UNREACHABLE` in Ansible does not always mean network failure. It can also mean authentication failure.

---

<br>
<br>


## Possible Causes

1️⃣ Wrong SSH password
2️⃣ Wrong SSH username
3️⃣ Server allows only SSH key authentication
4️⃣ User account is locked or password expired
5️⃣ Incorrect inventory configuration

---

<br>
<br>


## What to Check

### 1️⃣ Verify Manually

Test directly from your system:

```
ssh user@192.168.111.189
```

If it fails with "Permission denied", then either:

* The username is incorrect
* The password is incorrect
* Password login is disabled

Manual testing isolates whether the problem is Ansible-specific or general SSH authentication.

---

<br>
<br>


### 2️⃣ Check Inventory Configuration

Example inventory entry:

```
b1h-devlinkdocker3 ansible_user=username ansible_password=correctpassword
```

Verify:

* `ansible_user` matches a valid user on the server
* `ansible_password` is correct
* There are no typing errors or hidden characters

Avoid trailing spaces in passwords.

---

<br>
<br>


### 3️⃣ Check If Password Authentication Is Disabled

On the target server, check the SSH configuration file:

```
/etc/ssh/sshd_config
```

Look for:

```
PasswordAuthentication no
```

If this is set to `no`, the server will not allow password-based login.

In that case, authentication must be done using SSH keys.

---

<br>
<br>


### 4️⃣ Check Account Status

If the account exists but login still fails, verify:

* Account is not locked
* Password has not expired
* User is allowed to log in via SSH

System administrators can check this using:

```
passwd -S username
```

or by reviewing system logs.

---

<br>
<br>


# Secure and Recommended Solution

## Use SSH Key-Based Authentication

SSH key-based authentication is more secure and more reliable than passwords.

Generate a key pair if not already available:

```
ssh-keygen
```

Copy the public key to the server:

```
ssh-copy-id user@192.168.111.189
```

After successful setup:

* Remove `ansible_password` from inventory
* Ensure `ansible_user` is correct

Ansible will then authenticate using the private key automatically.

Benefits:

* No plain-text passwords in inventory
* No password expiration issues
* More secure authentication mechanism
* Better suited for automation

---

<br>
<br>


# Final Summary

This error does not indicate a network problem.

It indicates that SSH authentication failed because the credentials were incorrect or not permitted.

Ansible reached the server, but login was denied.

Correcting the username, password, SSH configuration, or moving to SSH key-based authentication resolves the issue.
