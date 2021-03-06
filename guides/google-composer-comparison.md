---
title: "Astronomer vs. Google Cloud Composer"
description: "A high-level comparison of Astronomer and Google Cloud Composer"
date: 2018-08-30T00:00:00.000Z
slug: "google-composer-comparison"
heroImagePath: null
tags: ["Astronomer Platform", "Google Composer"]
---

2019 may just be the year of [Apache Airflow](https://github.com/apache/airflow). Since the project started at Airbnb in 2014, adoption for Airflow as a reliable and scalable scheduler has grown quickly while parallel offerings like [Luigi](https://github.com/spotify/luigi) and [Azkaban](https://azkaban.github.io/) have slowed in development and adoption.

Astronomer was born from that need and in 2018 [announced Astronomer Enterprise](https://www.astronomer.io/blog/announcing-astronomer-enterprise-edition/) - a self-hosted Airflow management solution. We've since also developed [Astronomer Cloud](https://astronomer.io/cloud), a fully-managed Airflow solution for those looking to stay abstracted from all-things DevOps.

That same year, Google announced their entry into the market with [Cloud Composer](https://cloud.google.com/composer/), a parallel "Airflow as a Service" offering.

While we're incredibly excited that the commercialization of the Apache Airflow project is growing, we want to guide users deciding betweeen Astronomer and Composer as their managed Airflow service of choice. Read below for a thorough comparison of both offerings.

**Note**: As you might expect, we’re not heavy users of Composer and this comparison is derived from a combination of customer feedback and what we could find in public documentation and associated online communities (e.g. StackOverflow, Slack, etc.) at the time of writing. If anything appears incorrect or misleading, reach out to us at humans@astronomer.io and we'll work with you to adjust relevant content.

## At a Glance

||Cloud Composer|Astronomer Cloud|Astronomer Enterprise|
|-|--------------|----------------|---------------------|
|Cost|~$300 base|~ $110 base running Local Executor|Annual license pending cluster CPU and support structure|
|Hosting|Managed service hosted in Google's cloud environment|Managed service hosted in Astronomer's Cloud environment|Self-hosted service hosted on your own Kubernetes cluster|
|Monitoring and Logging|[Google Stackdriver](https://cloud.google.com/stackdriver/)|Deployment level metrics and logging in the Astronomer UI |Elasticsearch, Fluentd, Kibana (EFK) stack to track resource usage across all Airflow deployments|
|Support|Community support via Stack Overflow|Standard ticketing system, [community forum](https:/forum.astronomer.io), and premium real-time support option via a private Slack Channel |24x7 Business-Critical SLAs available|
|Training|None|[Astronomer SpaceCamp](https://astronomer.io/spacecamp) for on-site Airflow training |[Astronomer SpaceCamp](https://astronomer.io/spacecamp) for on-site Airflow training|


### Cost

#### Google Cloud Composer

It's hard to land on a specific cost for Cloud Composer, as Google measures the resources your deployments use and add the total cost of your Airflow deployments onto your wider GCP bill.

Based on the [estimates provided](https://cloud.google.com/composer/pricing), a single, full-time instance of Composer:

- Costs ~$300/month
- Assumes that all 3 workers use the relatively low-powered `n1-standard-1` (1 vCPU; 3.75GB) machine type
- Does not include Storage and Compute Engine costs

#### Astronomer

Astronomer Cloud is billed based on exact resources used per deployment. On Astronomer, you're free to ajust resource allocation to each Airflow component (Scheduler, Webserver and Celery Workers) to best fit both your budget and use case.

Based on our default resource allocation, it breaks down to:

- $110/mo for a default deployment with Airflow's Local Executor
- $250/mo for a default deployment with the Celery executor

On Astronomer Cloud v0.9, the node limits for any single task (based on Google's standard-16 machine type) are:

- 58 GB of Memory/RAM
- 15 CPU

For more details, check out our [pricing doc](https://astronomer.io/docs/pricing).

Astronomer Enterprise is priced based on an annual subscription license. Please [contact us](https://astronomer.io/contact) if you'd like to see a pricing sheet.

### System Packages

#### Astronomer

Astronomer's base Airflow images are [Alpine-based](https://alpinelinux.org/about/) (a distribution of Linux), which we leverage for its small size and resource efficiency (3.98MB)

With that said, we've recently published an experimental Debian image that we've offered to interested customers. Despite its larger size (123MB), we wanted our users to have that option for better package dependency compatibility.

All of our Docker Images are open-sourced and can be found [here](https://hub.docker.com/r/astronomerinc/ap-airflow/). Source code for them [here](https://github.com/astronomer/astronomer/tree/release-0.7/docker/airflow).

For a more in-depth comparison (and how this translates to cost when deployed to a cloud that charges for transfer cost), check out this [great post](https://nickjanetakis.com/blog/the-3-biggest-wins-when-using-alpine-as-a-base-docker-image) by Nick Janetakis.

### Deployment

Both Astronomer and Composer currently use a fairly similar setup for executing tasks. Each Airflow component (Airflow (Webserver, Scheduler, and Workers) are deployed as pods on Kubernetes using the Celery Executor. The largest difference in deployment between Astronomer and Cloud Composer is where these pods run.

#### Astronomer

While Google Cloud Composer only runs on Google's Cloud Platform, Astronomer Cloud runs on a [GKE](https://cloud.google.com/kubernetes-engine/)cluster fully managed and hosted by the Astronomer team.

Astronomer Enterprise is entirely Cloud agnostic and can be installed on any Kubernetes Cluster. We recommend using a managed Kubernetes service from any of the large cloud providers ([Azure AKS](https://azure.microsoft.com/en-us/services/kubernetes-service/), [AWS EKS](https://aws.amazon.com/eks/), [Digital Ocean](https://www.digitalocean.com/products/kubernetes/), [Pivotal PKS](https://pivotal.io/platform/pivotal-container-service), and even [GCP GKE](https://cloud.google.com/kubernetes-engine/)), but you'd be equally able to run our platform on any flavor of on-prem Kubernetes.

### Monitoring and Logging

**Google Cloud Composer**

As of the [1.0.0 release](https://cloud.google.com/composer/docs/release-notes#july_19_2018_composer-100-airflow-190), logging in Composer is handled by Stackdriver and based on [fluentd](https://www.fluentd.org).

**Astronomer**

Logging in Astronomer is handled by [Elasticsearch](https://www.elastic.co/products/elasticsearch).

Astronomer Cloud has leverages a few features on the logging and metrics front. 

#### 1. Real-time Scheduler, Webserver, Worker Logs

Astronomer pulls searchable, real-time logs from your Airflow Scheduler, Webserver, and Worker directly into the Astronomer UI.

![logging](https://assets2.astronomer.io/main/guides/logging.png)

#### 2. Deployment Level Metrics

As of Astronomer v0.9, we've also pulled a variety of deployment level metrics into the Astronomer UI, including:

- Container status
- Deployment Health
- Task success and failure rates
- CPU and Memory usage
- Database connections

![logging](https://assets2.astronomer.io/main/blog/metrics.gif)

#### 3. Cluster Wide Metrics (*Enterprise only*)

Astronomer Enterprise offers the same logging and metrics features incorporated in Astronomer Cloud but also comes with access to a Grafana/Prometheus monitoring stack (the basis for the deployment level metrics above) to keep an eye on health and metadata *across* Airflow deployments.

![grafana](https://assets2.astronomer.io/main/blog/grafana-dashboard.png)

### Airflow Executor Support

#### Google Cloud Composer

Google Cloud Composer supports both the Celery and Local Executors, but does not yet support the recently developed Kubernetes Executor.

#### Astronomer

As of v0.9, Astronomer has initial support for the Kubernetes Executor. While it's not scale-to-zero to start with, that's most certainly a reality we're working towards.

To read more, refer to [Airflow Executors: Explained](https://www.astronomer.io/guides/airflow-executors-explained/).

### Support and Training

#### Google Cloud Composer

Composer's support is largely community-based. Users can post questions to forums like [StackOverflow](https://stackoverflow.com) or use the generic [Airflow support system established via Slack](https://apache-airflow-slack.herokuapp.com/).

#### Astronomer

Astronomer comes out-of-the-box with support plans that give you access to a wealth of Airflow experts. We’ve been using Airflow internally for 2+ years and have helped many companies improve their data processes using Airflow.

We have multiple Airflow committers on our team who are constantly working to improve the core OSS project, and offer on-site Airflow trainings that are product agnostic ([SpaceCamp](https://www.astronomer.io/spacecamp/))

For more info, feel free to check out our [support plans](https://astronomer.io/pricing) or [reach out directly](https://www.astronomer.io/spacecamp/#request-spacecamp) to request a customized SpaceCamp curriculum for your team. 