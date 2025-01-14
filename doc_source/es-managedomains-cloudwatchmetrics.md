# Monitoring Amazon Elasticsearch Service cluster metrics with Amazon CloudWatch<a name="es-managedomains-cloudwatchmetrics"></a>

## Interpreting health dashboards<a name="es-managedomains-cloudwatchmetrics-box-charts"></a>

The **Instance health** tab in the Amazon ES console uses box charts to provide at\-a\-glance visibility into the health of each Elasticsearch node\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/box-charts.png)
+ Each colored box shows the range of values for the node over the specified time period\.
+ Blue boxes represent values that are consistent with other nodes\. Red boxes represent outliers\.
+ The white line within each box shows the node's current value\.
+ The “whiskers” on either side of each box show the minimum and maximum values for all nodes over the time period\.

Amazon ES domains send performance metrics to Amazon CloudWatch every minute\. If you use General Purpose or Magnetic EBS volumes, the EBS volume metrics update only every five minutes\. To view these metrics, use the **Cluster health** and **Instance health** tabs in the Amazon Elasticsearch Service console\. The metrics are provided at no extra charge\.

If you make configuration changes to your domain, the list of individual instances in the **Cluster health** and **Instance health** tabs often double in size for a brief period before returning to the correct number\. For an explanation of this behavior, see [Making configuration changes in Amazon ES](es-managedomains-configuration-changes.md)\.

