# Service Level Objective for Robot Shop

## What we measure (the SLI)

The SLI is availability. It is the share of requests that succeed.

    availability = successful requests / all requests

A request counts as successful when the shop answers with a status code below 500. It counts as a failure when the answer is 500 or higher, or when there is no answer at all.

## The target (the SLO)

99 percent of requests succeed, measured over a rolling 30 days.

## The error budget in plain numbers

The budget is the 1 percent that is allowed to fail.

* 30 days is 720 hours, or 43,200 minutes.
* 1 percent of 43,200 minutes is 432 minutes.
* That is 7 hours and 12 minutes of full downtime in 30 days.

If the shop is only partly broken, the budget is used more slowly. For example, if half of all requests fail, the budget is used at half speed, so the same budget lasts about 14 hours and 24 minutes.

Example with request counts: if the shop handles 1,000,000 requests in 30 days, up to 10,000 of them may fail before the SLO is missed.

## What the team does when the budget is used up

1. Freeze risky releases. Only fixes for reliability or security may go out.
2. Make reliability the first job. The team works on the cause of the failures before any new feature.
3. Hold a review. Write a blameless postmortem (see docs/postmortem-template.md) and agree on actions with owners and dates.
4. Lift the freeze only when the 30 day success rate is back above 99 percent, or when the team agrees the risk is low.

## When the budget is running low

At 50 percent used, the team looks at the trend and checks what is causing the failures.
At 75 percent used, the team stops optional risky changes and tells everyone.

## Honest note about the data

Robot Shop services do not all expose request metrics to Prometheus. Until request metrics exist, the alert and dashboard use the best real signals that are available (pod readiness and restarts). See the README for what was real and what was a substitute.
