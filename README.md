Ansible Role: journald
=========

An Ansible Role that install and configure journald on RHEL/CentOS, Fedora and Debian/Ubuntu.

Requirements
------------

No requirements.

Role Variables
--------------

**Available variables are listed below, along with default values (see defaults/main.yml):**

    journald_compress: yes

Specify if journald should compress large files.

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    rsyslog_conf_path: /etc/rsyslog.conf

rsyslog configuration path.

    rsyslogd_path: /etc/rsyslog.d

rsyslog configuration directory.

    journald_conf_path: /etc/systemd/journald.conf

journald configuration path.

Dependencies
------------

No dependencies.

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: guidugli.journald }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
