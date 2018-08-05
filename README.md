# Letsencrypt-dns
Use `nsupdate` with GSSAPI and machine keytabs and [dehydrated](https://github.com/lukas2511/dehydrated) to automatically obtain (wildcard) certificates.

## Assumptions
This role assumes that each host has a principal `host/fqdn@REALM` in `/etc/krb5.keytab`. It also assumes that each host is allowed to set TXT Records at `_acme-challenge.<domain>`. After applying the role to a new machine, you'll need to manually accept the ToS and run `/opt/le-dns/dehydrated --register --accept-terms`.

## Configuration
| Variable | Default value | Description |
| -------- | ------------- | ----------- |
| `ledns_tlsa_enabled` | `True` | Whether to enable `TLSA` support |
| `ledns_tlsa_services` | ['_443._tcp', '_25._tcp', '_465._tcp', '_587._tcp', '_143._tcp', '_993._tcp'] | Which services to set `TLSA` for |
| `ledns_base` | `None` | First SAN of the certificate to request. You need to set this! |
| `ledns_check_server` | `None` | The nameserver to poll after creating a DNS entry. You need to set this! |
| `ledns_domains` | `{{ ledns_base }} *.{{ ledns_base }}` | All SANs to request a certificate for |
| `ledns_ca` | `https://acme-staging-v02.api.letsencrypt.org/directory` | API endpoint of the CA to use. This defaults to Letsencrypt staging by design! |
| `ledns_keysize` | `384` | Keysize of the certificate |
| `ledns_keytype` | `secp384r1` | Keytype of the certificate |
| `ledns_self` |`{{ ansible_fqdn }}` | List of hostnames for this system under which TLSA records should be created |
| `ledns_timing` | `0,8,16:00:00` | When should the cron job be executed? |
| `ledns_krb_realm` | `None` | Kerberos realm to use |
| `ledns_krb_apkinit` | `True` | Enable anonymous PKINIT for FAST? |
| `ledns_servers` | `False` | List of servers the take nsupdates. A random one will be chosen at templating time |
| `ledns_cert_targets` | `False` | A dict with installation instructions for the certificates, see below. |

### `ledns_cert_targets`
This variable contains a dict which maps domains (as in `ledns_base`) to installation instructions. Example:
```
ledns_cert_targets:
  example.org:
    - name: dovecot
      key: /etc/dovecot/server.key
      chain: /etc/dovecot/server.pem
      owner: root
      group: root
      mode: 640
      exec: systemctl restart dovecot
```
would copy the full certificate chain and it's key and then restart dovecot. Alternatively, `full` can be specified to obtain a file with the certificate followed by it's key followed by the full certificate chain.
