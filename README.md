# dPanel Deploy

Ansible role for installing, upgrading, and uninstalling a dPanel-managed
systemd application service.

The role is intentionally focused on the runtime service layer. Build, artifact
creation, source checkout, and dependency installation are handled by other
dPanel roles and generated playbooks.

## Upgrade Behavior

Running the role again with `deploy_action: install` is treated as an upgrade or
redeploy:

- required service variables are validated before the unit is changed;
- the application work directory is ensured with the configured owner and group;
- the systemd unit is rendered to `deploy_systemd_unit_dir`;
- any failed systemd state is reset before restart;
- systemd is reloaded and the service is started/restarted;
- the role waits until `systemctl is-active` returns `active`.
- runtime stdout/stderr are sent to journald with a stable
  `SyslogIdentifier`.

This role does not implement release-directory rollback by itself. dPanel should
prepare the desired release or working tree before invoking this role, then pass
the final `deploy_workdir` and runtime command.

## Required Variables

For install:

```yaml
deploy_action: install
deploy_service_name: my-app
deploy_service_description: My App
deploy_user: dpanel
deploy_group: dpanel
deploy_workdir: /opt/dpanel/workspaces/personal/applications/123/current
deploy_start_command: ./scripts/start.sh
```

`deploy_workdir` must be an absolute path.

Runtime logs are not written to application-owned files by this role. The
systemd unit sends stdout and stderr to journald, which dPanel can collect and
forward to Uptrace.

For uninstall:

```yaml
deploy_action: uninstall
deploy_service_name: my-app
deploy_workdir: /opt/dpanel/workspaces/personal/applications/123
```

If `deploy_delete_workdir_on_uninstall` is false, `deploy_workdir` is not
required for uninstall.

## Systemd Controls

```yaml
deploy_systemd_unit_dir: /etc/systemd/system
deploy_systemd_service_state: restarted
deploy_systemd_restart: always
deploy_systemd_reset_failed_before_restart: true
deploy_systemd_standard_output: journal
deploy_systemd_standard_error: journal
deploy_systemd_syslog_identifier: "{{ deploy_service_name }}"
deploy_systemd_restart_startlimitburst: "20"
deploy_systemd_restart_startlimitinterval: "120"
deployment_systemd_custom_config: []
```

Use `deployment_systemd_custom_config` for extra systemd service directives.
Avoid overriding `StandardOutput`, `StandardError`, or `SyslogIdentifier` unless
you are intentionally replacing dPanel journald collection.

## Runtime Paths

The role prepends language runtime directories to `PATH` based on:

```yaml
deploy_service_language_name: go
deploy_service_language_version: 1.24.2
deploy_extra_languages:
  - name: nodejs
    version: 24.16.0
```

Supported language task files currently include `bun`, `docker`, `go`,
`nodejs`, `php`, `python`, and `ruby`.

## Example

```yaml
- hosts: all
  become: true
  roles:
    - role: dpanel-deploy
      vars:
        deploy_action: install
        deploy_service_name: dpanel-my-app
        deploy_service_description: My App
        deploy_user: dpanel
        deploy_group: dpanel
        deploy_workdir: /opt/dpanel/workspaces/personal/applications/123/current
        deploy_start_command: ./scripts/start.sh
        deploy_port: 9000
        deploy_service_language_name: go
        deploy_service_language_version: 1.24.2
        deploy_environment_variables:
          - key: APP_ENV
            value: production
```
