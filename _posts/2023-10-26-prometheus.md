---
title: "Prometheus"
date: 2023-10-26 09:30:00 +0900
categories: [Level, Senior]
tags: [Monitoring, Prometheus]
publish: false
---

# Overview
## What is Prometheus?
- Prometheus is an open-source systems monitoring and alerting toolkit
- Prometheus collects and stores its metrics as time series data, i.e. metrics information is stored with the timestamp at which it was recorded, alongside optional key-value pairs called labels.
- time series collection happens via a pull model over HTTP

### Components
- the main Prometheus server which scrapes and stores time series data
- client libraries for instrumenting application code
- a push gateway for supporting short-lived jobs

## Downloading Prometheus
```
tar xvfz prometheus-*.tar.gz
cd prometheus-*
./prometheus --help
```

## Configuring Prometheus
- The Prometheus download comes with a sample configuration in a file called `prometheus.yml`

```
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
```

- `scrape_interval`, controls how often Prometheus will scrape targets. You can override this for individual targets. 
- The `evaluation_interval` option controls how often Prometheus will evaluate rules. Prometheus uses rules to create new time series and to generate alerts.
- The `rule_files` block specifies the location of any rules we want the Prometheus server to load.
- The last block, `scrape_configs`, controls what resources Prometheus monitors.
- Prometheus expects metrics to be available on targets on a path of `/metrics`.

## Starting Prometheus
```
./prometheus --config.file=prometheus.yml
```
