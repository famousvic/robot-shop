# Robot Shop on Kubernetes with monitoring and logging

This is my fork of the Instana Robot Shop demo (https://github.com/instana/robot-shop). It is a small online shop made of many services. I run it on the Kubernetes that comes with Docker Desktop, and I add metrics, a dashboard, logs, an alert rule and some reliability documents around it.

## What is deployed

* Robot Shop, installed with Helm from K8s/helm into the namespace robot-shop.
* A load generator (K8s/load-deployment.yaml) that sends fake traffic to the shop. It is applied but has not started, because its image could not be pulled.
* Planned but NOT deployed: Prometheus, Alertmanager and Grafana (kube-prometheus-stack) in the namespace monitoring, and Loki and Fluent Bit for logs. Only their values files exist, and they are untested.

The Instana settings in the Helm values were emptied, because I do not use Instana here. The mysql memory limit was raised from 1024Mi to 2048Mi because mysql was killed for using too much memory while starting.

## Current status in one line

Robot Shop is partly running: 8 of 13 pods are Running (cart, catalogue, mongodb, mysql, rabbitmq, redis, user, web). dispatch, payment, ratings, shipping and the load generator are stuck in ImagePullBackOff because of slow image pulls. Monitoring and logging are not installed. See "What was and was not completed" below.

## How to install

The install steps below are what to run. Only the Robot Shop part was actually run. The monitoring and logging commands were not run.

You need Docker Desktop with Kubernetes on, plus kubectl and Helm.

Shop:

    helm install robot-shop ./K8s/helm -n robot-shop --create-namespace
    kubectl apply -f K8s/load-deployment.yaml -n robot-shop

Monitoring (values are in monitoring/prometheus-values.yaml):

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace -f monitoring/prometheus-values.yaml
    kubectl apply -f monitoring/robot-shop-alerts.yaml

Logging (values are in logging/loki-values.yaml and logging/fluent-bit-values.yaml):

    helm repo add grafana https://grafana.github.io/helm-charts
    helm repo add fluent https://fluent.github.io/helm-charts
    helm install loki grafana/loki -n monitoring -f logging/loki-values.yaml
    helm install fluent-bit fluent/fluent-bit -n monitoring -f logging/fluent-bit-values.yaml

## How to open things

Shop:

    kubectl port-forward svc/web 8082:8080 -n robot-shop

Then open http://localhost:8082.

Grafana:

    kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring

Then open http://localhost:3000. The user is admin. To read the password, run this yourself in your own terminal:

    kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 -d

## Where things are

* K8s/helm: the Helm chart for Robot Shop.
* monitoring/prometheus-values.yaml: small values for Prometheus and Grafana.
* monitoring/robot-shop-dashboard.json: the Grafana dashboard saved as code.
* monitoring/robot-shop-alerts.yaml: the alert rules.
* logging/: values for Loki and Fluent Bit.
* docs/slo.md: the SLI, SLO and error budget.
* docs/runbook.md: what to do when the alert fires.
* docs/postmortem-template.md: a blank blameless postmortem template.
* docs/slack-alert-setup.md: steps to connect Slack (not done yet).
* NOTES.md: a running list of what was done.

Screenshots are in docs/screenshots:

* shop-page.png: a real browser screenshot of http://localhost:8082 (the shop front page loads).
* pods.png: the output of kubectl get nodes and kubectl get pods, with the failing pods in red.
* mysql-rabbitmq.png: mysql and rabbitmq Running with 0 restarts, the ulimit command, and the memory limits.
* helm-release.png: the Helm release.
* shipping-pull-error.png: the image pull errors for shipping.

The four terminal images were made by running the real commands and drawing their output into an image. They are not photos of a terminal window. A screenshot of the GitHub repo page could not be taken, because the network was too slow.

## Notes on memory problems (mysql and rabbitmq)

When first installed, mysql and rabbitmq were killed for running out of memory within seconds of starting. Raising the memory limit did not help, because each one simply grew until it hit the new limit (mysql was killed at about its 2Gi limit, then at about 3Gi; rabbitmq at about 512Mi, then about 1Gi). The Docker Desktop VM itself still had about 4.6GB free.

Likely cause: the open file limit inside the containers is about 1 billion (`ulimit -n` showed 1073741816). MySQL 5.7 and RabbitMQ (Erlang) are known to size internal tables from that number. This is a likely cause, and it worked after the change below. I did not prove it in isolation.

Workaround: the chart templates for mysql and rabbitmq now start the process with `ulimit -n 65536` first. After that, both pods stayed Running with 0 restarts for 10 minutes. The memory limits are kept at 3072Mi for mysql and 1024Mi for rabbitmq, set in K8s/helm/values.yaml.

## Files that are written but not yet tested

These files were written without being installed. The chart docs could not be reached when they were made, so they come from memory of the charts and may need small fixes. They stay in this list until each one has been installed and checked.

* monitoring/prometheus-values.yaml: written but not yet tested.
* logging/loki-values.yaml: written but not yet tested.
* logging/fluent-bit-values.yaml: written but not yet tested.
* monitoring/robot-shop-alerts.yaml: written but not yet tested (not applied to a cluster).

## What was and was not completed

This section is the honest status. It is updated as work finishes.

* Step 1, deploy Robot Shop: NOT healthy yet. The Helm release is installed and the load generator is applied. cart, catalogue, mongodb, mysql, rabbitmq, redis, user and web are Running. dispatch, payment, ratings and shipping are still failing to pull images (ImagePullBackOff), and the load generator has not started for the same reason. The network on this machine is very slow and unreliable. So the shop does not work end to end. See NOTES.md for the evidence.
* Step 2, metrics and dashboard: NOT DONE. Prometheus and Grafana were never installed, because the shop was not healthy at the 8:40pm cut off and the slow network could not pull more images. monitoring/prometheus-values.yaml and monitoring/robot-shop-alerts.yaml are written but not tested. There is no dashboard JSON (monitoring/robot-shop-dashboard.json does not exist).
* Step 3, logs with Loki and Fluent Bit: NOT DONE. Nothing was installed. logging/loki-values.yaml and logging/fluent-bit-values.yaml are written but not tested, and the Loki data source is only defined inside monitoring/prometheus-values.yaml.
* Step 4, tracing: not done.
* Step 5, SLO document, alert rule and runbook: the documents are written (docs/slo.md, docs/runbook.md, docs/postmortem-template.md, docs/slack-alert-setup.md). The alert rule file is written but was never applied, so Prometheus never loaded it. Slack delivery was not set up and not tested.
* Step 6, incident drill: not done, because the instructor runs it.

The instructor starter configs were not available to me, so I wrote my own minimal configs. They use single replicas, no persistence and short retention, because Docker Desktop has only about 7GB of memory.

Robot Shop services do not expose Prometheus request metrics. So the alert rules use pod readiness and restart counts, which are real data but only a substitute for request errors and latency.
