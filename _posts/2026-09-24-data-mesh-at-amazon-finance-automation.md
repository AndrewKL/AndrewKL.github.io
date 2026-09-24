---
layout: post
title:  "A data mesh for Amazon's finance data"
date:   2026-09-24
excerpt: "I co-wrote a post on the AWS Big Data Blog about the data mesh we built at Amazon Finance Automation, where I was the lead developer."
---

I co-wrote a post on the AWS Big Data Blog with Nitin Arora, Kumar Satyen
Gaurav, Pradeep Misra and Rajesh Rao:
[How Amazon Finance Automation built a data mesh to support distributed data
ownership and centralize governance](https://aws.amazon.com/blogs/big-data/how-amazon-finance-automation-built-a-data-mesh-to-support-distributed-data-ownership-and-centralize-governance/).
I was the lead developer on it.

The problem is one every large finance organization eventually runs into.
Financial transactions used to fit on a single relational database. They
stopped fitting a long time ago, and the data ended up spread across a lot of
teams — each of which understood its own data best, and none of which wanted
to wait on a central team to publish it.

A data mesh is the usual answer: let the teams that own the data own the
publishing too, and centralize the governance rather than the data. The
interesting part is what that costs. Democratizing data product creation
removes the central bottleneck, but it hands every producer a new job —
onboarding a dataset to the catalog, then managing permissions on it, before
anyone else can use it. Get that wrong and adoption quietly stalls, because
the path of least resistance goes back to copying data around.

So most of the work went into making the obligations cheap. Producer accounts
sync their AWS Glue Data Catalog metadata into a central catalog. Lake
Formation in a central governance account holds the fine-grained permissions.
Consumers browse a front-end application, request access, and the request
routes to the data owner for approval, with a ticket behind it for auditing.
On approval, automation provisions an AWS RAM resource share, and the consumer
queries the data with Redshift Spectrum, Athena, EMR or QuickSight.

It currently hosts around 850 discoverable datasets and more than 300 curated
data products.

The post has the architecture diagrams and the details on the catalog sync,
which is the part that took the most care.
