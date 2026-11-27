# middleware_automation.open_liberty.security_config

Configure Open Liberty application security. Writes
`configDropins/overrides/security_config.xml` with the `appSecurity` feature, a
basic or LDAP user registry, `<webAppSecurity>`, `<httpSession>` cookie
hardening and `<ltpa>`.

Reference: [Security overview](https://openliberty.io/docs/latest/security-overview.html),
[basicRegistry](https://openliberty.io/docs/latest/reference/config/basicRegistry.html),
[ldapRegistry](https://openliberty.io/docs/latest/reference/config/ldapRegistry.html),
[webAppSecurity](https://openliberty.io/docs/latest/reference/config/webAppSecurity.html)

Administrator/reader role mapping is handled by
[`admin_center_config`](../admin_center_config/README.md).

## Requirements

- Ansible >= 2.16
- Open Liberty installed and running
- `become: true`
- For LDAP over TLS, run [`ssl_config`](../ssl_config/README.md) first (or set `openliberty_security_ldap_ssl_ref`)

## Security behaviour

- Passwords (users, LDAP bind, LTPA keys) must be `{aes}`/`{hash}` encoded or
  `${variable}` references; plaintext is rejected unless
  `openliberty_security_require_encoded_passwords: false`.
- Secure defaults: `httpOnlyCookies`, `ssoRequiresSSL`, session `cookieSecure`,
  `cookieHttpOnly`, `SameSite=Lax`, basic-auth failover off, realm name hidden.
  `ssoRequiresSSL` and `cookieSecure` require HTTPS, so set them to `false`
  only on HTTP-only development servers.
- Hardening defaults from the Open Liberty hardening guide:
  `removeServerHeader` and `disableXPoweredBy`.
- `SameSite=None` is rejected unless `cookieSecure` is true.
- LDAP connections use TLS by default.
- Missing features are installed with `featureUtility` unless
  `openliberty_security_install_features` is `false`.
- Rendering tasks use `no_log`.


## Role Variables

| Variable | Description | Default |
|:---------|:------------|:--------|
| `openliberty_security_registry_type` | `basic`, `ldap` or `none` | `basic` |
| `openliberty_security_users` | Basic registry users: `name`, `password` (encoded) | `[]` |
| `openliberty_security_groups` | Basic registry groups: `name`, `members` | `[]` |
| `openliberty_security_realm` / `_registry_id` | Basic registry realm / id | `libertyRealm` / `basic` |
| `openliberty_security_ldap_host`, `_port`, `_base_dn`, `_bind_dn`, `_bind_password`, `_type`, `_realm`, `_id` | LDAP registry settings (`host` and `base_dn` required for `ldap`) | port `636` |
| `openliberty_security_ldap_ssl_enabled` / `_ssl_ref` | TLS to the directory | `true` / `""` |
| `openliberty_security_ldap_search_timeout` | LDAP search timeout | `60s` |
| `openliberty_security_http_only_cookies` | `webAppSecurity` httpOnly | `true` |
| `openliberty_security_sso_requires_ssl` | SSO cookie only over HTTPS | `true` |
| `openliberty_security_allow_failover_to_basic_auth` | Fail over to basic auth | `false` |
| `openliberty_security_display_authentication_realm` | Show realm in login prompt | `false` |
| `openliberty_security_session_cookie_secure` / `_http_only` | Session cookie flags | `true` / `true` |
| `openliberty_security_session_cookie_same_site` | `Lax`, `Strict`, `None`, `Disabled` | `Lax` |
| `openliberty_security_ltpa_expiration` | LTPA token lifetime | `120m` |
| `openliberty_security_ltpa_keys_password` | Encoded LTPA keys password (optional) | `""` |
| `openliberty_security_feature` / `_ldap_feature` | Features to enable | `appSecurity-5.0` / `ldapRegistry-3.0` |
| `openliberty_security_install_features` | Run `featureUtility installFeature` | `true` |
| `openliberty_security_remove_server_header` | Emit `<httpOptions removeServerHeader>` | `true` |
| `openliberty_security_disable_x_powered_by` | Emit `<webContainer disableXPoweredBy>` | `true` |
| `openliberty_security_require_encoded_passwords` | Reject plaintext passwords | `true` |

## Example Playbook

```yaml
- name: Configure security
  hosts: liberty_servers
  vars:
    openliberty_security_users:
      - name: admin
        password: "{{ vault_admin_password }}"   # {aes}...
    openliberty_security_groups:
      - name: admins
        members: [admin]
  roles:
    - role: middleware_automation.open_liberty.security_config
```

LDAP:

```yaml
    openliberty_security_registry_type: ldap
    openliberty_security_ldap_host: ldap.example.com
    openliberty_security_ldap_base_dn: "dc=example,dc=com"
    openliberty_security_ldap_bind_dn: "cn=svc,dc=example,dc=com"
    openliberty_security_ldap_bind_password: "{{ vault_ldap_bind_password }}"
```

## License

GPL-2.0-only
