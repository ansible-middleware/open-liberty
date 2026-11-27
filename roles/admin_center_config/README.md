# middleware_automation.open_liberty.admin_center_config

Enable the Open Liberty [Admin Center](https://openliberty.io/docs/latest/admin-center.html).
Writes `configDropins/overrides/admin_center_config.xml` with the `adminCenter`
feature, `<administrator-role>`, optional `<reader-role>` and optional
`<remoteFileAccess>`.

Admin Center is available at `https://<host>:<https_port>/adminCenter/`.

## Requirements

- Ansible >= 2.16
- Open Liberty installed and running
- `become: true`
- HTTPS enabled: [`port_config`](../port_config/README.md) with `openliberty_https_enabled: true`, plus [`ssl_config`](../ssl_config/README.md)
- A user registry containing the mapped users/groups: [`security_config`](../security_config/README.md)

## Security behaviour

- The role fails unless at least one administrator user or group is set, so the
  console is never left without a role mapping.
- The role fails unless `openliberty_https_enabled` is `true` (override with
  `openliberty_admin_center_require_https: false` only if HTTPS is configured outside Ansible).
- `remoteFileAccess` directories are not opened unless you list them.
- `adminCenter-1.0` is not part of the Web Profile runtime; the role installs it
  with `featureUtility` unless `openliberty_admin_center_install_features` is `false`.


## Role Variables

| Variable | Description | Default |
|:---------|:------------|:--------|
| `openliberty_admin_center_administrator_users` / `_groups` | Mapped to `administrator-role` (at least one required) | `[]` |
| `openliberty_admin_center_reader_users` / `_groups` | Mapped to `reader-role` | `[]` |
| `openliberty_admin_center_read_dirs` / `_write_dirs` | `remoteFileAccess` directories | `[]` |
| `openliberty_admin_center_extra_features` | e.g. `["monitor-1.0"]` | `[]` |
| `openliberty_admin_center_feature` | Admin Center feature | `adminCenter-1.0` |
| `openliberty_admin_center_install_features` | Run `featureUtility installFeature` | `true` |
| `openliberty_admin_center_require_https` | Fail when HTTPS is not enabled | `true` |
| `openliberty_https_enabled` / `openliberty_https_port` | Same variables as `port_config` | `false` / `9443` |

## Example Playbook

```yaml
- name: Enable Admin Center
  hosts: liberty_servers
  vars:
    openliberty_https_enabled: true
    openliberty_admin_center_administrator_users: [admin]
    openliberty_admin_center_reader_groups: [operators]
    openliberty_admin_center_extra_features: [monitor-1.0]
  roles:
    - role: middleware_automation.open_liberty.admin_center_config
```

For the full stack see [`playbooks/configure_server.yml`](../../playbooks/configure_server.yml).

## License

GPL-2.0-only
