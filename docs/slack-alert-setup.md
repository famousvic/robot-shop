# Slack alert setup (not done yet)

Slack delivery has NOT been wired up or tested. These are the steps to do it yourself later.

## 1. Create an incoming webhook

1. Go to https://api.slack.com/apps and choose Create New App, then From scratch.
2. Name it, pick your workspace.
3. Open Incoming Webhooks and switch it on.
4. Choose Add New Webhook to Workspace and pick the channel for alerts.
5. Copy the webhook URL. Treat it like a password. Never put it in a file, a commit, or a screenshot.

## 2. Store the URL as a Kubernetes secret

Run this in your terminal and paste the URL where it says so. The URL stays out of the repo.

    kubectl create secret generic alertmanager-slack -n monitoring --from-literal=url=PASTE_YOUR_WEBHOOK_URL_HERE

## 3. Shape of the Alertmanager config

This is only the shape. The URL is read from a file that comes from the secret, so the real value is never written here.

    alertmanager:
      alertmanagerSpec:
        secrets:
          - alertmanager-slack
      config:
        route:
          receiver: slack
          group_by: [alertname, namespace]
        receivers:
          - name: slack
            slack_configs:
              - api_url_file: /etc/alertmanager/secrets/alertmanager-slack/url
                channel: "#YOUR_CHANNEL"
                send_resolved: true
                title: "{{ .CommonLabels.alertname }}"
                text: "{{ .CommonAnnotations.summary }} Runbook: {{ .CommonAnnotations.runbook_url }}"

Put this under the kube-prometheus-stack values (monitoring/prometheus-values.yaml), then run helm upgrade for the monitoring release.

## 4. Test

Stop one focus service for a few minutes, for example scale catalogue to 0, wait for the alert, then scale it back to 1. Check that the message reaches Slack.
