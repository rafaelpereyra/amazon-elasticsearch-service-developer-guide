# Supported instance types in Amazon Elasticsearch Service<a name="aes-supported-instance-types"></a>

Amazon Elasticsearch Service \(Amazon ES\) supports the following instance types\. Not all Regions support all instance types\. For availability details, see [Amazon Elasticsearch Service Pricing](https://aws.amazon.com/elasticsearch-service/pricing/)\.

For information about which instance type is appropriate for your use case, see [Sizing Amazon Elasticsearch Service domains](sizing-domains.md), [EBS volume size limits](aes-limits.md#ebsresource), and [Network limits](aes-limits.md#network-limits)\.


****  

| Instance Type | Restrictions | 
| --- | --- | 
|  C4  |    | 
|  C5  |  The C5 instance types require Elasticsearch version 5\.1 or later\.  | 
| C6G  |  The C6G instance types require Elasticsearch version 7\.9 or later\.  | 
|  I2  |    | 
|  I3  | The I3 instance types require Elasticsearch version 5\.1 or later and do not support EBS storage volumes\. | 
|  M3  |  The M3 instance types do not support encryption of data at rest, fine\-grained access control, or cross\-cluster search\. The M3 instance types have additional restrictions by Elasticsearch version\. To learn more, see [Invalid M3 instance type](aes-handling-errors.md#aes-m3-instance-types)\.  | 
|  M4  |    | 
|  M5  |  The M5 instance types require Elasticsearch version 5\.1 or later\.  | 
| M6G |  The M6G instance types require Elasticsearch version 7\.9 or later\.  | 
|  R3  |  The R3 instance types do not support encryption of data at rest or fine\-grained access control\.  | 
|  R4  |    | 
|  R5  |  The R5 instance types require Elasticsearch version 5\.1 or later\.  | 
| R6G |  The R6G instance types require Elasticsearch version 7\.9 or later\.  | 
| R6GD |  The R6GD instance types require Elasticsearch version 7\.9 or later and do not support EBS storage volumes\.  | 
|  T2  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-instance-types.html)  | 
|  T3  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-instance-types.html)  | 

**Tip**  
We often recommend different instance types for [dedicated master nodes](es-managedomains-dedicatedmasternodes.md) and data nodes\.