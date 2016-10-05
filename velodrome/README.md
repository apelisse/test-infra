Velodrome is the dashboard, monitoring and metrics for Kubernetes Developer
Productivity. It is hosted at [velodrome.k8s.io]().

It is made of three different main parts:
- Grafana-stack [grafana-stack/]() is mostly the front-end website where users
  see the metrics along with the back-end databases used to print those
  metrics. It has:
  - an InfluxDB (a time-series database) to save all sorts of metrics,
  - a Prometheus instance to save poll-based metrics (more monitoring
  based)
  - a Grafana instance to display graphs based on these metrics
  - and an nginx to proxy all of these services in a single URL.

- Github statistics are created from a copy of Github database in a SQL
  database. It has the following components:
  - Fetcher [fetcher/](): fetches Github database into a SQL database
  - SQL Proxy [mysql/](): SQL Proxy deployment to Cloud SQL
  - Transform [transform/](): Transform SQL (Github db) into valuable metrics

- Various other monitoring tools, only one for the moment:
  - token counter [token-counter/](): Monitors RateLimit usage of your github
    tokens
