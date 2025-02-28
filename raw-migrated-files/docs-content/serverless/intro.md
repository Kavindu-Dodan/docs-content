# Elastic Cloud Serverless [intro]

{{serverless-full}} is a fully managed solution that allows you to deploy and use Elastic for your use cases without managing the underlying infrastructure. It represents a shift in how you interact with {{es}} - instead of managing clusters, nodes, data tiers, and scaling, you create **serverless projects** that are fully managed and automatically scaled by Elastic. This abstraction of infrastructure decisions allows you to focus solely on gaining value and insight from your data.

{{serverless-full}} automatically provisions, manages, and scales your {{es}} resources based on your actual usage. Unlike traditional deployments where you need to predict and provision resources in advance, serverless adapts to your workload in real-time, ensuring optimal performance while eliminating the need for manual capacity planning.

Serverless projects use the core components of the {{stack}}, such as {{es}} and {{kib}}, and are based on an architecture that decouples compute and storage. Search and indexing operations are separated, which offers high flexibility for scaling your workloads while ensuring a high level of performance.

Elastic provides three serverless solutions available on {{ecloud}}:

* **/solutions/search.md[{{es-serverless}}]**: Build powerful applications and search experiences using a rich ecosystem of vector search capabilities, APIs, and libraries.
* **/solutions/observability.md[{{obs-serverless}}]**: Monitor your own platforms and services using powerful machine learning and analytics tools with your logs, metrics, traces, and APM data.
* **/solutions/security/elastic-security-serverless.md[{{sec-serverless}}]**: Detect, investigate, and respond to threats with SIEM, endpoint protection, and AI-powered analytics capabilities.

[Learn more about {{serverless-full}} in our blog](https://www.elastic.co/blog/elastic-cloud-serverless).


## Benefits of serverless projects [_benefits_of_serverless_projects]

**Management free.** Elastic manages the underlying Elastic cluster, so you can focus on your data. With serverless projects, Elastic is responsible for automatic upgrades, data backups, and business continuity.

**Autoscaled.** To meet your performance requirements, the system automatically adjusts to your workloads. For example, when you have a short time spike on the data you ingest, more resources are allocated for that period of time. When the spike is over, the system uses less resources, without any action on your end.

**Optimized data storage.** Your data is stored in cost-efficient, general storage. A cache layer is available on top of the general storage for recent and frequently queried data that provides faster search speed. The size of the cache layer and the volume of data it holds depend on [settings](../../../deploy-manage/deploy/elastic-cloud/project-settings.md) that you can configure for each project.

**Dedicated experiences.** All serverless solutions are built on the Elastic Search Platform and include the core capabilities of the Elastic Stack. They also each offer a distinct experience and specific capabilities that help you focus on your data, goals, and use cases.

**Pay per usage.** Each serverless project type includes product-specific and usage-based pricing.

**Data and performance control**. Control your project data and query performance against your project data. * Data. Choose the data you want to ingest and the method to ingest it. By default, data is stored indefinitely in your project, and you define the retention settings for your data streams. * Performance. For granular control over costs and query performance against your project data, serverless projects come with a set of predefined settings you can edit.


## Differences between serverless projects and hosted deployments on {{ecloud}} [general-what-is-serverless-elastic-differences-between-serverless-projects-and-hosted-deployments-on-ecloud]

You can run [hosted deployments](/deploy-manage/deploy/elastic-cloud/cloud-hosted.md) of the {{stack}} on {{ecloud}}. These hosted deployments provide more provisioning and advanced configuration options.

|     |     |     |
| --- | --- | --- |
| Option | Serverless | Hosted |
| **Cluster management** | Fully managed by Elastic. | You provision and manage your hosted clusters. Shared responsibility with Elastic. |
| **Scaling** | Autoscales out of the box. | Manual scaling or autoscaling available for you to enable. |
| **Upgrades** | Automatically performed by Elastic. | You choose when to upgrade. |
| **Pricing** | Individual per project type and based on your usage. | Based on deployment size and subscription level. |
| **Performance** | Autoscales based on your usage. | Manual scaling. |
| **Solutions** | Single solution per project. | Full Elastic Stack per deployment. |
| **User management** | Elastic Cloud-managed users. | Elastic Cloud-managed users and native Kibana users. |
| **API support** | Subset of [APIs](https://www.elastic.co/docs/api). | All Elastic APIs. |
| **Backups** | Projects automatically backed up by Elastic. | Your responsibility with Snapshot & Restore. |
| **Data retention** | Editable on data streams. | Index Lifecycle Management. |


## Answers to common serverless questions [general-what-is-serverless-elastic-answers-to-common-serverless-questions]

**Is there migration support between hosted deployments and serverless projects?**

Migration paths between hosted deployments and serverless projects are currently unsupported.

**How can I move data to or from serverless projects?**

We are working on data migration tools! In the interim, [use Logstash](asciidocalypse://docs/logstash/docs/reference/index.md) with Elasticsearch input and output plugins to move data to and from serverless projects.

**How does serverless ensure compatibility between software versions?**

Connections and configurations are unaffected by upgrades. To ensure compatibility between software versions, quality testing and API versioning are used.

**Can I convert a serverless project into a hosted deployment, or a hosted deployment into a serverless project?**

Projects and deployments are based on different architectures, and you are unable to convert.

**Can I convert a serverless project into a project of a different type?**

You are unable to convert projects into different project types, but you can create as many projects as you’d like. You will be charged only for your usage.

**How can I create serverless service accounts?**

Create API keys for service accounts in your serverless projects. Options to automate the creation of API keys with tools such as Terraform will be available in the future.

To raise a Support case with Elastic, raise a case for your subscription the same way you do today. In the body of the case, make sure to mention you are working in serverless to ensure we can provide the appropriate support.

**Where can I learn about pricing for serverless?**

See serverless pricing information for [Search](https://www.elastic.co/pricing/serverless-search), [Observability](https://www.elastic.co/pricing/serverless-observability), and [Security](https://www.elastic.co/pricing/serverless-security).

**Can I request backups or restores for my projects?**

It is not currently possible to request backups or restores for projects, but we are working on data migration tools to better support this.
