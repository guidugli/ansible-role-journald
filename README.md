[![CI](https://github.com/guidugli/ansible-role-journald/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-journald/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-journald?label=release)](https://github.com/guidugli/ansible-role-journald/tags)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.journald-blue)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/journald/)
[![License](https://img.shields.io/github/license/guidugli/ansible-role-journald)](LICENSE)

# Ansible Role: journald

Install and configure systemd-journald on Linux hosts. The role renders a managed `journald.conf.d` drop-in, can configure `systemd-journal-upload`, masks receiver-side `systemd-journal-remote` units, and can optionally manage journald tmpfiles permissions.

## Requirements

- Ansible Core 2.14 or newer according to the role metadata.
- Linux targets using systemd for service-management tasks.
- External privilege escalation for real hosts because the role writes under `/etc/systemd`, can write `/etc/tmpfiles.d`, installs packages, and manages system services.
- `containers.podman` collection 1.10.0 or newer for Molecule container scenarios, as declared in `requirements.yml`.
- Development dependencies from `requirements-dev.txt` for local linting and Molecule validation.

## Features

- Renders `/etc/systemd/journald.conf.d/01-ansible.conf` from role variables.
- Supports persistent journald storage and configurable compression, retention, and disk usage limits.
- Optionally installs and configures `systemd-journal-upload` for remote journal forwarding.
- Stops, disables, and masks `systemd-journal-remote.socket` and `systemd-journal-remote.service` when remote upload is used.
- Optionally stops, disables, and masks rsyslog units to prevent duplicate event collection.
- Optionally manages a systemd tmpfiles override for journald log file permissions.
- Uses systemd guards for service operations so non-systemd or minimal container contexts can skip service management safely.
- Includes shared Molecule converge and verify logic used by both default and systemd scenarios.

## Supported platforms

The generated role metadata lists Fedora, Ubuntu, and Debian. The bundled Molecule shared matrix currently covers Ubuntu 26.04 and 24.04, Debian 13 and 12, and Fedora 44 and 43.

## Variables

All public defaults are defined in `defaults/main.yml` and mirrored in `meta/argument_specs.yml`.

| Variable | Type | Default | Description |
| --- | --- | --- | --- |
| `journald_compress` | bool | `true` | Enables compression for large journal files. |
| `journald_system_max_use` | string | `1G` | Maximum disk space persistent journals may use. |
| `journald_system_keep_free` | string | `500M` | Disk space journald should leave free for persistent storage. |
| `journald_runtime_max_use` | string | `200M` | Maximum disk space runtime journals may use. |
| `journald_runtime_keep_free` | string | `50M` | Disk space journald should leave free for runtime storage. |
| `journal_max_file_sec` | string | `1month` | Maximum time range stored in a single journal file. |
| `journald_forward_to_syslog` | bool | `false` | Controls `ForwardToSyslog` in the rendered journald drop-in. |
| `journald_enable_remote_logging` | bool | `false` | Enables package installation and configuration for `systemd-journal-upload`. |
| `journald_remote_url` | string | `192.168.1.1` | Remote journald upload destination rendered as `URL`. |
| `journald_remote_server_key_file` | path | `/etc/ssl/private/journal-upload.pem` | Private key file path for `systemd-journal-upload`. |
| `journald_remote_server_cert_file` | path | `/etc/ssl/certs/journal-upload.pem` | Certificate file path for `systemd-journal-upload`. |
| `journald_trusted_cert_file` | path | `/etc/ssl/ca/trusted.pem` | Trusted CA certificate path for `systemd-journal-upload`. |
| `journald_disable_rsyslog` | bool | `true` | Stops, disables, and masks rsyslog service/socket units when available on systemd hosts. |
| `journald_manage_tmpfiles_permissions` | bool | `false` | Enables optional management of a systemd tmpfiles override for journald log file permissions. |
| `journald_tmpfiles_conf_path` | path | `/etc/tmpfiles.d/systemd.conf` | Override file path used when tmpfiles permission management is enabled. |
| `journald_tmpfiles_source_paths` | list | `['/usr/lib/tmpfiles.d/systemd.conf', '/lib/tmpfiles.d/systemd.conf']` | Candidate distribution tmpfiles defaults used to seed the managed override. |
| `journald_tmpfiles_file_mode` | string | `0640` | Mode applied to type `f` tmpfiles entries in the managed override file. |
| `journald_tmpfiles_require_source` | bool | `false` | Fail when tmpfiles permission management is enabled but none of the configured source files exists on the target. |

Internal role variables in `vars/main.yml` define file paths, unit names, package names, and storage mode used by the tasks and templates. Override these only when adapting the role to a distribution-specific package or path layout.

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

Remote upload example:

```yaml
---
- name: Configure journald remote upload
  hosts: servers
  become: true
  roles:
    - role: guidugli.journald
      vars:
        journald_enable_remote_logging: true
        journald_remote_url: https://logs.example.org:19532
        journald_remote_server_key_file: /etc/ssl/private/journal-upload.pem
        journald_remote_server_cert_file: /etc/ssl/certs/journal-upload.pem
        journald_trusted_cert_file: /etc/ssl/ca/trusted.pem
```

Tmpfiles permission management example:

```yaml
---
- name: Configure journald tmpfiles permissions
  hosts: servers
  become: true
  roles:
    - role: guidugli.journald
      vars:
        journald_manage_tmpfiles_permissions: true
        journald_tmpfiles_file_mode: "0640"
```

## Molecule testing instructions

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

The default scenario exercises shared converge and verify playbooks against the Podman container matrix. The systemd scenario prepares systemd-capable containers and runs the same shared converge and verify logic in that execution context.

## Execution notes

- **Privilege model:** the role never declares `become`, `become_user`, or `become_method`. Use `become: true` externally for real hosts because the role writes `/etc/systemd` and `/etc/tmpfiles.d` configuration, installs packages, and manages system services.
- **Container behavior:** Molecule scenario playbooks run with `become: false`; containers are expected to provide the required execution privileges externally.
- **Systemd behavior:** service-management tasks and handlers are guarded with `ansible_facts['service_mgr'] == 'systemd'`. Configuration files are still rendered for targets where the paths are valid, but service starts, restarts, and rsyslog masking are skipped outside systemd.
- **Remote upload behavior:** `systemd-journal-upload` is a systemd unit, so the role skips remote-upload tasks on non-systemd targets even when `journald_enable_remote_logging` is true. Molecule enables remote-upload coverage only for systemd-capable Fedora containers where the package is available from the base repositories.
- **Package cache behavior:** Debian and Ubuntu Molecule images clean apt lists during scenario preparation, so the role refreshes the apt cache before installing `systemd-journal-remote` when `ansible_facts['os_family'] == 'Debian'`.
- **Tmpfiles behavior:** tmpfiles permission management is disabled by default because CIS treats journald log file access as a manual, site-policy-dependent control. When enabled, the role looks for a systemd tmpfiles defaults file, seeds `/etc/tmpfiles.d/systemd.conf`, and normalizes type `f` entries to `journald_tmpfiles_file_mode`. Minimal/container images may not include a tmpfiles defaults file; in that case the role skips without change unless `journald_tmpfiles_require_source` is true.
- **Idempotency:** configuration uses Ansible modules, deterministic templates, package `state: present`, and handlers so repeat runs should report no changes after convergence.

## Release workflow

Generated repository metadata is refreshed through the shared generator scripts. Use the release helper after setting the desired semantic version tag:

```bash
./scripts/update_release_metadata.sh
./scripts/release.sh --version v1.2.0 --message "Release v1.2.0"
```

## License

MIT

## Author

Carlos Guidugli
