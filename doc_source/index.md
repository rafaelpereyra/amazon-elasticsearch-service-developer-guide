# Amazon Elasticsearch Service Developer Guide

-----
*****Copyright &copy; Amazon Web Services, Inc. and/or its affiliates. All rights reserved.*****

-----
Amazon's trademarks and trade dress may not be used in 
     connection with any product or service that is not Amazon's, 
     in any manner that is likely to cause confusion among customers, 
     or in any manner that disparages or discredits Amazon. All other 
     trademarks not owned by Amazon are the property of their respective
     owners, who may or may not be affiliated with, connected to, or 
     sponsored by Amazon.

-----
## Contents
+ [What is Amazon Elasticsearch Service?](what-is-amazon-elasticsearch-service.md)
+ [Getting started with Amazon Elasticsearch Service](es-gsg.md)
   + [Step 1: Create an Amazon ES domain](es-gsg-create-domain.md)
   + [Step 2: Upload data to Amazon ES for indexing](es-gsg-upload-data.md)
   + [Step 3: Search documents in Amazon ES](es-gsg-search.md)
   + [Step 4: Delete an Amazon ES domain](es-gsg-deleting.md)
+ [Creating and managing Amazon Elasticsearch Service domains](es-createupdatedomains.md)
   + [Making configuration changes in Amazon ES](es-managedomains-configuration-changes.md)
   + [Service software updates in Amazon Elasticsearch Service](es-service-software.md)
   + [Notifications in Amazon Elasticsearch Service](es-managedomains-notifications.md)
   + [Configuring a multi-AZ domain in Amazon Elasticsearch Service](es-managedomains-multiaz.md)
   + [Launching your Amazon Elasticsearch Service domains using a VPC](es-vpc.md)
   + [Monitoring Amazon Elasticsearch Service cluster metrics with Amazon CloudWatch](es-managedomains-cloudwatchmetrics.md)
   + [Configuring logs in Amazon Elasticsearch Service](es-createdomain-configure-slow-logs.md)
   + [Creating index snapshots in Amazon Elasticsearch Service](es-managedomains-snapshots.md)
   + [Upgrading Elasticsearch](es-version-migration.md)
   + [Creating a custom endpoint for Amazon Elasticsearch Service](es-customendpoint.md)
   + [Auto-Tune for Amazon Elasticsearch Service](auto-tune.md)
   + [Tagging Amazon Elasticsearch Service domains](es-managedomains-awsresourcetagging.md)
+ [Security in Amazon Elasticsearch Service](security.md)
   + [Data protection in Amazon Elasticsearch Service](es-data-protection.md)
      + [Encryption of data at rest for Amazon Elasticsearch Service](encryption-at-rest.md)
      + [Node-to-node encryption for Amazon Elasticsearch Service](ntn.md)
   + [Identity and Access Management in Amazon Elasticsearch Service](es-ac.md)
   + [Fine-grained access control in Amazon Elasticsearch Service](fgac.md)
   + [Managing audit logs in Amazon Elasticsearch Service](audit-logs.md)
   + [Monitoring Amazon Elasticsearch Service configuration API activity](es-managedomains-cloudtrailauditing.md)
   + [Compliance validation for Amazon Elasticsearch Service](es-compliance.md)
   + [Resilience in Amazon Elasticsearch Service](disaster-recovery-resiliency.md)
   + [Infrastructure security in Amazon Elasticsearch Service](infrastructure-security.md)
   + [SAML authentication for Kibana](saml.md)
   + [Configuring Amazon Cognito authentication for Kibana](es-cognito-auth.md)
   + [Using service-linked roles to provide Amazon Elasticsearch Service access to resources](slr-es.md)
+ [Sample code for Amazon Elasticsearch Service](es-samplecode.md)
   + [Signing HTTP requests to Amazon Elasticsearch Service](es-request-signing.md)
   + [Compressing HTTP requests in Amazon Elasticsearch Service](gzip.md)
   + [Using the AWS SDKs to interact with Amazon Elasticsearch Service](es-configuration-samples.md)
