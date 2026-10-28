# Robot Shop on Kubernetes with monitoring and logging

This is my fork of the Instana Robot Shop demo (https://github.com/instana/robot-shop). It is a small online shop made of many services. I run it on the Kubernetes that comes with Docker Desktop, and I add metrics, a dashboard, logs, an alert rule and some reliability documents around it.

## What is deployed

* Robot Shop, installed with Helm from K8s/helm into the namespace robot-shop.
* A load generator (K8s/load-deployment.yaml) that sends fake traffic to the shop.
* Prometheus, Alertmanager and Grafana (kube-prometheus-stack) in the namespace monitoring.
* Loki and Fluent Bit for logs, also in monitoring.

The Instana settings in the Helm values were emptied, because I do not use Instana here. The mysql memory limit was raised from 1024Mi to 2048Mi because mysql was killed for using too much memory while starting.

## How to install

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

Screenshots go in docs/screenshots.

## What was and was not completed

This section is the honest status. It is updated as work finishes.

* Step 1, deploy Robot Shop: NOT healthy yet. The Helm release is installed and the load generator is applied. cart, catalogue, mongodb, user and web are Running. mysql and rabbitmq are killed for running out of memory (the container limit is hit), and dispatch, payment, ratings, redis and shipping are still pulling images, which is very slow on this machine. So the shop does not work end to end. See NOTES.md for the evidence.
* Step 2, metrics and dashboard: not finished yet. The alert rule and files are written but have not been applied.
* Step 3, logs with Loki and Fluent Bit: not finished yet.
* Step 4, tracing: not done.
* Step 5, SLO document, alert rule and runbook: written. Slack delivery was not set up and not tested.
* Step 6, incident drill: not done, because the instructor runs it.

The instructor starter configs were not available to me, so I wrote my own minimal configs. They use single replicas, no persistence and short retention, because Docker Desktop has only about 7GB of memory.

Robot Shop services do not expose Prometheus request metrics. So the alert rules use pod readiness and restart counts, which are real data but only a substitute for request errors and latency.
