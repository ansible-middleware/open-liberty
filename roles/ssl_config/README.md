# middleware_automation.open_liberty.ssl_config

Configure TLS on an Open Liberty server. Writes
`configDropins/overrides/ssl_config.xml` with `<keyStore>`, optional truststore,
`<ssl>` and `<sslDefault>`, and copies keystore/truststore files into
`resources/security/`.

Reference: [ssl](https://openliberty.io/docs/latest/reference/config/ssl.html),
[keyStore](https://openliberty.io/docs/latest/reference/config/keyStore.html)

## Requirements

- Ansible >= 2.16
- Open Liberty installed and running (see [`install`](../install/README.md), [`server_config`](../server_config/README.md), [`systemd`](../systemd/README.md))
- `become: true`
- An existing keystore (PKCS12 recommended) and an encoded keystore password

## Security behaviour

- Only `TLSv1.2` and/or `TLSv1.3` are accepted for `openliberty_ssl_protocol`
  (single value or comma-separated list such as `TLSv1.3,TLSv1.2`).
- Passwords must be `{aes}`/`{hash}` encoded or a `${variable}` reference;
  plaintext is rejected unless `openliberty_ssl_require_encoded_passwords: false`.
  Encode with `<wlp>/bin/securityUtility encode --encoding=aes` and keep the
  value in Ansible Vault.
- Hostname verification is on by default.
- Client authentication (mutual TLS) requires a truststore password.
- Missing features are installed with `featureUtility` unless
  `openliberty_ssl_install_features` is `false`.

- Keystore files are written `0640` and owned by `openliberty_user`.
- Tasks that render passwords use `no_log`.
- If `openliberty_ssl_keystore_src` is empty and no keystore exists, Liberty
  generates a self-signed certificate. Use that for development only.

## Role Variables

| Variable | Description | Default |
|:---------|:------------|:--------|
| `openliberty_ssl_keystore_src` | Controller path of the keystore to copy (empty = do not copy) | `""` |
| `openliberty_ssl_keystore_filename` | File name under `resources/security/` | `key.p12` |
| `openliberty_ssl_keystore_type` | Keystore type | `PKCS12` |
| `openliberty_ssl_keystore_password` | **Required.** Encoded password | `""` |
| `openliberty_ssl_keystore_id` | `<keyStore>` id | `defaultKeyStore` |
| `openliberty_ssl_truststore_src` / `_filename` / `_type` / `_password` / `_id` | Truststore equivalents (truststore is emitted when src or password is set) | `""` / `trust.p12` / `PKCS12` / `""` / `defaultTrustStore` |
| `openliberty_ssl_config_id` | `<ssl>` id | `defaultSSLConfig` |
| `openliberty_ssl_protocol` | `TLSv1.3`, `TLSv1.2`, or `TLSv1.3,TLSv1.2` | `TLSv1.3,TLSv1.2` |
| `openliberty_ssl_enabled_ciphers` | JSSE cipher names; empty = defaults | `[]` |
| `openliberty_ssl_verify_hostname` | Verify peer host name | `true` |
| `openliberty_ssl_client_authentication` | Require client certs | `false` |
| `openliberty_ssl_client_authentication_supported` | Accept client certs | `false` |
| `openliberty_ssl_set_default` | Emit `<sslDefault>` | `true` |
| `openliberty_ssl_feature` | Feature providing SSL elements | `transportSecurity-1.0` |
| `openliberty_ssl_install_features` | Run `featureUtility installFeature` | `true` |
| `openliberty_ssl_require_encoded_passwords` | Reject plaintext passwords | `true` |
| `openliberty_home`, `openliberty_server_name`, `openliberty_user`, `openliberty_group`, `openliberty_service_name` | Same meaning as in other roles | see `defaults/main.yml` |

## Example Playbook

```yaml
- name: Configure TLS
  hosts: liberty_servers
  vars:
    openliberty_ssl_keystore_src: files/key.p12
    openliberty_ssl_keystore_password: "{{ vault_keystore_password }}"   # {aes}...
    openliberty_ssl_enabled_ciphers:
      - TLS_AES_256_GCM_SHA384
      - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
  roles:
    - role: middleware_automation.open_liberty.ssl_config
```

Pair with [`port_config`](../port_config/README.md) (`openliberty_https_enabled: true`)
to open the HTTPS listener.

## License

GPL-2.0-only
