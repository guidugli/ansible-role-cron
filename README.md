# Ansible Role: cron

[![CI](https://github.com/guidugli/ansible-role-cron/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-cron/actions/workflows/CI.yml)
[![Release](https://github.com/guidugli/ansible-role-cron/actions/workflows/release.yml/badge.svg)](https://github.com/guidugli/ansible-role-cron/actions/workflows/release.yml)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.cron-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/cron/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

Install and configure cron on supported Linux distributions with a small, explicit, and testable interface.

## Overview

This role installs the appropriate cron package for the target distribution, manages the base cron environment in the system configuration file, ensures secure cron directory permissions, manages `cron.allow`, and optionally creates or removes cron jobs.

## Features

- Explicit public defaults in `defaults/main.yml`
- Automatic argument validation via `meta/argument_specs.yml`
- Additional semantic validation in `tasks/assert.yml`
- Clean task flow in `tasks/main.yml`
- Distribution-specific package, service, and configuration mapping in `vars/main.yml`
- Molecule-friendly verification playbook in `molecule/shared/verify.yml`
- Template-first role metadata with `templates/meta_main.yml.j2`

## Supported platforms

The generated metadata currently targets:

- Debian 12 (bookworm)
- Debian 13 (trixie)
- Ubuntu 22.04 (jammy)
- Ubuntu 24.04 (noble)
- Fedora 42
- Fedora 43

## Role variables

Public variables are defined in `defaults/main.yml`.

```yaml
cron_shell: /bin/bash
cron_path: /sbin:/bin:/usr/sbin:/usr/bin
cron_mailto: root
cron_allowed_users:
  - root
cron_jobs: []
```

### `cron_jobs` structure

Each entry in `cron_jobs` supports:

- `name` (required)
- `job` (required when `state: present`)
- `state` (`present` or `absent`, default `present`)
- `minute`
- `hour`
- `day`
- `month`
- `weekday`
- `user`

Example:

```yaml
cron_jobs:
  - name: rotate logs
    minute: '0'
    hour: '3'
    job: /usr/local/bin/rotate-logs.sh
    user: root
```

## Important behavior

### Package, configuration, and service mapping

The role keeps OS-specific mappings in `vars/main.yml`:

- `cron_packages`
- `cron_configuration`
- `cron_service`

These are resolved from internal lookup maps so callers do not need to set them directly.

### `cron.allow`

The role ensures:

- `/etc/cron.deny` is absent
- `/etc/cron.allow` exists with mode `0640`
- each user listed in `cron_allowed_users` is present in `/etc/cron.allow`

### Privilege escalation

This role does not force `become` inside tasks or handlers. In normal usage, call the role from a play with `become: true`.

## How it works

1. Ansible automatically validates role inputs using `meta/argument_specs.yml`.
2. `tasks/assert.yml` performs semantic checks that are clearer to express with asserts.
3. The role installs cron packages.
4. The role manages the base cron configuration (`SHELL`, `PATH`, `MAILTO`).
5. The role enforces secure cron directory and allow-file state.
6. Optional cron jobs are managed with `ansible.builtin.cron`.

## Usage

### Minimal usage

```yaml
- name: Configure cron
  hosts: all
  become: true
  roles:
    - role: guidugli.cron
```

### Manage allowed users and jobs

```yaml
- name: Configure cron with managed entries
  hosts: all
  become: true
  roles:
    - role: guidugli.cron
      vars:
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

## Molecule testing

The repository already contains Molecule scenario directories. This modernization adds `molecule/shared/verify.yml` with real assertions for:

- package installation
- base cron configuration
- cron directory permissions
- `cron.allow` and `cron.deny` state
- service enablement and activity on systemd targets
- optional managed cron jobs

If you later wire scenarios to shared playbooks, this file is ready to reuse.

## Metadata and release notes

Role metadata is maintained template-first:

- source: `templates/meta_main.yml.j2`
- generated artifact: `meta/main.yml`

If you already have a repository-level generator flow, keep the template as the source of truth and regenerate `meta/main.yml` from it.

## Repository structure

```text
defaults/
handlers/
meta/
molecule/
  shared/
tasks/
templates/
vars/
```

## Design notes

- Public inputs live in `defaults/main.yml`.
- Semantic validation is separated from the execution path.
- The role leaves privilege escalation to the calling play.
- Internal OS-specific mappings remain in `vars/main.yml`.
- The metadata template and generated metadata are intentionally aligned.

## License

MIT

## Author

Carlos Guidugli
