# script-exporter

[![Build Status](https://drone.osshelp.ru/api/badges/ansible/script-exporter/status.svg)](https://drone.osshelp.ru/ansible/script-exporter)

Role for [script_exporter](https://github.com/ricoberger/script_exporter) installation and configuration.

## Usage (example)

```yaml
    - role: script-exporter
      script_exporter_scripts:
        - { name: "some_script", path: "/path/to/script", sudo: false }
        - { name: "another_script", path: "/path/to/another/script", sudo: true }
```

## Available parameters

### Main

Short description here.

| Param | Default | Description |
| -------- | -------- | -------- |
| `script_exporter_setup` | `full` | Setup mode. See [OSSHelp KB article](https://oss.help/kb4895) |
| `script_exporter_scripts` | See [defaults](defaults/main.yml) | Array, describing scripts for exporter to be able to launch. If `sudo` param is set to `true` - corresponding entry will be generated in sudoers.d and the target script will be run with sudo |

## FAQ

### Passing script-name in params

Use location like this:

```nginx
location ~/(.+)/(.+)/metrics$ {
    if ($args ~ "script=.+") {
        rewrite ^/(.+)/(.+)/metrics$ /probe?$args break;
        proxy_pass http://$1-$2-exporter;
    }
    rewrite ^/(.+)/(.+)/metrics$ /metrics break;
    proxy_pass http://$1-$2-exporter;
}
```

Upstream for this location:

```nginx
upstream container-name-script-exporter {
    server container-name:9469;
}
```

Replace the `container-name` with appropriate name here.

### Struggling with Prometheus setup

Job for `prometheus.yml` should be like this, nothing special:

```yaml
- job_name: script-exporter
  metrics_path: /metrics
  scrape_interval: 60s
  scheme: http
  relabel_configs:
  - source_labels:
    - __address__
    regex: ^(.+)\.\w+\.\w+:\d+
    target_label: instance
    action: replace
  file_sd_configs:
  - files:
    - /etc/prometheus/targets/script-exporter/*.yml
    refresh_interval: 5m
```

The tricky part is the include itself, template:

```yaml
- targets:
  - 'server-alias.ossdata.ru'
  labels:
    __scheme__: http
    __metrics_path__: /container-name/script/metrics
    __param_script: "script-alias"
    job: script-exporter
    env: production
    group: somegroup
    project: project-name
    instance: server-alias
    source: container-name
```

`script-alias` here is a name, used for script in script-exporter's cfg.

## Useful links

- [Official documentation](https://github.com/ricoberger/script_exporter/blob/master/README.md)

## TODO

- improve this readme (quick'n'dirty for now)

## License

GPL3

## Author

OSSHelp Team, see <https://oss.help>
