# How Prometheus Monitoring works | Prometheus Architecture explained

Source: https://www.youtube.com/watch?v=h4Sl21AKiDg

* Used for constant monitoring all the services
* Alert when crash
* For identifying problems before they occur

# Main components of Prometheus server

## Prometheus server

* Storage - time series database
* Retrieval - data retrieval worker - pulls metrics data from applications, services, servers, etc. and storing them to the database
* HTTP server - Accept PromQL queries and display it on **Prometheus web UI** or **Grafana** etc.

# Targets and Metrics

What does Prometheus monitor ?

**Targets** - can be Win/Lin servers, Apache server, Single app, Service like database.

Each tartget has units: CPU Status, Memory/Disk space usage, Exception count, Request count, Request duration, etc...

## Metrics

There are 3 metrics types

| Counter                     | Gauge                                    | Histogram             |
| --------------------------- | ---------------------------------------- | --------------------- |
| how many times x happened ? | Gauge - what is current value of x now ? | how long or how big ? |

# Collecting metrics data from targes

Data retrieval worker pulls over HTTP. Pull from **hostaddress/metrics**
