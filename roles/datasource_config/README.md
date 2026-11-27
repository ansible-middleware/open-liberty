# middleware_automation.open_liberty.datasource_config

Configure JDBC on Open Liberty. Writes
`configDropins/overrides/datasource_config.xml` with the `jdbc` feature,
`<library>`/`<jdbcDriver>`, `<authData>`, `<dataSource>` and
`<connectionManager>`, and places driver jars in `<server>/lib/jdbc/`.

This role is the structured replacement for the minimal `openliberty_datasources`
variable in [`server_config`](../server_config/README.md). It uses the separate
variable `openliberty_datasources_config` so both can coexist; do not define the
same data source `id` in both.

Reference: [JDBC connections](https://openliberty.io/docs/latest/relational-database-connections-JDBC.html),
[dataSource](https://openliberty.io/docs/latest/reference/config/dataSource.html),
[connectionManager](https://openliberty.io/docs/latest/reference/config/connectionManager.html)

## Requirements

- Ansible >= 2.16
- Open Liberty installed and running
- `become: true`
- JDBC driver jars, from the controller (`src`) or a URL (`url`)

## Security behaviour

- `authData` passwords must be `{aes}`/`{hash}` encoded or `${variable}`
  references; plaintext is rejected unless
  `openliberty_datasource_require_encoded_passwords: false`.
- Credentials live in `<authData>` and are referenced with `containerAuthDataRef`,
  not inlined in `<properties>`.
- Downloaded drivers can be pinned with `checksums`. Downloads use HTTPS with
  certificate validation and a 60s timeout.
- Jars and the XML are `0640`, owned by `openliberty_user`. Rendering uses `no_log`.
- When drivers, auth and data sources are all empty, the role is a no-op (and
  removes a previously managed `datasource_config.xml` override).
- Missing features are installed with `featureUtility` unless
  `openliberty_datasource_install_features` is `false`.


## Role Variables

| Variable | Description | Default |
|:---------|:------------|:--------|
| `openliberty_jdbc_drivers` | Drivers: `id`, `jars`, and `src` (controller dir) or `url` (base URL); optional `checksums`, `data_source` / `connection_pool_data_source` / `xa_data_source` (or `class`) | `[]` |
| `openliberty_datasource_auth` | `id`, `user`, `password` (encoded) | `[]` |
| `openliberty_datasources_config` | Data sources: `id`, `jndiName`, `driver`; optional `properties_element`, `properties`, `auth`, `type`, `isolationLevel`, `statementCacheSize`, `pool` | `[]` |
| `openliberty_jdbc_driver_dir` | Driver directory relative to the server dir | `lib/jdbc` |
| `openliberty_datasource_feature` | Feature to enable (`jdbc-4.2` for Java 8) | `jdbc-4.3` |
| `openliberty_datasource_install_features` | Run `featureUtility installFeature` | `true` |
| `openliberty_datasource_require_encoded_passwords` | Reject plaintext passwords | `true` |

`pool` keys map to `<connectionManager>` attributes (`maxPoolSize`,
`minPoolSize`, `connectionTimeout`, `maxIdleTime`, `agedTimeout`, `reapTime`,
`purgePolicy`).

## Example Playbook

```yaml
- name: Configure PostgreSQL data source
  hosts: liberty_servers
  vars:
    openliberty_jdbc_drivers:
      - id: postgresql
        jars: [postgresql-42.7.4.jar]
        url: https://repo1.maven.org/maven2/org/postgresql/postgresql/42.7.4
        checksums:
          postgresql-42.7.4.jar: "sha256:<hex>"
    openliberty_datasource_auth:
      - id: appDbAuth
        user: app
        password: "{{ vault_db_password }}"   # {aes}...
    openliberty_datasources_config:
      - id: appDB
        jndiName: jdbc/appDB
        driver: postgresql
        properties_element: properties.postgresql
        properties:
          serverName: db.internal
          portNumber: 5432
          databaseName: app
        auth: appDbAuth
        pool:
          maxPoolSize: 20
          minPoolSize: 2
  roles:
    - role: middleware_automation.open_liberty.datasource_config
```

## License

GPL-2.0-only
