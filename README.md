Ansible Role: cron
=========

An Ansible Role that install and configure cron on RHEL/CentOS, Fedora and Debian/Ubuntu.

Requirements
------------

No requirements.

Role Variables
--------------

**Available variables are listed below, along with default values (see defaults/main.yml):**

    cron_shell: /bin/bash

The shell to use for running cronjobs.

    cron_path: /sbin:/bin:/usr/sbin:/usr/bin

The path to set for running jobs.

    cron_mailto: root

The address where mails should be sent to.

    cron_allowed_users: ['myuser', 'root']

Users that can use cron.

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    cron_packages:

Packages to install cron.

    cron_configuration:

Configuration path.

    cron_service:

Cron service name.

Dependencies
------------

No dependencies.

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: guidugli.cron }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
