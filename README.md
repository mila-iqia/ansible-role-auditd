# [Ansible role auditd](#auditd)

Install and configure auditd on your system.

|GitHub|GitLab|Downloads|Version|
|------|------|---------|-------|
|[![github](https://github.com/robertdebock/ansible-role-auditd/workflows/Ansible%20Molecule/badge.svg)](https://github.com/robertdebock/ansible-role-auditd/actions)|[![gitlab](https://gitlab.com/robertdebock-iac/ansible-role-auditd/badges/master/pipeline.svg)](https://gitlab.com/robertdebock-iac/ansible-role-auditd)|[![downloads](https://img.shields.io/ansible/role/d/robertdebock/auditd)](https://galaxy.ansible.com/robertdebock/auditd)|[![Version](https://img.shields.io/github/release/robertdebock/ansible-role-auditd.svg)](https://github.com/robertdebock/ansible-role-auditd/releases/)|

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/robertdebock/ansible-role-auditd/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true

  roles:
    - role: ansible-role-auditd
      auditd_start_service: false
      auditd_local_events: "no"
      auditd_rules:
        - file: /var/log/audit/
          keyname: auditlog
        - file: /etc/audit/
          permissions:
            - write
            - attribute_change
          keyname: auditconfig
        - file: /etc/libaudit.conf
          permissions:
            - write
            - attribute_change
          keyname: auditconfig
        - file: /etc/audisp/
          permissions:
            - write
            - attribute_change
          keyname: audispconfig
        - file: /sbin/auditctl
          permissions:
            - execute
          keyname: audittools
        - file: /sbin/auditd
          permissions:
            - execute
          keyname: audittools
        - syscall: open
          action: always
          filter: exit
          filters:
            - auid!=4294967295
            - auid!=unset
          keyname: my_keyname
          arch: b32
        - syscall: adjtimex
          action: always
          filter: exit
          keyname: time_change
        - syscall: settimeofday
          action: always
          filter: exit
          keyname: time_change
        - action: always
          filter: exit
          filters:
            - path=/bin/ping
            - perm=x
            - auid>=500
            - auid!=4294967295
          keyname: privileged
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/robertdebock/ansible-role-auditd/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: true
  gather_facts: false

  roles:
    - role: robertdebock.bootstrap
```

Also see a [full explanation and example](https://robertdebock.nl/how-to-use-these-roles.html) on how to use these roles.

## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/robertdebock/ansible-role-auditd/blob/master/defaults/main.yml):

```yaml
---
# defaults file for auditd

# Below variables are docuemented in the man page for auditd.conf
# https://linux.die.net/man/5/auditd.conf
auditd_buffer_size: 32768
auditd_fail_mode: 1
auditd_maximum_rate: 60
auditd_enable_flag: 1
auditd_local_events: "yes"
auditd_write_logs: "yes"
auditd_log_file: /var/log/audit/audit.log
auditd_log_group: root
auditd_log_format: RAW
auditd_flush: incremental_async
auditd_freq: 50
auditd_max_log_file: 8
auditd_num_logs: 5
auditd_priority_boost: 4
auditd_disp_qos: lossy
auditd_dispatcher: /sbin/audispd
auditd_name_format: none
auditd_max_log_file_action: rotate
auditd_space_left: "75"  # This can be a number ('25') or a percentage. ('25%')
auditd_space_left_action: syslog
auditd_verify_email: "yes"
auditd_action_mail_acct: root
auditd_admin_space_left: 50
auditd_admin_space_left_action: suspend
auditd_disk_full_action: suspend
auditd_disk_error_action: suspend
auditd_use_libwrap: "yes"
auditd_tcp_listen_queue: 5
auditd_tcp_max_per_addr: 1
auditd_tcp_client_max_idle: 0
auditd_enable_krb5: "no"
auditd_krb5_principal: auditd
auditd_distribute_network: "no"

# You can opt to manage the rules with this role or not.
# Setting auditd_manage_rules to false will not manage the rules.
auditd_manage_rules: true

# Some rules require a specific architecture to be set.
auditd_default_arch: b64


# You can opt to start the auditd service or not.
# Mostly useful in CI, to avoid starting the service.
auditd_start_service: true

# This section includes configuration for the syslog plugin
# The role will loop through the dict and for each item it
# will create\update a property of the same name in the
# plugins.d/syslog.conf file
auditd_plugin_syslog:
  active: "yes"
  direction: out
  path: /sbin/audisp-syslog
  type: always

```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/robertdebock/ansible-role-auditd/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | GitLab |
|-------------|--------|--------|
|[robertdebock.bootstrap](https://galaxy.ansible.com/robertdebock/bootstrap)|[![Build Status GitHub](https://github.com/robertdebock/ansible-role-bootstrap/workflows/Ansible%20Molecule/badge.svg)](https://github.com/robertdebock/ansible-role-bootstrap/actions)|[![Build Status GitLab](https://gitlab.com/robertdebock-iac/ansible-role-bootstrap/badges/master/pipeline.svg)](https://gitlab.com/robertdebock-iac/ansible-role-bootstrap)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://robertdebock.nl/) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/robertdebock/ansible-role-auditd/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/robertdebock):

|container|tags|
|---------|----|
|[EL](https://hub.docker.com/r/robertdebock/enterpriselinux)|9|
|[Debian](https://hub.docker.com/r/robertdebock/debian)|all|
|[Fedora](https://hub.docker.com/r/robertdebock/fedora)|all|
|[Ubuntu](https://hub.docker.com/r/robertdebock/ubuntu)|all|

The minimum version of Ansible required is 2.12, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/robertdebock/ansible-role-auditd/issues).

## [License](#license)

[Apache-2.0](https://github.com/robertdebock/ansible-role-auditd/blob/master/LICENSE).

## [Author Information](#author-information)

[robertdebock](https://robertdebock.nl/)

Please consider [sponsoring me](https://github.com/sponsors/robertdebock).
