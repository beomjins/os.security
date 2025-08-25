# os.security
Ansible role to perform security hardening tasks on Rocky Linux systems.

## Requirements
- Ansible 2.11 or higher
- Supported OS: Rocky Linux 8.x

## Role Variables
See [defaults/main.yml](defaults/main.yml) for default variables.
Some important variables you can override in `group_vars` or playbook:

| Variable | Description | Default |
|----------|-------------|---------|
| `ansible_users` | List of users to apply password aging policy (optional) | unset |
| `target_ssh_port` | SSH server port (22 is default; migrate safely when changing) | 22 |
| `faillock_deny` | Maximum number of authentication failures before locking account | 5 |
| `faillock_unlock_time` | Time in seconds after which the account is unlocked | 1800 |
| `pass_min_len` | Minimum password length | 9 |
| `pass_max_days` | Maximum password age in days | 90 |
| `pass_min_days` | Minimum password age in days | 1 |
| `password_exceptions` | List of users to skip password policy | ['root','Username'] |
| `password_exception_enabled` | Enable skipping password policy for exceptions | true |

## Applied Security Hardening Items

This role configures the following security policies:

1. Authentication & Account Lock
   - Enables sssd profile with authselect
   - Activates with-faillock feature
   - Configures /etc/security/faillock.conf (deny=5, unlock_time=1800, fail_interval=900)

2. Password Complexity
   - Applies /etc/security/pwquality.conf rules:
     - minlen=9, minclass=1
     - must include lowercase, uppercase, digit, special char
     - prevents repeat and class-repeat

3. Password Aging
   - Sets /etc/login.defs parameters:
     - PASS_MAX_DAYS=90, PASS_MIN_DAYS=7, PASS_WARN_AGE=14
   - Excludes certain accounts (root, Username) if password_exception_enabled is true

4. Disable Unnecessary Accounts
   - Sets sync, shutdown, halt shell to /sbin/nologin

5. Remove Dangerous SUID Binaries
   - Strips SUID bit from /usr/bin/newgrp, /sbin/unix_chkpwd

6. UMASK Enforcement
   - Ensures umask 0022 in target user’s .bashrc

7. MOTD Warning Banner
   - Deploys /etc/motd with system usage warning

8. Change SSH Port
   - Works if sshport is not 22
   - Syncs sshd_config Port with ansible_port
   - Restarts sshd only when configuration changes

## Dependencies
- None (this role is standalone)

## Example Playbook
- hosts: all
  become: yes
  roles:
    - { role: runtime.security }

## License
- Code released under [Apache License 2.0](LICENSE)
- Documentation released under [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/)

------------------------------------------

    - Beomjin Kim
    - <https://github.com/beomjins>
