# Grafana Agent

This sets up the grafana agent to send metrics to a remote prometheus `remote_write` compatible endpoint.
So far, only the integrated node_exporter is configured.

## Requirements

This role requires the Ansible action plugin `ansible_merge_vars`.

## Config options

* `grafana_agent__<name>_prometheus_scrapes__to_merge`: Allows you to pass through Prometheus scraping blocks
* `grafana_agent__<name>_logs_scrapes__to_merge`: Allows you to pass through loki scraping blocks
* `grafana_agent__<name>_blackbox_targets__to_merge`: Allows you to pass through blackbox scraping blocks

\* When config-options are defined in `group_vars` as well as in `host_vars`
Ansible would overwrite the `group_vars` variable with the one from the
`host_vars` (Similarly, when it's defined in several `group_vars`). To avoid
this, this role merges all dictionaries with the suffix
`_prometheus_scrapes__to_merge` (or `_logs_scrapes__to_merge`, etc.).
**This only works if the variables have different names, therefore `<name>`
should be unique across all group_vars and host_vars a host is in**.

## Logs

You can enable log-collection using the integrated Promtail.
For this, some scrape config `grafana_agent__<name>_logs_scrapes__to_merge` and
`grafana_agent_logs_client_url` must be set. The latter is a Loki-endpoint to
which the logs will be sent. An example for `grafana_agent__<name>_logs_scrapes__to_merge`
is:

    grafana_agent__rundeck_logs_scrapes__to_merge:
      - job_name: varlogs
        static_configs:
          - targets: [localhost]
            labels:
              job: varlogs
              __path__: /var/log/*log
              hostname: "{{ ansible_fqdn }}"

## Blackbox (Website probes)

Set `grafana_agent_enable_blackbox` to `true` and add the targets you want to
be probed in `grafana_agent__<name>_blackbox_targets__to_merge`, e.g:

    grafana_agent__websites_blackbox_targets__to_merge:
      - name: example
        address: example.com
        module: http_2xx

The module must be a [blackbox module](https://github.com/prometheus/blackbox_exporter/blob/master/CONFIGURATION.md) defined in `grafana_agent_blackbox_module_config`.


## Postgres exporter

Follow the [official documentation](https://github.com/prometheus-community/postgres_exporter#running-as-non-superuser)
on how to configure a separate user for the agent. This will configure the DB
for user `postgres_exporter`.
Specify database `postgres` and your user (e.g. `postgres_exporter`) in
`grafana_agent_prometheus_postgresexporter_datasourcenames`:

    "psql://postgres_exporter:password@example.com/postgres"
