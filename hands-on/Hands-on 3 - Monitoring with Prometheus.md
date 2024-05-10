## Hands-on 3 - Monitoring with Prometheus

### Overview

For the purpose of the ISC 2024 Compute Continuum Tutorial we have deployed a monitoring stack into the tutorial cluster.
The stack consists of:
- Prometheus, as metrics storage and time-series database
- Node Exporter, as per-node metrics exporter
- Alertmanager, as alerting service based on metrics events
- Grafana, as metrics dashboard

This stack can easily be deployed using the `kube-prometheus-stack` helm chart ( https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack ) and is a common monitoring stack for Kubernetes deployments.

For this part of the hands-on you will interact with Grafana to discover the metrics stored in Prometheus, which were gathered by the Node Exporter.
Additionally, you will also create your own dashboard and alerts.

### Exploring Grafana

#### Logging In

Open this URL in your browser: http://141.2.112.17:32536/
This takes you to the login page of the Grafana instance deployed in the cluster.

Sign-up as been enabled, go ahead and create an account.
The email is only required for the login and is not actually used for anything so it can be a fake email such as `YOUR_NAME@test.test`.

#### Default Dashboards

Once logged in, you can find on the left through the hamburger menu a number of options including `Dashboards`.
Click this option to see a number of pre-configured dashboards that are automatically provisioned by Grafana when deploying as the `kube-prometheus-stack`.

Have a look at `Kubernetes / Compute Resources / Cluster` for the cluster-wide resource consumption.
Be mindful the dashboard might take a moment to load the first time.
The graphs show resource consumption by namespace.

Next have a look at `Kubernetes / Compute Resources / Namespaces (Pods)` and under `namespace` in the top bar set it to a specific namespace such as `kube-system` or `decice`.

Also have a look at `Node Exporter / Nodes` to see resource usage per node.
Under `instance` you can select the node you wish to see metrics for, in our setup this currently shows the respective IPs of the node exporter pods on the cluster.
The mapping to the nodes is as follows:
- 20.0.0.12   cn01.guoehi.cluster (control-plane)
- 20.0.1.16   cn02.guoehi.cluster (worker)
- 20.0.2.56   cn03.guoehi.cluster (worker)
- 20.0.0.59   cn04.guoehi.cluster (worker)
- 20.0.0.26   cn05.guoehi.cluster (worker)
- 20.0.5.22   cn06.guoehi.cluster (worker)
- 20.0.7.11   cn07.guoehi.cluster (edge)

The last node serves as our test edge node running KubeEdge.
The node itself is a regular node of the cluster that is running KubeEdge instead of the regular Kubernetes stack.
As an experimental addition for the tutorial there might be one or more additional node that reflect Raspberry-Pis, which we have brought to the ISC.
Have a look through the nodes with a focus on the edge nodes.

Feel free to continue and explore the other dashboards.

#### Exploring Metrics

Prometheus can be queried using PromQL, a query language specifically made for querying metrics.
PromQL supports operations such as data selection and filtering but also advanced operations such as aggregation and manipulation.

In the Grafana interface, from the hamburger menu on the left select `Explore`.
This opens a query builder that enables the creation of new queries.
It supports two modes, `builder` and `code`, where builder provides assistance with the query creation and code allows one to freely type in queries.

Switch to query mode `code` on the right and paste in the following query
`topk(10, count by (__name__)({__name__=~".+"}))`
Furthermore, below the query, under options, set `Type` to `Instant`.
Then run your query.
This returns the 10 metric buckets in the system with the highest cardinality of entries, meaning these are the top 10 busies or noisiest metrics in the system.
Feel free to change the number from top 10 to something else to see more results.

Please note that a query might fail due to a timeout, in that case, try again, the query should work after a few tries.

From the previous query, the metrics with the highest cardinality should be `apiserver_request_slo_duration_seconds_bucket`.
Try entering just the name of the metric as a query.
You can replaced your old query or create another query via the `Add query` button.
Please note that Grafana will run all queries at once and try to group the outputs together, which might be more difficult to parse for vastly different outputs.
When running just the name of a metric as a query with query `Type` `Instant`, it will give you the list of entries for that metrics.
Using the `Expand results` toggle allows one to view the results in a more readable format.

Next try to query for the metric `node_memory_MemAvailable_bytes`.
Run the query to see the amount of free memory per node.
Use `Expand results` to see the available labels.
See that one the labels is `instance`, which refers to the node.

Set `Type` for the query back `Both` and switch from query mode `code` to `builder`.
Next to the metric it now suggests to add `Label filters`.
Select from the drop-down menu `instance` as label and for the value any IP and re-run the query.

Now lets look at the pods that were deployed during the earlier hands-on parts.
Run this query `sum(node_namespace_pod_container:container_cpu_usage_seconds_total:sum_irate{namespace="decice"}) by (pod)` to get the CPU usage of all pods in the namespace `decice`.

