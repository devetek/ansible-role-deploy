Ansible role: Deploy
=========

This Ansible role automates the deployment of an application across environments (development, staging, production). It supports flexible configuration, zero-downtime deployment strategies, environment-specific variables, and integration with common service managers (systemd, Docker, etc.).

Requirements
------------

None.

Role Variables
--------------

List of variables in ansible-role-build:

```sh
---

# Action Type 
deploy_action: "install"

# App Meta
deploy_service_name: ""
deploy_service_description: ""
deploy_service_document: https://cloud.terpusat.com/

# App Runtime Common Env
deploy_env_path: "{{ ansible_env.PATH }}"
deploy_service_language_name: ""
deploy_service_language_version: ""

# RBAC
deploy_user: root
deploy_group: root

# App Directory and Runtime
deploy_start_command: ""
deploy_reload_command: ""
deploy_workdir: ""
deploy_environment_variables: []

# App Log Forwarder
deploy_logdir: "/var/log"

# App Advance Configuration
deploy_systemd_restart: always # always, on-success, on-failure, on-abnormal, on-abort, on-watchdog
deploy_systemd_network_dependency: true
deploy_systemd_restart_startlimitburst: "20"
deploy_systemd_restart_startlimitinterval: "120"
deployment_systemd_custom_config: [] # systemd extra config for your serivice if needed
```

Dependencies
------------

None.

Example Playbook
----------------

```yaml
- hosts: servers
  roles:
    - devetek.deploy
```

License
-------

GNU General Public License v3.0 or later

Author Information
------------------

[Nedya Prakasa]. Role created for [dPanel].

[dPanel]: https://cloud.terpusat.com/
[Nedya Prakasa]: https://github.com/prakasa1904
[devetek]: https://github.com/devetek
[geerlingguy.php]: https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/php/
[geerlingguy.composer]: https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/composer/