Ansible Role: journald
=========

An Ansible Role that install and configure journald Fedora and Debian/Ubuntu following CIS recommendation (default variable values). 

Requirements
------------

No requirements.

Role Variables
--------------

**Available variables are listed below, along with default values (see defaults/main.yml):**

    journald_compress: yes

Specify if journald should compress large files.

    journald_system_max_use: 1G
    journald_system_keep_free:  500M
    #journald_system_max_file_size: 100M
    #journald_system_max_files: 100
    journald_runtime_max_use: 200M
    journald_runtime_keep_free: 50M
    #journald_runtime_max_file_size: 50M
    #journald_runtime_max_files: 100
    journal_max_file_sec: 1month

These variables enforce size limits on the journal files stored.

    journald_forward_to_syslog: false

Enable or disable sending events to syslog. Recommended to use only journald or syslog, not both.

    journald_enable_remote_logging: false
    #journald_remote_url: 192.168.x.x
    journald_remote_server_key_file: /etc/ssl/private/journal-upload.pem
    journald_remote_server_cert_file: /etc/ssl/certs/journal-upload.pem
    journald_trusted_cert_file: /etc/ssl/ca/trusted.pem

Configure remote logging. Recommended to enable remote logging. It is disabled by default because it needs remote
host and certs configured.

    journald_disable_rsyslog: true

This will disable syslog on the system.

    #journald_seal: true
    #journald_split_mode: uid
    #journald_sync_interval_sec: 5m
    #journald_rate_limit_interval_sec: 30s
    #journald_rate_limit_burst: 10000
    #journald_max_retention_sec: 12month
    #journald_sync_interval_sec: 5m
    #journald_forward_to_console: false
    #journald_forward_to_kmsg: false
    #journald_forware_to_wall: true
    #journald_max_level_store: debug
    #journald_max_level_syslog: debug
    #journald_max_level_kmsg: notce
    #journald_max_level_console: info
    #journald_max_level_wall: emerg
    #journald_read_kmsg: true
    #journald_audit: true
    #journald_tty_path: /dev/console
    #journald_line_max: 48K
 
Miscelaneous options. Check journald.conf man page for more details.

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    rsyslog_conf_path: /etc/rsyslog.conf

rsyslog configuration path.

    rsyslogd_path: /etc/rsyslog.d

rsyslog configuration directory.

    journald_conf_path: /etc/systemd/journald.conf

journald configuration path.

    journald_confd_directory: /etc/systemd/journald.conf.d

journald.conf.d path.

    journald_confd_filename: 01-ansible.conf

Name of the configuration file to be created under journald.conf.d.

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
