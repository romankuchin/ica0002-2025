# Lab 6

In this lab we will install some monitoring. It's not a complete solution, we will make it better during next labs.

## Task 1: Install Prometheus Node Exporters

Install Prometheus node exporters on all your VMs. No need to configure them, default settings are good enough. Installation can be done in the `init` role.

## Task 2: Install and configure Prometheus

Install Prometheus on VM that doesn't have Agama running.

Configure Prometheus job "linux" with static_configs. Include **all** your VMs there, even future ones.

Don't delete the job "prometheus" which comes with default config.

Hint: use Ansible variable `groups['all']` and for loops.

It's not allowed to use IP addresses in the Prometheus config, only DNS names are allowed.

## Task 3: Make Prometheus available from outside

Install Nginx and configure reverse proxy on the VM with Prometheus installed:

    /prometheus -> localhost:9090

Reuse existing `nginx` role.

Hint: use Ansible variable `groups['prometheus']` for condition in Nginx config.

    {% if inventory_hostname in groups['prometheus'] %}
        ...

## Task 4: Adjust Prometheus service

To make Prometheus reachable from outside, run it with

    --web.external-url=http://<your_public_http_endpoint>/prometheus

Put the required arguments into the `/etc/default/prometheus` file. Adjust `metrics_path`
for job `prometheus` in the Prometheus config to make Prometheus self-monitoring work.
Default `/metrics` won't work because all the links in Prometeheus will have prefix `/prometheus`, so `metric_path` will become `/prometheus/metrics`.

## Task 5: Write some Prometheus queries

Using [docs](https://prometheus.io/docs/prometheus/latest/querying/basics/)
write a query for memory consumption and average CPU load for each VM.

Save queries to `prom_queries.txt`, you will use them during next lab.

## Expected result

Your repository contains these files and directories:

    ansible.cfg
    group_vars/all.yaml
    hosts
    infra.yaml
    prom_queries.txt
    roles/bind/tasks/main.yaml
    roles/prometheus/tasks/main.yaml

Your repository also contains all the required files from the previous labs.

Your repository **does not contain** Ansible Vault master password.

Prometheus and Node Exporters are installed and configured with this command:

	ansible-playbook infra.yaml

Running the same command again does not make any changes to any of the managed
hosts.

After playbook execution all targets at \<your_VM_http_link\>/prometheus/classic/targets should be `up`: all the VMs + Prometheus itself.

After playbook execution you should be able to query historical data from Prometheus web interface \<your_VM_http_link\>/prometheus/classic/graph.
