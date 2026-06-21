[![CI](https://github.com/guidugli/ansible-role-journald/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-journald/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-journald?sort=semver)](https://github.com/guidugli/ansible-role-journald/tags)
[![Galaxy](https://img.shields.io/badge/ansible--galaxy-guidugli.journald-blue.svg)](https://galaxy.ansible.com/ui/standalone/namespaces/2679/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/guidugli/ansible-role-journald/blob/main/LICENSE)

# Ansible Role: journald

Install and configure `systemd-journald` on Linux hosts, including optional remote upload support and defaults aligned with CIS-oriented journald sizing and forwarding guidance.

## Requirements

- No runtime role dependencies are required.
- For local validation and Molecule testing, install the development dependencies from `requirements-dev.txt` and the collections from `requirements.yml`.
- The current Molecule matrix in this role covers Ubuntu `26.04` and `24.04`, Debian `13` and `12`, and Fedora `44` and `43`.

## Variables

The role exposes the following primary input variables in `defaults/main.yml`:

- `journald_compress: true`
- `journald_system_max_use: 1G`
- `journald_system_keep_free: 500M`
- `journald_runtime_max_use: 200M`
- `journald_runtime_keep_free: 50M`
- `journal_max_file_sec: 1month`
- `journald_forward_to_syslog: false`
- `journald_enable_remote_logging: false`
- `journald_remote_url: 192.168.1.1`
- `journald_remote_server_key_file: /etc/ssl/private/journal-upload.pem`
- `journald_remote_server_cert_file: /etc/ssl/certs/journal-upload.pem`
- `journald_trusted_cert_file: /etc/ssl/ca/trusted.pem`
- `journald_disable_rsyslog: true`

The role also uses targeted system variables from `vars/main.yml`:

- `rsyslog_conf_path: /etc/rsyslog.conf`
- `rsyslogd_path: /etc/rsyslog.d`
- `journald_conf_path: /etc/systemd/journald.conf`
- `journald_confd_directory: /etc/systemd/journald.conf.d`
- `journald_confd_filename: 01-ansible.conf`
- `journald_storage: persistent`
- `journald_remote_pkg: systemd-journal-remote`
- `journald_upload_service: systemd-journal-upload.service`
- `journald_upload_confd_path: /etc/systemd/journal-upload.conf.d`
- `journald_upload_conf_file: 01-ansible.conf`
- `journald_remote_socket: systemd-journal-remote.socket`
- `journald_remote_service: systemd-journal-remote.service`

Additional optional journald settings are already scaffolded in `defaults/main.yml` as commented variables for rate limits, retention, log levels, audit, and other native `journald.conf` settings.

## Example playbook

```yaml
---
- name: Configure journald
  hosts: servers
  become: true
  roles:
    - role: guidugli.journald
      vars:
        journald_enable_remote_logging: false
        journald_disable_rsyslog: true
```

## Molecule testing

Use the shared Molecule scenarios that are already present in the role:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

The `default` scenario exercises the shared converge and verify playbooks against the standard Podman container matrix. The `systemd` scenario prepares containers with `systemd`, commits them into local images, and recreates them with a detected systemd entrypoint so service-management tasks can be exercised in a systemd-capable environment.

## Execution notes

- **Privilege model:** this role does not enforce privilege escalation internally. Run the role with external privilege (for example `become: true` in real host playbooks) because it writes configuration under `/etc/systemd/`, can install `systemd-journal-remote`, manages `systemd-journald.service` and `systemd-journal-upload.service`, and can stop or mask `rsyslog.socket` and `rsyslog.service`.
- **Container and systemd behavior:** the role manages systemd services and systemd-owned configuration paths. The dedicated `molecule/systemd` scenario exists specifically for that execution context. On non-systemd targets or minimal containers without a service manager, service-management tasks should be expected to require environment-specific handling.
