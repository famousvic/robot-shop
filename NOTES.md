Checked cluster: node desktop-control-plane is Ready, git remote is famousvic/robot-shop.
Emptied the Instana EUM url in K8s/helm/values.yaml (key was already null).
helm lint passed with no errors.
Installed Robot Shop with helm into namespace robot-shop (revision 1).
mysql memory limit raised from 1024Mi to 2048Mi because it was OOMKilled during startup.
Upgraded Robot Shop with helm (revision 2) to apply the mysql memory change.
PROBLEM: web, dispatch, payment, rabbitmq, ratings, redis and shipping pods are stuck in ImagePullBackOff. The cluster node pulls through registry-mirror:1273 which returns 500 Internal Server Error. Pulling the same images with docker pull on the host works.
PROBLEM: mysql is still OOMKilled (exit code 137) even with the 2048Mi limit. Cause not yet found.
OOM evidence from dmesg: mysqld was killed at about 2.0Gi resident memory (the 2048Mi limit), rabbitmq beam.smp at about 512Mi (its 512Mi limit). The Docker VM itself had about 4.6G available, so this is the app limits, not a full machine.
The voting namespace has no pods, so it was already uninstalled; nothing was freed by a new uninstall.
Fixed mysql and rabbitmq OOM kills by capping open files with ulimit -n 65536 in the chart templates (likely cause, worked after the change). Raised limits to 3072Mi and 1024Mi.
Deleted the shipping pod once to retry its pull. Reason was a mirror 500 error then a timeout, a network problem.
Wrote monitoring/prometheus-values.yaml, logging/loki-values.yaml and logging/fluent-bit-values.yaml. Not installed and not tested. Chart docs could not be reached.
Wrote docs/slo.md, docs/runbook.md, docs/postmortem-template.md, docs/slack-alert-setup.md and monitoring/robot-shop-alerts.yaml (alert rule not applied).
20:40 cut off: shop not fully healthy, so Phase 2 and 3 were not started. README finalized to match.
