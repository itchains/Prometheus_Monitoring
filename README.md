### Prometheus_monitoring ###
Monitoring solution with Prometheus, Alertmanager and Grafana

## Firewall ##

The only ports allowed are 80,443 and 22


## Services ##

All services are controlled via systemd (/etc/systemd/system)
prometheus.service
alertmanager.service
node_exporter.service


## Grafana ##

The service its installed from repository, so it can be upgraded via yum update command
The dashboards have a regex filter /pfm.*/ and /std.*/


## Prometheus ##

Data retention is 60 days, in can be changed in the systemd unit file
To undapte prometheus binary, you can download the latest from https://prometheus.io/download/

# Add prometheus clients
1. Downloaded node_exporter binary from Prometheus download page directly https://prometheus.io/download/
2. Create and enable the systemd unit file

$ cat < /etc/systemd/system/node_exporter

# Paste the text below and press CTRL+C

[Unit]
Description=Prometheus exporter for machine metrics
Documentation=https://github.com/prometheus/node_exporter

[Service]
Restart=always
ExecStart=/opt/monitoring/prometheus/node_exporter/node_exporter --collector.systemd

ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
SendSIGKILL=no

[Install]
WantedBy=multi-user.target

3. Run the commands to enable the unit file on boot and start the service

$ systemctl enable node_exporter
$ systemctl start node_exporter

4. On the client allow port 9100 incoming traffic on the firewall
5. Add client hostname in prometheus.yml config file
$ vim /opt/monitoring/prometheus/prometheus.yml

# Add the hostname with a comma separated on the following block and save the file

  - job_name: 'node_exporter'

    static_configs:
    - targets: ['localhost:9100', '<new client hostname in here>']



## Postfix server - to send the email alerts

Postfix server has the default settings and can only send emails, port 25 its not allowed inbound


## ALERTS ##

# Check alerts from command line
$ cd /opt/monitor/alertmanager
$./amtool alert --alertmanager.url=http://127.0.0.1:9093

# Check alerts in Prometheus web gui
http://127.0.0.1:9090/alerts

# Check alerts in AlertManager web gui
http://127.0.0.1:9093/#/alerts

# A handy alerts list
https://awesome-prometheus-alerts.grep.to/rules.html

# To setup additional alert rules edit the following config file
$vim /opt/monitoring/prometheus/alert_rules.yml

# Example additional rule declaration
# Alert when a systemd unit is in failed state, it can be for a specific service
- alert: SystemdServiceFailed
    expr: node_systemd_unit_state{state="failed", name="alertmanager.service"} == 1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: Host SystemD service crashed (instance {{ $labels.instance }})
      description: SystemD service crashed\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}


