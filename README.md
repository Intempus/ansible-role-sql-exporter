# Ansible role: Prometheus SQL Exporter

Requirements
------------

None

Role Variables
--------------

The variables that can be passed to this role and a brief description about them are as follows. (For all variables, take a look at defaults/main.yml)

```yaml
# User for the exporter
sql_exporter_user: 'postgres'
sql_exporter_group: 'prometheus'

# Base config for exporter
sql_exporter_config:
  global:
    scrape_timeout_offset: 500ms
    min_interval: 0s
    max_connections: 3
    max_idle_connections: 3
    max_connection_lifetime: 10m
    
  collector_files:
  - "{{ sql_exporter_release_dir }}/collectors/*.collector.yml"

  target:
    data_source_name: 'postgres://postgres@/var/run/postgresql/postgres?sslmode=disable'
    # Add any new collectors here
    collectors: [replication]

# TLS Config
sql_exporter_web_config:
  tls_server_config:
    cert_file: "{{ base_certificates_path }}/server.crt"
    key_file: "{{ base_certificates_path }}/server.key"
    client_auth_type: "RequireAndVerifyClientCert"
    client_ca_file: "{{ base_certificates_path }}/root.crt"
    min_version: "TLS12"
    max_version: "TLS13"

# Collectors
sql_exporter_collectors:
  - name: replication
    collector:
      collector_name: replication
      metrics:
      - metric_name: replay_lag_seconds
        type: gauge
        help: 'The replay lag for each server'
        key_labels:
          - client_addr
        values: [replay_lag_seconds]
        query: |
          SELECT client_addr, COALESCE(EXTRACT(EPOCH FROM replay_lag), 0) AS replay_lag_seconds FROM pg_stat_replication

```

Dependencies
------------

None

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2018 by [Maxim Baranov](https://github.com/mbaran0v) updated by David Lintern.