+ [Indexing data in Amazon Elasticsearch Service](es-indexing.md)
   + [Loading streaming data into Amazon Elasticsearch Service](es-aws-integrations.md)
   + [Loading data into Amazon Elasticsearch Service with Logstash](es-managedomains-logstash.md)
+ [Searching data in Amazon Elasticsearch Service](es-searching.md)
   + [Custom packages for Amazon Elasticsearch Service](custom-packages.md)
   + [Querying your Amazon Elasticsearch Service data with SQL](sql-support.md)
   + [Querying Amazon Elasticsearch Service data using Piped Processing Language](ppl-support.md)
   + [k-Nearest Neighbor (k-NN) search in Amazon Elasticsearch Service](knn.md)
   + [Cross-cluster search for Amazon Elasticsearch Service](cross-cluster-search.md)
   + [Learning to Rank for Amazon Elasticsearch Service](learning-to-rank.md)
   + [Asynchronous search for Amazon Elasticsearch Service](asynchronous-search.md)
+ [Using Kibana with Amazon Elasticsearch Service](es-kibana.md)
+ [Managing indices in Amazon Elasticsearch Service](managing-indices.md)
   + [UltraWarm storage for Amazon Elasticsearch Service](ultrawarm.md)
   + [Cold storage for Amazon Elasticsearch Service](cold-storage.md)
   + [Index State Management in Amazon Elasticsearch Service](ism.md)
   + [Summarizing indices in Amazon Elasticsearch Service with index rollups](rollup.md)
   + [Using Curator to rotate data in Amazon Elasticsearch Service](curator.md)
   + [Migrating Amazon Elasticsearch Service indices using remote reindex](remote-reindex.md)
+ [Monitoring data in Amazon Elasticsearch Service](monitoring-data.md)
   + [Configuring alerts in Amazon Elasticsearch Service](alerting.md)
   + [Anomaly detection in Amazon Elasticsearch Service](ad.md)
   + [Trace Analytics for Amazon Elasticsearch Service](trace-analytics.md)
+ [Best practices for Amazon Elasticsearch Service](aes-bp.md)
   + [Sizing Amazon Elasticsearch Service domains](sizing-domains.md)
   + [Petabyte scale for Amazon Elasticsearch Service](petabyte-scale.md)
   + [Dedicated master nodes in Amazon Elasticsearch Service](es-managedomains-dedicatedmasternodes.md)
   + [Recommended CloudWatch alarms for Amazon Elasticsearch Service](cloudwatch-alarms.md)
+ [General reference for Amazon Elasticsearch Service](aes-genref.md)
   + [Supported instance types in Amazon Elasticsearch Service](aes-supported-instance-types.md)
   + [Features by Elasticsearch version](aes-features-by-version.md)
   + [Plugins by Elasticsearch version](aes-supported-plugins.md)
   + [Supported Elasticsearch operations](aes-supported-es-operations.md)
   + [Amazon Elasticsearch Service limits](aes-limits.md)
   + [Reserved Instances in Amazon Elasticsearch Service](aes-ri.md)
   + [Other supported resources in Amazon Elasticsearch Service](aes-supported-resources.md)
+ [Amazon Elasticsearch Service Tutorials](tutorials.md)
   + [Migrating to Amazon Elasticsearch Service](migration.md)
   + [Creating a search application with Amazon Elasticsearch Service](search-example.md)
   + [Visualizing customer support calls with Amazon Elasticsearch Service and Kibana](es-walkthrough.md)
+ [Troubleshooting Amazon Elasticsearch Service](aes-handling-errors.md)
+ [Configuration API reference for Amazon Elasticsearch Service](es-configuration-api.md)
+ [Document history for Amazon Elasticsearch Service](release-notes.md)
+ [AWS glossary](glossary.md)