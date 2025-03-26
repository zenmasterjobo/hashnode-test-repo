---
title: "An analysis of the Square and Cash App outage"
datePublished: Wed Mar 05 2025 11:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm8q3lfd6000909kv4pxg2hdp
slug: an-analysis-of-the-square-and-cash-app-outage
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1743004027292/e915d4ca-f1d0-4339-9145-32d37a7587b3.png

---


## What Happened

On February 26, we experienced a system-wide service disruption affecting Square payment processing and Cash App services. The technical root cause was related to a security certificate validation problem that temporarily prevented our payment systems from communicating properly with our databases.

## Our Immediate Response

Our monitoring systems first detected an issue at 20:00 UTC, and our engineering teams immediately started investigating the issue. At 20:20 Square and Cash App payments were impacted.

We rolled back the problematic certificate deployment at 20:42, allowing systems to reconnect and process transactions. Functionality began to recover at 20:58 and fully recovered at 21:03. There was continued impact to a handful of additional services, including reporting, that lasted for about another hour.

At 20:53, in response to the service disruption, we activated an internal lever to process payments offline. While sellers can always activate offline payments themselves, automatically transitioning sellers enables us to keep business running smoothly for sellers opted-in to accept offline payments. However, some sellers experienced issues accepting offline payments, and we have identified and are implementing fixes to ensure offline payments work smoother and take effect faster during incidents.

Throughout the incident we sent SMS updates to sellers [enrolled in notifications](https://squareup.com/dashboard/business/outage-notifications) and provided real-time updates on our [status page](https://issquareup.com/).

## Timeline of Critical Events

* All times are in UTC.
* 19:40 UTC: A routine update to a security certificate bundle was applied.
* 19:40-19:55 UTC: The updated certificate bundle was distributed to services.
* 19:58 UTC: External monitoring services detected potential issues.
* 20:00 UTC: Automated systems identify elevated errors, oncall engineers begin investigating.
* 20:20 UTC: Disruption spreads to Payment systems, additional engineers join investigation.
* 20:30 UTC: Incident severity was escalated to ensure a rapid response and [issquareup.com](https://issquareup.com/) activation.
* 20:42 UTC: Security certificate update rolled back.
* 20:53 UTC: Offline payments were enabled for card-present transactions to help stabilize processing.
* 20:58 UTC: Functionality begins to recover.
* 21:03 UTC: Functionality is largely returned to normal. Some lingering reporting issues remain.
* 22:13 UTC: The incident is fully resolved.

## Service Improvements

A portion of our services rely on a manual process to implement changes to service communication certificates. In order to prevent the issue from reoccurring we are eliminating this manual process and examining all processes that involve security certificates. As noted above, we are also working to improve offline payments for sellers in the coming weeks to ensure they can take payments in case of a disruption.

Finally, we will audit our deployment surfaces, reassess and improve Square’s offline playbook, and enhance degradation product experiences for both Square and Cash App as a core part of functionality.

## In Closing

We apologize for the disruption our outage might have created for you, your customers, and your employees. We will learn from this event and improve our systems and processes.

We appreciate your business and we are committed to doing better.