All metrics are in the `AWS/ES` namespace\. Metrics for individual nodes are in the `ClientId, DomainName, NodeId` dimension\. Cluster metrics are in the `Per-Domain, Per-Client Metrics` dimension\. Some node metrics are aggregated at the cluster level and thus included in both dimensions\. Shart metrics are in the `ClientId, DomainName, NodeId, ShardRole` dimension\. The service archives metrics for two weeks before discarding them\.
+ [Cluster metrics](#es-managedomains-cloudwatchmetrics-cluster-metrics)
+ [Dedicated master node metrics](#es-managedomains-cloudwatchmetrics-master-node-metrics)
+ [EBS volume metrics](#es-managedomains-cloudwatchmetrics-master-ebs-metrics)
+ [Instance metrics](#es-managedomains-cloudwatchmetrics-instance-metrics)
+ [UltraWarm metrics](#es-managedomains-cloudwatchmetrics-uw)
+ [Cold storage metrics](#es-managedomains-cloudwatchmetrics-coldstorage)
+ [Alerting metrics](#es-managedomains-cloudwatchmetrics-alerting)
+ [Anomaly detection metrics](#es-managedomains-cloudwatchmetrics-anomaly-detection)
+ [Asynchronous search metrics](#es-managedomains-cloudwatchmetrics-asynchronous-search)
+ [SQL metrics](#es-managedomains-cloudwatchmetrics-sql)
+ [k\-NN metrics](#es-managedomains-cloudwatchmetrics-knn)
+ [Cross\-cluster search metrics](#es-managedomains-cloudwatchmetrics-cross-cluster-search)

## Cluster metrics<a name="es-managedomains-cloudwatchmetrics-cluster-metrics"></a>

Amazon Elasticsearch Service provides the following metrics for clusters\.


| Metric | Description | 
| --- | --- | 
| ClusterStatus\.green |  A value of 1 indicates that all index shards are allocated to nodes in the cluster\. Relevant statistics: Maximum  | 
| ClusterStatus\.yellow | A value of 1 indicates that the primary shards for all indices are allocated to nodes in the cluster, but replica shards for at least one index are not\. For more information, see [Yellow cluster status](aes-handling-errors.md#aes-handling-errors-yellow-cluster-status)\.Relevant statistics: Maximum | 
| ClusterStatus\.red |  A value of 1 indicates that the primary and replica shards for at least one index are not allocated to nodes in the cluster\. For more information, see [Red cluster status](aes-handling-errors.md#aes-handling-errors-red-cluster-status)\. Relevant statistics: Maximum  | 
| Nodes |  The number of nodes in the Amazon ES cluster, including dedicated master nodes and UltraWarm nodes\. For more information, see [Making configuration changes in Amazon ES](es-managedomains-configuration-changes.md)\. Relevant statistics: Maximum  | 
| SearchableDocuments |  The total number of searchable documents across all data nodes in the cluster\. Relevant statistics: Minimum, Maximum, Average  | 
| DeletedDocuments |  The total number of documents marked for deletion across all data nodes in the cluster\. These documents no longer appear in search results, but Elasticsearch only removes deleted documents from disk during segment merges\. This metric increases after delete requests and decreases after segment merges\. Relevant statistics: Minimum, Maximum, Average  | 
| CPUUtilization |  The percentage of CPU usage for data nodes in the cluster\. Maximum shows the node with the highest CPU usage\. Average represents all nodes in the cluster\. This metric is also available for individual nodes\. Relevant statistics: Maximum, Average  | 
| FreeStorageSpace |  The free space for data nodes in the cluster\. `Sum` shows total free space for the cluster, but you must leave the period at one minute to get an accurate value\. `Minimum` and `Maximum` show the nodes with the least and most free space, respectively\. This metric is also available for individual nodes\. Amazon ES throws a `ClusterBlockException` when this metric reaches `0`\. To recover, you must either delete indices, add larger instances, or add EBS\-based storage to existing instances\. To learn more, see [Lack of available storage space](aes-handling-errors.md#aes-handling-errors-watermark)\. The Amazon ES console displays this value in GiB\. The Amazon CloudWatch console displays it in MiB\.  `FreeStorageSpace` will always be lower than the value that the Elasticsearch `_cluster/stats` API provides\. Amazon ES reserves a percentage of the storage space on each instance for internal operations\.  Relevant statistics: Minimum, Maximum, Average, Sum  | 
| ClusterUsedSpace |  The total used space for the cluster\. You must leave the period at one minute to get an accurate value\. The Amazon ES console displays this value in GiB\. The Amazon CloudWatch console displays it in MiB\. Relevant statistics: Minimum, Maximum  | 
| ClusterIndexWritesBlocked |  Indicates whether your cluster is accepting or blocking incoming write requests\. A value of 0 means that the cluster is accepting requests\. A value of 1 means that it is blocking requests\. Some common factors include the following: `FreeStorageSpace` is too low or `JVMMemoryPressure` is too high\. To alleviate this issue, consider adding more disk space or scaling your cluster\. Relevant statistics: Maximum  | 
| JVMMemoryPressure |  The maximum percentage of the Java heap used for all data nodes in the cluster\. Amazon ES uses half of an instance's RAM for the Java heap, up to a heap size of 32 GiB\. You can scale instances vertically up to 64 GiB of RAM, at which point you can scale horizontally by adding instances\. See [Recommended CloudWatch alarms for Amazon Elasticsearch Service](cloudwatch-alarms.md)\. Relevant statistics: Maximum  | 
| AutomatedSnapshotFailure |  The number of failed automated snapshots for the cluster\. A value of `1` indicates that no automated snapshot was taken for the domain in the previous 36 hours\. Relevant statistics: Minimum, Maximum  | 
| CPUCreditBalance |  The remaining CPU credits available for data nodes in the cluster\. A CPU credit provides the performance of a full CPU core for one minute\. For more information, see [CPU credits](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-credits-baseline-concepts.html) in the *Amazon EC2 Developer Guide*\. This metric is available only for the T2 instance types\. Relevant statistics: Minimum  | 
| KibanaHealthyNodes |  A health check for Kibana\. If the minimum, maximum, and average are all equal to 1, Kibana is behaving normally\. If you have 10 nodes with a maximum of 1, minimum of 0, and average of 0\.7, this means 7 nodes \(70%\) are healthy and 3 nodes \(30%\) are unhealthy\. Relevant statistics: Minimum, Maximum, Average  | 
| KMSKeyError |  A value of 1 indicates that the KMS customer master key used to encrypt data at rest has been disabled\. To restore the domain to normal operations, re\-enable the key\. The console displays this metric only for domains that encrypt data at rest\. Relevant statistics: Minimum, Maximum  | 
| KMSKeyInaccessible |  A value of 1 indicates that the KMS customer master key used to encrypt data at rest has been deleted or revoked its grants to Amazon ES\. You can't recover domains that are in this state\. If you have a manual snapshot, though, you can use it to migrate the domain's data to a new domain\. The console displays this metric only for domains that encrypt data at rest\. Relevant statistics: Minimum, Maximum  | 
| InvalidHostHeaderRequests |  The number of HTTP requests made to the Elasticsearch cluster that included an invalid \(or missing\) host header\. Valid requests include the domain hostname as the host header value\. Amazon ES rejects invalid requests for public access domains that don't have a restrictive access policy\. We recommend applying a restrictive access policy to all domains\. If you see large values for this metric, confirm that your Elasticsearch clients include the domain hostname \(and not, for example, its IP address\) in their requests\. Relevant statistics: Sum  | 
| ElasticsearchRequests |  The number of requests made to the Elasticsearch cluster\. Relevant statistics: Sum  | 
| 2xx, 3xx, 4xx, 5xx |  The number of requests to the domain that resulted in the given HTTP response code \(2*xx*, 3*xx*, 4*xx*, 5*xx*\)\. Relevant statistics: Sum  | 

## Dedicated master node metrics<a name="es-managedomains-cloudwatchmetrics-master-node-metrics"></a>

Amazon Elasticsearch Service provides the following metrics for [dedicated master nodes](es-managedomains-dedicatedmasternodes.md)\.


| Metric | Description | 
| --- | --- | 
| MasterCPUUtilization |  The maximum percentage of CPU resources used by the dedicated master nodes\. We recommend increasing the size of the instance type when this metric reaches 60 percent\. Relevant statistics: Average  | 
| MasterFreeStorageSpace |  This metric is not relevant and can be ignored\. The service does not use master nodes as data nodes\.  | 
| MasterJVMMemoryPressure |  The maximum percentage of the Java heap used for all dedicated master nodes in the cluster\. We recommend moving to a larger instance type when this metric reaches 85 percent\. Relevant statistics: Maximum  | 
| MasterCPUCreditBalance |  The remaining CPU credits available for dedicated master nodes in the cluster\. A CPU credit provides the performance of a full CPU core for one minute\. For more information, see [CPU credits](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-credits-baseline-concepts.html) in the *Amazon EC2 Developer Guide*\. This metric is available only for the T2 instance types\. Relevant statistics: Minimum  | 
| MasterReachableFromNode |  A health check for `MasterNotDiscovered` exceptions\. A value of 1 indicates normal behavior\. A value of 0 indicates that `/_cluster/health/` is failing\. Failures mean that the master node stopped or is not reachable\. They are usually the result of a network connectivity issue or AWS dependency problem\. Relevant statistics: Minimum  | 
| MasterSysMemoryUtilization |  The percentage of the master node's memory that is in use\. Relevant statistics: Maximum  | 

## EBS volume metrics<a name="es-managedomains-cloudwatchmetrics-master-ebs-metrics"></a>

Amazon Elasticsearch Service provides the following metrics for EBS volumes\.


| Metric | Description | 
| --- | --- | 
| ReadLatency |  The latency, in seconds, for read operations on EBS volumes\. Relevant statistics: Minimum, Maximum, Average  | 
| WriteLatency |  The latency, in seconds, for write operations on EBS volumes\. Relevant statistics: Minimum, Maximum, Average  | 
| ReadThroughput |  The throughput, in bytes per second, for read operations on EBS volumes\. Relevant statistics: Minimum, Maximum, Average  | 
| WriteThroughput |  The throughput, in bytes per second, for write operations on EBS volumes\. Relevant statistics: Minimum, Maximum, Average  | 
| DiskQueueDepth |  The number of pending input and output \(I/O\) requests for an EBS volume\. Relevant statistics: Minimum, Maximum, Average  | 
| ReadIOPS |  The number of input and output \(I/O\) operations per second for read operations on EBS volumes\. Relevant statistics: Minimum, Maximum, Average  | 
| WriteIOPS |  The number of input and output \(I/O\) operations per second for write operations on EBS volumes\. Relevant statistics: Minimum, Maximum, Average  | 

## Instance metrics<a name="es-managedomains-cloudwatchmetrics-instance-metrics"></a>

Amazon Elasticsearch Service provides the following metrics for each instance in a domain\. Amazon ES also aggregates these instance metrics to provide insight into overall cluster health\. You can verify this behavior using the **Sample Count** statistic in the console\. Note that each metric in the following table has relevant statistics for the node *and* the cluster\.

**Important**  
Different versions of Elasticsearch use different thread pools to process calls to the `_index` API\. Elasticsearch 1\.5 and 2\.3 use the index thread pool\. Elasticsearch 5\.*x*, 6\.0, and 6\.2 use the bulk thread pool\. 6\.3 and later use the write thread pool\. Currently, the Amazon ES console doesn't include a graph for the bulk thread pool\.  
Use `GET _cluster/settings?include_defaults=true` to check thread pool and queue sizes for your cluster\.


| Metric | Description | 
| --- | --- | 
| IndexingLatency |  The average time, in milliseconds, that it takes a shard to complete an indexing operation\. Relevant node statistics: Average Relevant cluster statistics: Average, Maximum  | 
| IndexingRate |  The number of indexing operations per minute\. A single call to the `_bulk` API that adds two documents and updates two counts as four operations, which might be spread across one or more nodes\. If that index has one or more replicas, other nodes in the cluster also record a total of four indexing operations\. Document deletions do not count towards this metric\. Relevant node statistics: Average Relevant cluster statistics: Average, Maximum, Sum  | 
| SearchLatency |  The average time, in milliseconds, that it takes a shard on a data node to complete a search operation\. Relevant node statistics: Average Relevant cluster statistics: Average, Maximum  | 
| SearchRate |  The total number of search requests per minute for all shards on a data node\. A single call to the `_search` API might return results from many different shards\. If five of these shards are on one node, the node would report 5 for this metric, even though the client only made one request\. Relevant node statistics: Average Relevant cluster statistics: Average, Maximum, Sum  | 
| SysMemoryUtilization |  The percentage of the instance's memory that is in use\. High values for this metric are normal and usually do not represent a problem with your cluster\. For a better indicator of potential performance and stability issues, see the `JVMMemoryPressure` metric\. Relevant node statistics: Minimum, Maximum, Average Relevant cluster statistics: Minimum, Maximum, Average  | 
| JVMGCYoungCollectionCount |  The number of times that "young generation" garbage collection has run\. A large, ever\-growing number of runs is a normal part of cluster operations\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average  | 
| JVMGCYoungCollectionTime |  The amount of time, in milliseconds, that the cluster has spent performing "young generation" garbage collection\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average  | 
| JVMGCOldCollectionCount |  The number of times that "old generation" garbage collection has run\. In a cluster with sufficient resources, this number should remain small and grow infrequently\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average  | 
| JVMGCOldCollectionTime |  The amount of time, in milliseconds, that the cluster has spent performing "old generation" garbage collection\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average  | 
| KibanaConcurrentConnections |  The number of active concurrent connections to Kibana\. If this number is consistently high, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average  | 
| KibanaHealthyNode |  A health check for the individual Kibana node\. A value of 1 indicates normal behavior\. A value of 0 indicates that Kibana is inaccessible\.  Relevant node statistics: Minimum Relevant cluster statistics: Minimum, Maximum, Average  | 
| KibanaHeapTotal |  The amount of heap memory allocated to Kibana in MiB\. Different EC2 instance types can impact the exact memory allocation\.  Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average  | 
| KibanaHeapUsed |  The absolute amount of heap memory used by Kibana in MiB\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average  | 
| KibanaHeapUtilization |  The maximum percentage of available heap memory used by Kibana\. If this value increases above 80%, consider scaling your cluster\.  Relevant node statistics: Maximum Relevant cluster statistics: Minimum, Maximum, Average  | 
| KibanaOS1MinuteLoad |  The one\-minute CPU load average for Kibana\. The CPU load should ideally stay below 1\.00\. While temporary spikes are fine, we recommend increasing the size of the instance type if this metric is consistently above 1\.00\. Relevant node statistics: Average Relevant cluster statistics: Average, Maximum  | 
| KibanaRequestTotal |  The total count of HTTP requests made to Kibana\. If your system is slow or you see high numbers of Kibana requests, consider increasing the size of the instance type\. Relevant node statistics: Sum Relevant cluster statistics: Sum  | 
| KibanaResponseTimesMaxInMillis |  The maximum amount of time, in milliseconds, that it takes for Kibana to respond to a request\. If requests consistently take a long time to return results, consider increasing the size of the instance type\. Relevant node statistics: Maximum Relevant cluster statistics: Maximum, Average  | 
| ThreadpoolForce\_mergeQueue |  The number of queued tasks in the force merge thread pool\. If the queue size is consistently high, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average  | 
| ThreadpoolForce\_mergeRejected |  The number of rejected tasks in the force merge thread pool\. If this number continually grows, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum  | 
| ThreadpoolForce\_mergeThreads |  The size of the force merge thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum  | 
| ThreadpoolIndexQueue |  The number of queued tasks in the index thread pool\. If the queue size is consistently high, consider scaling your cluster\. The maximum index queue size is 200\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average  | 
| ThreadpoolIndexRejected |  The number of rejected tasks in the index thread pool\. If this number continually grows, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum  | 
| ThreadpoolIndexThreads |  The size of the index thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum  | 
| ThreadpoolSearchQueue |  The number of queued tasks in the search thread pool\. If the queue size is consistently high, consider scaling your cluster\. The maximum search queue size is 1,000\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average  | 
| ThreadpoolSearchRejected |  The number of rejected tasks in the search thread pool\. If this number continually grows, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum  | 
| ThreadpoolSearchThreads |  The size of the search thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum  | 
| ThreadpoolBulkQueue |  The number of queued tasks in the bulk thread pool\. If the queue size is consistently high, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum, Maximum, Average  | 
| ThreadpoolBulkRejected |  The number of rejected tasks in the bulk thread pool\. If this number continually grows, consider scaling your cluster\. Relevant node statistics: Maximum Relevant cluster statistics: Sum  | 
| ThreadpoolBulkThreads |  The size of the bulk thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum  | 
| ThreadpoolWriteThreads |  The size of the write thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum  | 
| ThreadpoolWriteQueue |  The number of queued tasks in the write thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum  | 
| ThreadpoolWriteRejected |  The number of rejected tasks in the write thread pool\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum  Because the default write queue size was increased from 200 to 10000 in version 7\.9, this metric is no longer the only indicator of rejections from Amazon ES\. Use the `CoordinatingWriteRejected`, `PrimaryWriteRejected`, and `ReplicaWriteRejected` metrics to monitor rejections in versions 7\.9 and later\.   | 
| CoordinatingWriteRejected |  The total number of rejections happened on the coordinating node due to indexing pressure since the last Amazon ES process startup\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum This metric is available in version 7\.9 and above\.  | 
| PrimaryWriteRejected |  The total number of rejections happened on the primary shards due to indexing pressure since the last Amazon ES process startup\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum This metric is available in version 7\.9 and above\.  | 
| ReplicaWriteRejected |  The total number of rejections happened on the replica shards due to indexing pressure since the last Amazon ES process startup\. Relevant node statistics: Maximum Relevant cluster statistics: Average, Sum This metric is available in version 7\.9 and above\.  | 

## UltraWarm metrics<a name="es-managedomains-cloudwatchmetrics-uw"></a>

Amazon Elasticsearch Service provides the following metrics for [UltraWarm](ultrawarm.md) nodes\.


| Metric | Description | 
| --- | --- | 
| WarmCPUUtilization |  The percentage of CPU usage for UltraWarm nodes in the cluster\. Maximum shows the node with the highest CPU usage\. Average represents all UltraWarm nodes in the cluster\. This metric is also available for individual UltraWarm nodes\. Relevant statistics: Maximum, Average  | 
| WarmFreeStorageSpace |  The amount of free warm storage space in MiB\. Because UltraWarm uses Amazon S3 rather than attached disks, `Sum` is the only relevant statistic\. You must leave the period at one minute to get an accurate value\. Relevant statistics: Sum  | 
| WarmJVMMemoryPressure |  The maximum percentage of the Java heap used for the UltraWarm nodes\. Relevant statistics: Maximum  | 
| WarmSearchableDocuments |  The total number of searchable documents across all warm indices in the cluster\. You must leave the period at one minute to get an accurate value\. Relevant statistics: Sum  | 
|  WarmSearchLatency  |  The average time, in milliseconds, that it takes a shard on an UltraWarm node to complete a search operation\. Relevant node statistics: Average Relevant cluster statistics: Average, Maximum  | 
|  WarmSearchRate  |  The total number of search requests per minute for all shards on an UltraWarm node\. A single call to the `_search` API might return results from many different shards\. If five of these shards are on one node, the node would report 5 for this metric, even though the client only made one request\. Relevant node statistics: Average Relevant cluster statistics: Average, Maximum, Sum  | 
| WarmStorageSpaceUtilization |  The total amount of warm storage space, in MiB, that the cluster is using\.  Relevant statistics: Maximum  | 
|  HotStorageSpaceUtilization  |  The total amount of hot storage space that the cluster is using\.  Relevant statistics: Maximum  | 
| WarmSysMemoryUtilization |  The percentage of the warm node's memory that is in use\. Relevant statistics: Maximum  | 
|  HotToWarmMigrationQueueSize  |  The number of indices currently waiting to migrate from hot to warm storage\. Relevant statistics: Maximum  | 
|  WarmToHotMigrationQueueSize  |  The number of indices currently waiting to migrate from warm to hot storage\. Relevant statistics: Maximum  | 
|  HotToWarmMigrationFailureCount  |  The total number of failed hot to warm migrations\. Relevant statistics: Sum  | 
|  HotToWarmMigrationForceMergeLatency  |  The average latency of the force merge stage of the migration process\. If this stage consistently takes too long, consider increasing `index.ultrawarm.migration.force_merge.max_num_segments`\. Relevant statistics: Average  | 
|  HotToWarmMigrationSnapshotLatency  |  The average latency of the snapshot stage of the migration process\. If this stage consistently takes too long, ensure that your shards are appropriately sized and distributed throughout the cluster\. Relevant statistics: Average  | 
|  HotToWarmMigrationProcessingLatency  |  The average latency of successful hot to warm migrations, *not* including time spent in the queue\. This value is the sum of the amount of time it takes to complete the force merge, snapshot, and shard relocation stages of the migration process\. Relevant statistics: Average  | 
|  HotToWarmMigrationSuccessCount  |  The total number of successful hot to warm migrations\. Relevant statistics: Sum  | 
|  HotToWarmMigrationSuccessLatency  |  The average latency of successful hot to warm migrations, including time spent in the queue\. Relevant statistics: Average  | 

## Cold storage metrics<a name="es-managedomains-cloudwatchmetrics-coldstorage"></a>

Amazon Elasticsearch Service provides the following metrics for [cold storage](cold-storage.md)\.


| Metric | Description | 
| --- | --- | 
| ColdStorageSpaceUtilization  |  The total amount of cold storage space, in MiB, that the cluster is using\.  Relevant statistics: Max  | 
| ColdToWarmMigrationFailureCount |  The total number of failed cold to warm migrations\. Relevant statistics: Sum  | 
| ColdToWarmMigrationLatency |  The amount of time for successful cold to warm migrations to complete\. Relevant statistics: Average  | 
| ColdToWarmMigrationQueueSize |  The number of indices currently waiting to migrate from cold to warm storage\.  Relevant statistics: Maximum  | 
| ColdToWarmMigrationSuccessCount  |  The total number of successful cold to warm migrations\. Relevant statistics: Sum  | 
| WarmToColdMigrationFailureCount  |  The total number of failed warm to cold migrations\. Relevant statistics: Sum  | 
| WarmToColdMigrationLatency |  The amount of time for successful warm to cold migrations to complete\. Relevant statistics: Average  | 
| WarmToColdMigrationQueueSize |  The number of indices currently waiting to migrate from warm to cold storage\.  Relevant statistics: Maximum  | 
| WarmToColdMigrationSuccessCount |  The total number of successful warm to cold migrations\. Relevant statistics: Sum  | 

## Alerting metrics<a name="es-managedomains-cloudwatchmetrics-alerting"></a>

Amazon Elasticsearch Service provides the following metrics for [alerting](alerting.md)\.


| Metric | Description | 
| --- | --- | 
| AlertingDegraded |  A value of 1 means that either the alerting index is red or one or more nodes is not on schedule\. A value of 0 indicates normal behavior\. Relevant statistics: Maximum  | 
| AlertingIndexExists |  A value of 1 means the `.opendistro-alerting-config` index exists\. A value of 0 means it does not\. Until you use the alerting feature for the first time, this value remains 0\. Relevant statistics: Maximum  | 
| AlertingIndexStatus\.green |  The health of the index\. A value of 1 means green\. A value of 0 means that the index either doesn't exist or isn't green\. Relevant statistics: Maximum  | 
| AlertingIndexStatus\.red |  The health of the index\. A value of 1 means red\. A value of 0 means that the index either doesn't exist or isn't red\. Relevant statistics: Maximum  | 
| AlertingIndexStatus\.yellow |  The health of the index\. A value of 1 means yellow\. A value of 0 means that the index either doesn't exist or isn't yellow\. Relevant statistics: Maximum  | 
| AlertingNodesNotOnSchedule |  A value of 1 means some jobs are not running on schedule\. A value of 0 means that all alerting jobs are running on schedule \(or that no alerting jobs exist\)\. Check the Amazon ES console or make a `_nodes/stats` request to see if any nodes show high resource usage\. Relevant statistics: Maximum  | 
| AlertingNodesOnSchedule |  A value of 1 means that all alerting jobs are running on schedule \(or that no alerting jobs exist\)\. A value of 0 means some jobs are not running on schedule\. Relevant statistics: Maximum  | 
| AlertingScheduledJobEnabled |  A value of 1 means that the `opendistro.scheduled_jobs.enabled` cluster setting is true\. A value of 0 means it is false, and scheduled jobs are disabled\. Relevant statistics: Maximum  | 

## Anomaly detection metrics<a name="es-managedomains-cloudwatchmetrics-anomaly-detection"></a>

Amazon Elasticsearch Service provides the following metrics for [anomaly detection](ad.md)\.


| Metric | Description | 
| --- | --- | 
| AnomalyDetectionPluginUnhealthy |  A value of 1 means that the anomaly detection plugin is not functioning properly, either because of a high number of failures or because one of the indices that it uses is red\. A value of 0 indicates the plugin is working as expected\. Relevant statistics: Maximum  | 
| AnomalyDetectionRequestCount |  The number of requests to detect anomalies\. Relevant statistics: Sum  | 
| AnomalyDetectionFailureCount |  The number of failed requests to detect anomalies\.  Relevant statistics: Sum  | 
| AnomalyResultsIndexStatusIndexExists |  A value of 1 means the index that the `.opendistro-anomaly-results` alias points to exists\. Until you use the anomaly detection feature for the first time, this value remains 0\. Relevant statistics: Maximum  | 
| AnomalyResultsIndexStatus\.red |  A value of 1 means the index that the `.opendistro-anomaly-results` alias points to is red\. A value of 0 means it is not\. Until you use the anomaly detection feature for the first time, this value remains 0\. Relevant statistics: Maximum  | 
| AnomalyDetectorsIndexStatusIndexExists |  A value of 1 means that the `.opendistro-anomaly-detectors` index exists\. A value of 0 means it does not\. Until you use the anomaly detection feature for the first time, this value remains 0\. Relevant statistics: Maximum  | 
| AnomalyDetectorsIndexStatus\.red |  A value of 1 means that the `.opendistro-anomaly-detectors` index is red\. A value of 0 means it is not\. Until you use the anomaly detection feature for the first time, this value remains 0\. Relevant statistics: Maximum  | 
| ModelsCheckpointIndexStatusIndexExists |  A value of 1 means that the `.opendistro-anomaly-checkpoints` index exists\. A value of 0 means it does not\. Until you use the anomaly detection feature for the first time, this value remains 0\. Relevant statistics: Maximum  | 
| ModelsCheckpointIndexStatus\.red |  A value of 1 means that the `.opendistro-anomaly-checkpoints` index is red\. A value of 0 means it is not\. Until you use the anomaly detection feature for the first time, this value remains 0\. Relevant statistics: Maximum  | 

## Asynchronous search metrics<a name="es-managedomains-cloudwatchmetrics-asynchronous-search"></a>

Amazon Elasticsearch Service provides the following metrics for [asynchronous search](asynchronous-search.md)\.

**Asynchronous search coordinator node statistics \(per coordinator node\)**


| Metric | Description | 
| --- | --- | 
| AsynchronousSearchSubmissionRate |  The number of asynchronous searches submitted in the last minute\.  | 
| AsynchronousSearchInitializedRate |  The number of asynchronous searches initialized in the last minute\.  | 
| AsynchronousSearchCompletionRate |  The number of asynchronous searches successfully completed in the last minute\.  | 
| AsynchronousSearchFailureRate |  The number of asynchronous searches that completed and failed in the last minute\.  | 
| AsynchronousSearchPersistRate |  The number of asynchronous searches that persisted in the last minute\.  | 
| AsynchronousSearchPersistFailedRate |  The number of asynchronous searches that failed to persist in the last minute\.  | 
| AsynchronousSearchRejected |  The total number of asynchronous searches rejected since the node up time\.  | 
| AsynchronousSearchCancelled |  The total number of asynchronous searches cancelled since the node up time\.  | 
| AsynchronousSearchMaxRunningTime |  The duration of longest running asynchronous search on a node in the last minute\.  | 

**Asynchronous search cluster statistics**


| Metric | Description | 
| --- | --- | 
| AsynchronousSearchStoreHealth |  The health of the store in the persisted index \(RED/non\-RED\) in the last minute\.  | 
| AsynchronousSearchStoreSize |  The size of the system index across all shards in the last minute\.  | 
| AsynchronousSearchStoredResponseCount |  The numbers of stored responses in the system index in the last minute\.  | 

## SQL metrics<a name="es-managedomains-cloudwatchmetrics-sql"></a>

Amazon Elasticsearch Service provides the following metrics for [SQL support](sql-support.md)\.


| Metric | Description | 
| --- | --- | 
| SQLFailedRequestCountByCusErr |  The number of requests to the `_opendistro/_sql` API that failed due to a client issue\. For example, a request might return HTTP status code 400 due to an `IndexNotFoundException`\. Relevant statistics: Sum  | 
| SQLFailedRequestCountBySysErr |  The number of requests to the `_opendistro/_sql` API that failed due to a server problem or feature limitation\. For example, a request might return HTTP status code 503 due to a `VerificationException`\. Relevant statistics: Sum  | 
| SQLRequestCount |  The number of requests to the `_opendistro/_sql` API\. Relevant statistics: Sum  | 
| SQLDefaultCursorRequestCount |   Similar to `SQLRequestCount` but only counts pagination requests\. Relevant statistics: Sum  | 
| SQLUnhealthy |  A value of 1 indicates that, in response to certain requests, the SQL plugin is returning 5*xx* response codes or passing invalid query DSL to Elasticsearch\. Other requests should continue to succeed\. A value of 0 indicates no recent failures\. If you see a sustained value of 1, troubleshoot the requests your clients are making to the plugin\. Relevant statistics: Maximum  | 

## k\-NN metrics<a name="es-managedomains-cloudwatchmetrics-knn"></a>

Amazon Elasticsearch Service includes the following metrics for the k\-nearest neighbor \([k\-NN](knn.md)\) plugin\.


| Metric | Description | 
| --- | --- | 
| KNNCacheCapacityReached |  Per\-node metric for whether cache capacity has been reached\. This metric is only relevant to approximate k\-NN search\. Relevant statistics: Maximum  | 
| KNNCircuitBreakerTriggered |  Per\-cluster metric for whether the circuit breaker is triggered\. If any nodes return a value of 1 for `KNNCacheCapacityReached`, this value will also return 1\. This metric is only relevant to approximate k\-NN search\. Relevant statistics: Maximum  | 
| KNNEvictionCount |  Per\-node metric for the number of graphs that have been evicted from the cache due to memory constraints or idle time\. Explicit evictions that occur because of index deletion are not counted\. This metric is only relevant to approximate k\-NN search\. Relevant statistics: Sum  | 
| KNNGraphIndexErrors |  Per\-node metric for the number of requests to add the `knn_vector` field of a document to a graph that produced an error\. Relevant statistics: Sum  | 
| KNNGraphIndexRequests |  Per\-node metric for the number of requests to add the `knn_vector` field of a document to a graph\. Relevant statistics: Sum  | 
| KNNGraphMemoryUsage |  Per\-node metric for the current cache size \(total size of all graphs in memory\) in kilobytes\. This metric is only relevant to approximate k\-NN search\. Relevant statistics: Average  | 
| KNNGraphQueryErrors |  Per\-node metric for the number of graph queries that produced an error\. Relevant statistics: Sum  | 
| KNNGraphQueryRequests |  Per\-node metric for the number of graph queries\. Relevant statistics: Sum  | 
| KNNHitCount |  Per\-node metric for the number of cache hits\. A cache hit occurs when a user queries a graph that is already loaded into memory\. This metric is only relevant to approximate k\-NN search\. Relevant statistics: Sum  | 
| KNNLoadExceptionCount |  Per\-node metric for the number of times an exception occurred while trying to load a graph into the cache\. This metric is only relevant to approximate k\-NN search\. Relevant statistics: Sum  | 
| KNNLoadSuccessCount |  Per\-node metric for the number of times the plugin successfully loaded a graph into the cache\. This metric is only relevant to approximate k\-NN search\. Relevant statistics: Sum  | 
| KNNMissCount |  Per\-node metric for the number of cache misses\. A cache miss occurs when a user queries a graph that is not yet loaded into memory\. This metric is only relevant to approximate k\-NN search\. Relevant statistics: Sum  | 
| KNNQueryRequests |  Per\-node metric for the number of query requests the k\-NN plugin received\. Relevant statistics: Sum  | 
| KNNScriptCompilationErrors |  Per\-node metric for the number of errors during script compilation\. This statistic is only relevant to k\-NN score script search\. Relevant statistics: Sum  | 
| KNNScriptCompilations |  Per\-node metric for the number of times the k\-NN script has been compiled\. This value should usually be 1 or 0, but if the cache containing the compiled scripts is filled, the k\-NN script might be recompiled\. This statistic is only relevant to k\-NN score script search\. Relevant statistics: Sum  | 
| KNNScriptQueryErrors |  Per\-node metric for the number of errors during script queries\. This statistic is only relevant to k\-NN score script search\. Relevant statistics: Sum  | 
| KNNScriptQueryRequests |  Per\-node metric for the total number of script queries\. This statistic is only relevant to k\-NN score script search\. Relevant statistics: Sum  | 
| KNNTotalLoadTime |  The time in nanoseconds that k\-NN has taken to load graphs into the cache\. This metric is only relevant to approximate k\-NN search\. Relevant statistics: Sum  | 

## Cross\-cluster search metrics<a name="es-managedomains-cloudwatchmetrics-cross-cluster-search"></a>

Amazon Elasticsearch Service provides the following metrics for [cross\-cluster search](cross-cluster-search.md)\.

**Source domain metrics**


| Metric | Dimension | Description | 
| --- | --- | --- | 
| CrossClusterOutboundConnections |  `ConnectionId`  |  Number of connected nodes\. If your response includes one or more skipped domains, use this metric to trace any unhealthy connections\. If this number drops to 0, then the connection is unhealthy\.  | 
| CrossClusterOutboundRequests |  `ConnectionId`  |  Number of search requests sent to the destination domain\. Use to check if the load of cross\-cluster search requests are overwhelming your domain, correlate any spike in this metric with any JVM/CPU spike\.  | 

**Destination domain metric**


| Metric | Dimension | Description | 
| --- | --- | --- | 
| CrossClusterInboundRequests |  `ConnectionId`  |  Number of incoming connection requests received from the source domain\.  | 

Add a CloudWatch alarm in the event that you lose a connection unexpectedly\. For steps to create an alarm, see [Create a CloudWatch Alarm Based on a Static Threshold](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ConsoleAlarms.html)\.

## Learning to Rank metrics<a name="es-managedomains-cloudwatchmetrics-learning-to-rank"></a>

Amazon Elasticsearch Service provides the following metrics for [Learning to Rank](learning-to-rank.md)\.


| Metric | Description | 
| --- | --- | 
| LTRRequestTotalCount |  Total count of ranking requests\.  | 
| LTRRequestErrorCount |  Total count of unsuccessful requests\.  | 
| LTRStoreIndexIsRed |  Tracks if one of the indices needed to run the plugin is red\.  | 
| LTRMemoryUsage |  Total memory used by the plugin\.  | 