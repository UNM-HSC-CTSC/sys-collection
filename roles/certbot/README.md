# ctsc.sys.certbot

Role for configuring certbot on a Linux server using an ACME account.

Installs `certbot` (apt), registers the ACME account using External Account
Binding (EAB) credentials, requests a certificate for
`certbot_primary_domain` + `certbot_subject_alternative_names`, and makes sure
`certbot.timer` is enabled for automatic renewal. The account's domains are
pre-validated (e.g. CertiNEXT), so no HTTP/DNS challenge is actually solved.

Re-running the role is a no-op while the certificate isn't due for renewal.
Changing the SAN list re-issues the certificate with the new names. The
certificate lands in `/etc/letsencrypt/live/<certbot_primary_domain>/`.

See `defaults/main.yml` for all variables. `certbot_email`, `certbot_eab_kid`,
`certbot_eab_hmac_key` and `certbot_primary_domain` are required; keep the EAB
credentials in vault.

Use `certbot_deploy_hooks` to run scripts after each successful renewal, e.g.
to reload a web server.

## Example

```yaml
- hosts: web
  become: true
  roles:
    - role: ctsc.sys.certbot
      vars:
        certbot_email: you@salud.unm.edu
        certbot_eab_kid: "{{ vault_certbot_eab_kid }}"
        certbot_eab_hmac_key: "{{ vault_certbot_eab_hmac_key }}"
        certbot_primary_domain: myhost.unm.edu
        certbot_subject_alternative_names:
          - myhost.salud.unm.edu
        certbot_deploy_hooks:
          - name: reload-nginx
            content: |
              #!/bin/sh
              systemctl reload nginx
```
