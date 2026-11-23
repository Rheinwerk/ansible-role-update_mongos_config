mongos config update
=========

This role can be used to update a mongos config, e. g. early in the
startup process before the actual services runs.

[![Build Status](https://github.com/Rheinwerk/ansible-role-update_mongos_config/actions/workflows/ci.yml/badge.svg)](https://github.com/Rheinwerk/ansible-role-update_mongos_config/actions/workflows/ci.yml)

It writes `/etc/mongos.conf`, the systemd unit `mongos.service` (disabled, the role
does not start it), `/usr/local/bin/mongos-pre-start.sh`, `/etc/logrotate.d/mongos`
and, with TLS, the certificate key file.

Requirements
------------

The target machine must have mongos installed, for example with the
[mongos](https://github.com/Rheinwerk/ansible-role-mongos) role, and the user and
group in `_mongos_user` and `_mongos_group` must exist.

Role Variables
--------------

Every variable has a default (`defaults/main.yml`); set only the ones that differ.
`meta/argument_specs.yml` and `tasks/assert.yml` check the values before anything
is written, so a wrong type, an unknown key inside `_mongos_sharding` or `_mongos_tls`,
or the `_mongos` dict of v0.x fails the run at the start.

| Variable | Default | Description |
| --- | --- | --- |
| `_mongos_log_path` | `/var/log/mongodb` | Absolute path of the log directory (mongos.log). |
| `_mongos_user` | `mongodb` | User mongos runs as. |
| `_mongos_group` | `mongodb` | Group mongos runs as. |
| `_mongos_bind_ip` | `127.0.0.1` | `net.bindIp`, one address or a comma-separated list. |
| `_mongos_port` | `27017` | `net.port`. |
| `_mongos_max_connections` | `300` | `net.maxIncomingConnections`. |
| `_mongos_sharding` | `configdb` with `replicaset_name: configsrv` and `members: [mongo-config.service.centerdevice:27019]` | Config server replica set, rendered as `sharding.configDB: <replicaset_name>/<members>`. Both keys are required. |
| `_mongos_tls` | `certificatekeyfile: etc/clientserver-mongo-server_combined.pem`, `certificatekeyfilepassword: "{{ S_MONGOS.tls.certificatekeyfilepassword }}"`, `cert_glob_file: "{{ SITE.cert_glob_file }}"` | TLS settings, `{}` disables TLS. See below. |

`_mongos_tls` keys:

| Key | Required | Description |
| --- | --- | --- |
| `certificatekeyfile` | yes | Combined certificate and key, relative to the playbook's `files/`; copied to `/<certificatekeyfile>`. |
| `certificatekeyfilepassword` | yes | Password of the key, not empty. |
| `cert_glob_file` | no | Absolute path of a file that lists certificate files; the role adds `/<certificatekeyfile>`. |
| `mode` | no | `net.tls.mode`: `disabled`, `allowTLS`, `preferTLS` or `requireTLS` (default). |
| `cafile` | no | `net.tls.CAFile`, default `/etc/ssl/certs/ca-certificates.crt`. |
| `allow_connections_without_certificates` | no | Boolean, default `true`. |

The TLS default reads the CenterDevice consumer contract: `SITE.cert_glob_file` and the
vault dict `S_MONGOS`. A play without them sets `_mongos_tls` itself.

Dependencies
------------

None.

Example Playbook
----------------

    - hosts: servers
      vars:
        _mongos_max_connections: 1000
      roles:
        - { role: update_mongos_config, tags: ['mongos'] }

License
-------

Please see LICENSE.

Author Information
------------------

Original author is [Daniel Schneller](https://github.com/dschneller) as member of the [Rheinwerk](https://github.com/Rheinwerk) project.
