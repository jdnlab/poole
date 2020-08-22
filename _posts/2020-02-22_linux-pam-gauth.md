---
title: "provisioning google authenticator pam config updates via ansible"
date: 2020-02-22T02:17:20Z
---

Often, systems administrators use Linux as their underlying host for their work host/VM/VDI sessions. To protect such assets from unauthorized access from externalities such as red team operators, one could harden their host by adding another authentication factor such as [TOTP](https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm)/[HOTP](https://en.wikipedia.org/wiki/HMAC-based_One-time_Password_algorithm) soft tokens in addition to the default Unix username/password combination. Google Authenticator works great as another factor and even has a [compatible PAM (pluggable authentication module)](https://github.com/google/google-authenticator-libpam) ready for consumption.

Some of us provision using procedural/declarative infrastructure as code languages such as Ansible.  In this post, we will modify the default Unix `system-auth` PAM module to include the Google Authenticator PAM directive, such that all implementors of the `system-auth` PAM configuration file have a second authentication factor. This post assumes the following:
- familiarity with Ansible
- `google-authenticator-libpam` pam module  provisioned on their host and token sync completed (see link at top of post)
- `sudo` privileges to rollback if needed

## Technical Overview

On most Linux distributions, there exists a PAM configuration file located at `/etc/pam.d/system-auth`. Per its [`man page`](https://linux.die.net/man/5/system-auth), this provides a common configuration for PAM-ified services. Essentially, upon service provisioning, services requiring PAM support must drop a configuration file in `/etc/pam.d/`. Common services e.g. `login`, `sshd`, `sudo` typically reference the `system-auth` configuration file in theirs. Red Hat as a [great overview](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/managing_smart_cards/pam_configuration_files) of PAM configuration files and modules.

## Provisioning
By examining the `system-auth` PAM configuration, we see the following:

```
[~]$ cat /etc/pam.d/system-auth                                                                                                                                                  
#%PAM-1.0
# Updated by Ansible - 2020-02-18T01:59:26.852434

auth       required pam_unix.so
auth       optional pam_permit.so
auth       required pam_env.so

< stdout truncated >
[~]$      
```

Creating a new Ansible task is as simple as the following, which leverages @kevensen's [`pamd` module](https://docs.ansible.com/ansible/latest/modules/pamd_module.html). In this example, the 2FA prompt will *always* occur after the standard username/password Unix authentication module, [`pam_unix`](https://linux.die.net/man/8/pam_unix) due to the PAM modules' directive order. One could modify the `state` value to `before` if they desired for the 2FA collection to precede password collection. This before/after order is relative to the `pam_unix` PAM module, as indicated by `module_path` key.

```
---
- name: provision google authenticator
  hosts: managed
  become: yes

  tasks:
    - name: add system-auth pam directive for google authenticator
      pamd:
        backup: true
        name: system-auth
        new_type: auth
        new_control: required
        new_module_path: pam_google_authenticator.so
        state: after
        type: auth
        module_path: pam_unix.so
        control: required
```

If using HOTP (not applicable for TOTP), append K/V pair `module_arguments: 'no_increment_hotp'` to the `pamd` construct. This is to prevent the module failed auth counter from incrementing during valid authentication attempts, per the `google-authenticator-libpam` README. 

__Changes will take place as soon as Ansible establishes a connection (localhost or over TCP), so ensure you have a fallback method, e.g. an existing elevated shell session for reverting__

After executing `ansible-playbook` on the playbook, we can see that `Verification code: ` is mandatory on all `system-auth` implementors -- `sudo`, gnome (leverages `polkit`), etc.

```
#%PAM-1.0
# Updated by Ansible - 2020-02-22T03:23:31.780905

auth       required pam_unix.so
auth       required pam_google_authenticator.so
auth       optional pam_permit.so
auth       required pam_env.so

< stdout truncated >
```

## De-Provisioning

In the event of deprovisioning, one way a sample playbook can be expressed is as such:

```
---
- name: deprovision google authenticator
  hosts: managed
  become: yes

  tasks:
    - name: remove system-auth pam directive for google authenticator
      pamd:
        backup: true
        name: system-auth
        state: absent
        type: auth
        module_path: pam_google_authenticator.so
        control: required
```

Execution shall use `sudo`, as Ansible currently does not support interactive `become` authentication with the mandatory MFA requirement.
