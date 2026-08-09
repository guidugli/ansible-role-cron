[![CI](https://github.com/guidugli/ansible-role-cron/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-cron/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-cron?label=release)](https://github.com/guidugli/ansible-role-cron/tags)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-guidugli.cron-blue)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/cron/)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

## Ansible Role: cron

Install and configure cron on supported Linux distributions with a small, explicit, and testable interface.
The role installs the distribution cron package, manages the base cron environment, secures standard cron directories, controls `/etc/cron.allow`, removes `/etc/cron.deny`, and optionally manages user crontab entries.

### Requirements

- Ansible Core 2.16 or newer according to role metadata.
- Linux targets supported by the role metadata and Molecule platform matrix.
- Root-level permissions are required on real hosts because the role installs packages, manages `/etc/crontab`, writes `/etc/cron.allow`, removes `/etc/cron.deny`, and manages system cron directories. Supply privilege externally from the playbook, inventory, or automation platform.
- The `containers.podman` collection is required for Molecule container scenarios and is pinned in `requirements.yml` with a minimum version.

### Features

- Installs the correct cron package from internal distribution mappings.
- Manages base cron environment values for `SHELL`, `PATH`, and `MAILTO`.
- Ensures standard cron directories exist with secure ownership and permissions.
- Ensures `/etc/cron.allow` exists with secure permissions and contains the configured allow list.
- Ensures `/etc/cron.deny` is absent.
- Optionally creates or removes user crontab entries with `ansible.builtin.cron`.
- Uses role argument validation through `meta/argument_specs.yml` and semantic validation through `tasks/assert.yml`.
- Leaves privilege escalation to the execution context and uses tags on all role tasks.

### Supported platforms

The generated metadata currently lists Fedora, Ubuntu, and Debian. The bundled Molecule shared matrix includes Ubuntu 26.04 and 24.04, Debian 13 and 12, and Fedora 44 and 43.

### Variables

All public inputs are defined in `defaults/main.yml` and surfaced through `meta/argument_specs.yml`.

| Variable | Type | Default | Description |
| --- | --- | --- | --- |
| `cron_shell` | string | `/bin/bash` | Shell written to the cron configuration file as `SHELL=<value>`. |
| `cron_path` | string | `/sbin:/bin:/usr/sbin:/usr/bin` | PATH written to the cron configuration file as `PATH=<value>`. |
| `cron_mailto` | string | `root` | Mail recipient written to the cron configuration file as `MAILTO=<value>`. |
| `cron_allowed_users` | list of strings | `['root']` | Users that must be present in `/etc/cron.allow`. Duplicate values are rejected by validation. |
| `cron_jobs` | list of dictionaries | `[]` | Optional crontab entries to create or remove. |

#### `cron_jobs` item schema

Each item in `cron_jobs` supports the following keys.

| Key | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | yes | Unique Ansible cron entry name. |
| `job` | string | required when `state` is `present` | Command written to the crontab entry. |
| `state` | string | no | `present` or `absent`. Omitted values are treated as `present`. |
| `minute` | raw | no | Minute field passed to `ansible.builtin.cron`. |
| `hour` | raw | no | Hour field passed to `ansible.builtin.cron`. |
| `day` | raw | no | Day-of-month field passed to `ansible.builtin.cron`. |
| `month` | raw | no | Month field passed to `ansible.builtin.cron`. |
| `weekday` | raw | no | Day-of-week field passed to `ansible.builtin.cron`. |
| `user` | string | no | User that owns the crontab entry. If omitted, Ansible uses the module default for the target context. |

### Example playbook

#### Minimal usage

```yaml
---
- name: Configure cron
  hosts: all
  become: true
  roles:
    - role: guidugli.cron
```

#### Manage allowed users and cron jobs

```yaml
---
- name: Configure cron with managed jobs
  hosts: all
  become: true
  roles:
    - role: guidugli.cron
      vars:
        cron_shell: /bin/bash
        cron_path: /sbin:/bin:/usr/sbin:/usr/bin
        cron_mailto: root
        cron_allowed_users:
          - root
          - opsuser
        cron_jobs:
          - name: cleanup tmp
            minute: '15'
            hour: '2'
            job: /usr/local/bin/cleanup-tmp.sh
            user: root
          - name: remove legacy entry
            state: absent
            user: root
```

### Molecule testing instructions

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

A convenience script is also present.

```bash
./scripts/run_local.sh
```

### Execution notes

- **Privilege model:** the role never declares `become`, `become_user`, or `become_method`. Use `become: true` at the play, inventory, or automation-controller level for real hosts where package installation, `/etc` changes, service management, and crontab ownership require elevated privileges.
- **Container behavior:** Molecule containers generally execute as root and use `become: false` in shared converge logic. Role tasks do not assume privilege escalation inside the role.
- **Systemd behavior:** service management and service verification run only when `ansible_facts['service_mgr'] == 'systemd'` and a cron service name is configured. Non-systemd containers still receive package, file, cron.allow, and optional crontab configuration.
- **Idempotency:** file, line, package, service, and cron modules are used for deterministic changes. Commands in Molecule verification use `changed_when: false`.
- **Metadata generation:** generated metadata is controlled by `templates/meta_main.yml.j2` and the release metadata scripts. Do not edit generated `meta/main.yml` directly.

### Release workflow

Generated repository metadata is refreshed through the shared generator scripts.

```bash
./scripts/update_release_metadata.sh
./scripts/release.sh --version v1.2.0 --message "Release v1.2.0"
```

### License

MIT

### Author

Carlos Guidugli
