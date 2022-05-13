# ECS Service Metrics Exporter

This software is meant for AWS ECS environments, where it scrapes metrics from all running services that support it. Then, it makes the scraped metrics
available via a HTTP endpoint.

When deployed within an ECS cluster, the service should run in `daemon` mode (so it is deployed to each instance in the cluster) as well as given
permission to access the Docker daemon. For this, the easiest way is to mount the Docker socket into the container, but please be **careful** about
security implementations. This will effectively grant this app root privileges on the ECS server.

When the app's HTTP endpoint `/metrics` is invoked, it will connect to the Docker daemon and enumerate all container that have a specific label. The label
name can be configured via the environment (see `.env.example` for configuration). The label for each service should contain the service's port and path,
like so: `9100/path/to/metrics`. This allows flexible configuration for each service, on which port and under which part the metrics are available.
For all running containers, the app will then try to execute a cURL command `curl -s http://localhost:$PORT$$METRICPATH$`. For the above example, it would
execute the command `curl -s http://localhost:9100/path/to/metrics`.

The output of the metrics of all services will be combined and returned to the (HTTP) caller. To allow discriminiating metrics by service, and extra label
`container_name` is added to every metric. The source of this label is the Docker label `com.amazonaws.ecs.container-name`.
