# Lab 8

In this lab we will setup centralized logging.

## Task 1: Install Loki

Use VM without Prometheus. Follow the official install guide:
https://grafana.com/docs/loki/latest/setup/install/local/#install-using-apt-or-rpm-package-manager

## Task 2: Install Promtail

Install it to all VMs in init role(same docs as in task 1).
Files to track:

  - /var/log/syslog
  - /var/log/uwsgi/app/agama.log
  - /var/log/nginx/access.log
  - /var/log/nginx/error.log

Make sure that Promtail will keep its state in case of VM reboot (do not store anything in /tmp).

Do not track files that do not exist.

Use {{ inventory_hostname }} for hostname label.

Send logs from these files to Loki.

## Task 3: Create new dashboard with Agama logs

And allow developers to view it without login: 
allow anonymous grafana access as viewer for Main Org.
Docs: https://grafana.com/docs/grafana/latest/setup-grafana/configure-security/configure-authentication/anonymous-auth/#configuration

Don't forget to add new json in your Ansible repo.

Don't forget to add Loki to provisioned datasources in Grafana.

## Task 4: Add Promtail and Loki monitoring to main Grafana dashboard

Make sure that you gather metrics from Promtail and Loki.

Add to main dashboard rate of `loki_log_messages_total` and rate of `promtail_sent_entries_total`.

Don't forget to update json in your Ansible repo.

## Task 5: Create slo.md

Prepare documentation for SRE team. Write at least 2 user journeys for main Agama page. Describe SLIs and SLOs.

Must have SLI types:
  - availability
  - latency

If you feel that something is missing - add it.

Docs for inspiration: https://sre.google/resources/practices-and-processes/art-of-slos/

Example from worksheet:

Make sure that your SLIs have an event, a success criterion, and specify where and 
how you record success or failure. Describe your specification as the proportion of 
events that were good. Make sure that your SLO specifies both a target and a 
measurement window.

User Journey:

SLI Type:

SLI Specification:

SLI Implementations:

SLO:

## Task 6: Setup SLI tracking

Configure Nginx logs for Agama SLI tracking and start collecting them into Loki.

Make sure you're collecting only `valid` events (requests to prometheus/grafana are not considered valid for main agama page SLI)! Separate access_log for agama will make it much easier.

Adjust Grafana main dashboard, add SLO as thresholds to SLI tracking panels.

Examples of logQL queries for all events (not only valid ones):

    avg_over_time(
    {filename="/var/log/nginx/agama.log"}
    | regexp '(?P<latency>\d+\.\d+)$'
    | unwrap latency
    [10m]
    )

    100*(
    rate({filename="/var/log/nginx/agama.log"} 
    |~ `^[23]\\d\\d ` [10m])
    / rate({filename="/var/log/nginx/agama.log"}[10m]))

Do not forget to add Loki to provisioned datasources in Grafana.

Useful docs: https://docs.nginx.com/nginx/admin-guide/monitoring/logging/

## Task 7: Test your SLO

1. Drop `agama` database in MySQL to verify that availability goes down.
2. Add latency between VMs with

        tc qdisc add dev ens3 root netem delay 100ms 50ms

    this will add 100 ±50ms to every outgoing packet, including ping and ssh! Rollback:

        tc qdisc del dev ens3 root

## Expected result

Your repository contains these files and directories:

    ansible.cfg
    group_vars/all.yaml
    hosts
    infra.yaml
    slo.md
    roles/loki/tasks/main.yaml

Your repository also contains all the required files from the previous labs.

Your repository **does not contain** Ansible Vault master password.

Everything is installed and configured with this command:

	ansible-playbook infra.yaml

Running the same command again does not make any changes to any of the managed
hosts.

After playbook execution you should be able to see all logs in Grafana->Drilldown->Logs.
