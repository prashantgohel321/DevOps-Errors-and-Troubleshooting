# Ansible Error: Timeout Waiting for Privilege Escalation Prompt


<br>
<br>
## Error Message

```
fatal: [hostname]: FAILED! =>
  msg: 'Timeout (12s) waiting for privilege escalation prompt: '
```

- [Ansible Error: Timeout Waiting for Privilege Escalation Prompt](#ansible-error-timeout-waiting-for-privilege-escalation-prompt)
  - [Error Message](#error-message)
  - [What This Error Means](#what-this-error-means)
  - [Why This Happens](#why-this-happens)
  - [Real Practical Example](#real-practical-example)
- [Secure and Legitimate Fix Options](#secure-and-legitimate-fix-options)
  - [Option 1: Ask for Sudo Password During Execution (Secure for Manual Runs)](#option-1-ask-for-sudo-password-during-execution-secure-for-manual-runs)
  - [Option 2: Configure Passwordless Sudo (Best for Automation)](#option-2-configure-passwordless-sudo-best-for-automation)
  - [Option 3: Store Become Password Securely Using Ansible Vault (Secure and Controlled)](#option-3-store-become-password-securely-using-ansible-vault-secure-and-controlled)
  - [Option 4: Define Become Password in Inventory (Not Recommended for Production)](#option-4-define-become-password-in-inventory-not-recommended-for-production)
  - [Verification Steps After Fix](#verification-steps-after-fix)
- [Final Summary](#final-summary)



<br>
<br>
## What This Error Means

Ansible successfully connected to the target server over SSH. The connection worked correctly.

The failure happened after login, when Ansible attempted to execute a task with elevated privileges using **privilege escalation**.

Privilege escalation in Ansible is controlled using `become: yes`. This tells Ansible to run tasks as another user, usually `root`, by using `sudo`.

`sudo` (short for "superuser do") is a Linux command that allows a normal user to execute commands with administrative permissions.

When Ansible tried to use sudo, the server prompted for a sudo password. Since Ansible was not given that password, it waited for 12 seconds and then stopped with a timeout.

In simple terms: the server was waiting for a sudo password, but Ansible had nothing to provide.

---

<br>
<br>

## Why This Happens

This usually occurs when the playbook contains:

```
become: yes
```

or

```
become: true
```

When this setting is enabled:

* Ansible connects as the remote SSH user
* Then tries to execute commands using sudo
* The remote user requires a password for sudo
* No password was provided to Ansible

As a result, Ansible waits for the password prompt and eventually times out.

---

<br>
<br>

## Real Practical Example

Consider this task:

```
- name: Install nginx
  yum:
    name: nginx
    state: present
  become: yes
```

Here:

* `yum` is the package manager used in RHEL/CentOS systems
* Installing packages requires root privileges
* `become: yes` tells Ansible to run the task using sudo

Now manually test on the server:

```
sudo yum install nginx
```

If the system asks for a password, that means sudo authentication is required.

If Ansible is not configured with that password, it cannot proceed and produces the timeout error.

---

<br>
<br>

# Secure and Legitimate Fix Options

Below are practical, production-safe solutions. Choose based on your environment.

---

## Option 1: Ask for Sudo Password During Execution (Secure for Manual Runs)

Run the playbook with:

```
ansible-playbook -i myini.ini myplaybook.yml --ask-become-pass
```

`--ask-become-pass` tells Ansible to prompt you for the sudo password before starting execution.

This is secure because:

* The password is not stored in files
* It is not saved in plain text
* It is entered interactively

This method is suitable for:

* Personal environments
* One-time runs
* Testing

It is not ideal for automation pipelines.

---

<br>
<br>

## Option 2: Configure Passwordless Sudo (Best for Automation)

In automation environments such as CI/CD pipelines, cron jobs, or infrastructure deployments, interactive password prompts are not practical.

In this case, configure passwordless sudo for the Ansible user.

On the target server, edit the sudoers file safely:

```
sudo visudo
```

`visudo` is a safe editor for the sudoers file. It prevents syntax errors that could break sudo completely.

Add this line:

```
username ALL=(ALL) NOPASSWD: ALL
```

Replace `username` with the SSH user used by Ansible.

What this means:

* `ALL=(ALL)` → the user can run commands as any user
* `NOPASSWD` → no password required
* `ALL` → applies to all commands

This method is considered best practice for controlled automation environments.

For higher security, instead of allowing all commands, restrict sudo access to specific commands only.

Example:

```
username ALL=(ALL) NOPASSWD: /usr/bin/yum, /usr/bin/systemctl
```

This limits what the automation user can execute.

---

<br>
<br>

## Option 3: Store Become Password Securely Using Ansible Vault (Secure and Controlled)

If passwordless sudo is not allowed due to security policies, you can securely store the sudo password using Ansible Vault.

Ansible Vault encrypts sensitive data such as passwords inside a file.

Create a vaulted variable file:

```
ansible-vault create secrets.yml
```

Inside it:

```
ansible_become_password: your_sudo_password
```

Then reference it in your playbook and run:

```
ansible-playbook myplaybook.yml --ask-vault-pass
```

This method:

* Keeps passwords encrypted
* Avoids plain-text exposure
* Works well in team environments

It is secure and enterprise-appropriate when managed properly.

---

<br>
<br>

## Option 4: Define Become Password in Inventory (Not Recommended for Production)

You can define:

```
ansible_become_password=yourpassword
```

inside the inventory file.

This works technically, but storing passwords in plain text is not secure.

Use only in temporary lab environments.

---

<br>
<br>

## Verification Steps After Fix

After applying a solution, verify manually on the server:

```
sudo -l
```

This lists allowed sudo commands.

Then test:

```
sudo whoami
```

If it prints:

```
root
```

without asking for a password (or works after providing it when expected), Ansible will function correctly.

---

<br>
<br>

# Final Summary

This error does not indicate an SSH failure.

It indicates that Ansible required privilege escalation using sudo, but the sudo password was not supplied or properly configured.

Best practice depends on environment:

* Manual usage → `--ask-become-pass`
* Automation pipelines → passwordless sudo with restrictions
* Security-restricted environments → Ansible Vault

Once sudo authentication is correctly handled, the timeout error is fully resolved.
