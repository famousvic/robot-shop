# Runbook for Robot Shop alerts

This page is linked from the alerts in monitoring/robot-shop-alerts.yaml.

## What the alerts mean

* RobotShopPodNotReady: a pod for web, catalogue, cart or payment has not been ready for 5 minutes. Customers may see errors or slow pages for that part of the shop.
* RobotShopPodRestarting: a pod for one of those services restarted more than 3 times in 15 minutes. This usually means it is crashing, often from a bug, bad config or running out of memory.

## How to check

1. Look at the pods:

       kubectl get pods -n robot-shop

2. Look at why a pod is unhappy (check the Events at the bottom):

       kubectl describe pod <pod name> -n robot-shop

3. Look at its recent logs (add --previous if it restarted):

       kubectl logs <pod name> -n robot-shop --tail=50

4. Open the Robot Shop dashboard in Grafana (http://localhost:3000 after the port forward) and look for spikes in CPU, memory, restarts or errors.
5. In Grafana, open Explore, pick the Loki data source and run:

       {namespace="robot-shop"} |= "error"

## How to reduce the damage first (mitigate)

Do the safest step first and check again after each one.

1. Restart the service:

       kubectl rollout restart deployment/<name> -n robot-shop

2. If it started after a recent change, roll back:

       helm history robot-shop -n robot-shop
       helm rollback robot-shop <previous revision> -n robot-shop

3. If it is overloaded, add a replica for a short time:

       kubectl scale deployment/<name> --replicas=2 -n robot-shop

   Check that the machine has enough memory before you do this.

## Who to tell

* Tell the person on call and the team lead straight away.
* Post a short note in the team chat: what is broken, what you tried, what you will try next.
* If customers are affected for more than 15 minutes, tell the product owner.
* After it is fixed, write a postmortem using docs/postmortem-template.md.