To further filter down the metrics to only show your own pods you can use a regex filter.
Expand the query to `sum(node_namespace_pod_container:container_cpu_usage_seconds_total:sum_irate{namespace="decice", pod=~".*YOUR_NAME.*"}) by (pod)`
and replace `YOUR_NAME` with your user name that was used as part of the name of the pods that you deployed and re-run the query.

### Creating a Custom Dashboard

#### From Exploration to Dashboard

One of the powerful features of Grafana is the creation of custom dashboards to aggregate and visualize metrics.
The queries that were just created through the explore feature can now be turned into a dashboard.

In the left hamburger menu, switch back to dashboards and find the folder ISC2024.
Open the folder and select `New` in the top right and then `New Dashboard`.
As Datasource select `Prometheus` and then start via `Add visualization`.
This brings you to a query builder similar to the one from the explore section.

Insert `sum(node_namespace_pod_container:container_cpu_usage_seconds_total:sum_irate{namespace="decice", pod=~".*YOUR_NAME.*"}) by (pod)` with `YOUR_NAME` replaced as earlier for the query and run the query.
On the right you can find a menu with a lot of options to further customize the look of your query.
Try setting under `Panel options` the `Title` to a name of your choice.
Scroll down until you see `Standard options` and for `Unit` select in the drop-down `Time` and then `seconds`.
Then `Save` and `Apply` your changes in the top right to now see your dashboard.
The dashboard should now contain one time-series graph for CPU time.

#### Additional Visualization Formats

Grafana supports many more types of graphs.
In the top right of your graph, find the three dot menu and under `More...` find the option `Duplicate`.
You should now see your graph twice.
For the new copy select the menu three dot menu again and then `Edit`.
Back in the query editor find in the top right the menu for Visualizations, which currently reads `Time series`.
Try switching to a few other Visualizations such as
- Bar chart
- Stat
- Gauge
- Table
- Pie chart
- Histogram

For the Gauge visualization, also try adding a threshold by finding the `Thresholds` section and setting a Threshold to `0.0001`, which is equivalent to 100 µs.
When you are done `Save` and `Apply` your changes to return to your dashboard.

#### Alerting

Grafana also allows the creation of alerts for dashboards.
One way to create a new rule is directly from a dashboard, for this select the three dot menu and under `More...` select `New alert rule`.
This opens a new window for creating the conditions for a new alert.

Under `1. Enter alert rule name` set a new name for the alert or leave the existing one.
Under `2. Define query and alert condition` a pipeline with three steps has been created for you.
For `A` you should see your query that you used earlier to see cpu per pod.
Then below under `Expressions` you should see `B Reduce` and `C Threshold`, with each using the previous component as `Input`.
For `B` it should be set to `Function: Last` and `Mode: Strict`.
For `C` a threshold needs to be defined, here you can use `0.0001` for `IS ABOVE` and click on `Preview`, which will render the query and evaluate the threshold.

Next under `3. Set evaluation behavior` rules can be set for how often a rule needs to be evaluated and how long it needs to be triggered before firing an alert.
Set the Evaluation group to `ISC2024` or set a new evaluation group.

The following two sections `4. Configure labels and notifications` as well as `5. Add annotations` can be used to further customize behavior and set what `Contact points` and `Notification policies` to use.
Grafana supports a wide range of integrations for `Contact points` including Email, Microsoft Teams, Slack and Webhooks as well as Alertmanager itself, which in turn supports further integrations and alert rules.
`Notification policies` allow setting when an alert is send the first time and in what intervals to repeat the alert.
Setting up `Contact points` and `Notification policies` goes beyond the scope of this tutorial.

### Outlook

#### Application Specific Metrics

The above hands-on has demonstrated how to visualize the metrics captured by Prometheus through Node Exporter by default but what about application specific metrics?
Many applications provide integrations with Prometheus but for self-developed applications it also possible to add metrics endpoints using the official client libraries (https://prometheus.io/docs/instrumenting/clientlibs/).

However, exposing metrics is not enough as Prometheus needs to be told where to find the metrics in order to scrape them.
In a dynamic environment such as Kubernetes, where the location and IP of pods running is not fixed, Prometheus needs to also be able to dynamically discover new scraping endpoints.

The `kube-prometheus-stack` comes with two CRDs (Custom Resource Definitions) called `ServiceMonitor` and `PodMonitor`, which can be used to collect metrics from a number of `Services` via `ServiceMonitor` and a number of `Pods` using `PodMonitor`.
See the tutorial from the official documentation for further details (https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/user-guides/getting-started.md).

#### Closing Remarks

PromQL and the Grafana dashboard may seem daunting but modern LLM system such as ChatGPT and LLaMA 3 are competent assistants for designing PromQL queries.
Furthermore, many example queries are available online or even through the default dashboards.
Grafana supports many more datasources besides just Prometheus such as InfluxDB and can be a valuable companion for any administrator.
