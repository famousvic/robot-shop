Checked cluster: node desktop-control-plane is Ready, git remote is famousvic/robot-shop.
Emptied the Instana EUM url in K8s/helm/values.yaml (key was already null).
helm lint passed with no errors.
Installed Robot Shop with helm into namespace robot-shop (revision 1).
mysql memory limit raised from 1024Mi to 2048Mi because it was OOMKilled during startup.
Upgraded Robot Shop with helm (revision 2) to apply the mysql memory change.
PROBLEM: web, dispatch, payment, rabbitmq, ratings, redis and shipping pods are stuck in ImagePullBackOff. The cluster node pulls through registry-mirror:1273 which returns 500 Internal Server Error. Pulling the same images with docker pull on the host works.
PROBLEM: mysql is still OOMKilled (exit code 137) even with the 2048Mi limit. Cause not yet found.
