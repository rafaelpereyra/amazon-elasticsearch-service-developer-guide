# Fine\-grained access control in Amazon Elasticsearch Service<a name="fgac"></a>

Fine\-grained access control offers additional ways of controlling access to your data on Amazon Elasticsearch Service \(Amazon ES\)\. For example, depending on who makes the request, you might want a search to return results from only one index\. You might want to hide certain fields in your documents or exclude certain documents altogether\. Fine\-grained access control offers the following benefits:
+ Role\-based access control
+ Security at the index, document, and field level
+ Kibana multi\-tenancy
+ HTTP basic authentication for Elasticsearch and Kibana

**Topics**
+ [The bigger picture: fine\-grained access control and Amazon ES security](#fgac-access-policies)
+ [Key concepts](#fgac-concepts)
+ [Enabling fine\-grained access control](#fgac-enabling)
+ [Accessing Kibana as the master user](#fgac-kibana)
+ [Managing permissions](#fgac-access-control)
+ [Recommended configurations](#fgac-recommendations)
+ [Tutorial: IAM master user and Amazon Cognito](#fgac-walkthrough-iam)
+ [Tutorial: Internal user database and HTTP basic authentication](#fgac-walkthrough-basic)
+ [Limitations](#fgac-limitations)
+ [Modifying the master user](#fgac-forget)
+ [Additional master users](#fgac-more-masters)
+ [Manual snapshots](#fgac-snapshots)
+ [Integrations](#fgac-integrations)
+ [REST API differences](#fgac-rest-api)

## The bigger picture: fine\-grained access control and Amazon ES security<a name="fgac-access-policies"></a>

Amazon Elasticsearch Service security has three main layers:

**Network**  
The first security layer is the network, which determines whether requests reach an Amazon ES domain\. If you choose **Public access** when you create a domain, requests from any internet\-connected client can reach the domain endpoint\. If you choose **VPC access**, clients must connect to the VPC \(and the associated security groups must permit it\) for a request to reach the endpoint\. For more information, see [Launching your Amazon Elasticsearch Service domains using a VPC](es-vpc.md)\.

**Domain access policy**  
The second security layer is the domain access policy\. After a request reaches a domain endpoint, the [resource\-based access policy](es-ac.md#es-ac-types-resource) allows or denies the request access to a given URI\. The access policy accepts or rejects requests at the "edge" of the domain, before they reach Elasticsearch itself\.

**Fine\-grained access control**  
The third and final security layer is fine\-grained access control\. After a resource\-based access policy allows a request to reach a domain endpoint, fine\-grained access control evaluates the user credentials and either authenticates the user or denies the request\. If fine\-grained access control authenticates the user, it fetches all roles mapped to that user and uses the complete set of permissions to determine how to handle the request\.

**Note**  
If a resource\-based access policy contains IAM users or roles, clients must send signed requests using AWS Signature Version 4\. As such, access policies can conflict with fine\-grained access control, especially if you use the internal user database and HTTP basic authentication\. You can't sign a request with a user name and password *and* IAM credentials\. In general, if you enable fine\-grained access control, we recommend using a domain access policy that doesn't require signed requests\.

This first diagram illustrates a common configuration: a VPC access domain with fine\-grained access control enabled, an IAM\-based access policy, and an IAM master user\.

![\[Fine-grained access control authorization flow with a VPC domain\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/fgac-vpc-iam.png)

This second diagram illustrates another common configuration: a public access domain with fine\-grained access control enabled, an access policy that doesn't use IAM principals, and a master user in the internal user database\.

![\[Fine-grained access control authorization flow with a public access domain\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/fgac-public-basic.png)

### Example<a name="fgac-example"></a>

Consider a `GET` request to `movies/_search?q=thor`\. Does the user have permissions to search the `movies` index? If so, does the user have permissions to see all documents within it? Should the response omit or anonymize any fields? For the master user, the response might look like this:

```
{
  "hits": {
    "total": 7,
    "max_score": 8.772789,
    "hits": [{
        "_index": "movies",
        "_type": "_doc",
        "_id": "tt0800369",
        "_score": 8.772789,
        "_source": {
          "directors": [
            "Kenneth Branagh",
            "Joss Whedon"
          ],
          "release_date": "2011-04-21T00:00:00Z",
          "genres": [
            "Action",
            "Adventure",
            "Fantasy"
          ],
          "plot": "The powerful but arrogant god Thor is cast out of Asgard to live amongst humans in Midgard (Earth), where he soon becomes one of their finest defenders.",
          "title": "Thor",
          "actors": [
            "Chris Hemsworth",
            "Anthony Hopkins",
            "Natalie Portman"
          ],
          "year": 2011
        }
      },
      ...
    ]
  }
}
```

If a user with more limited permissions issues the exact same request, the response might look like this:

```
{
  "hits": {
    "total": 2,
    "max_score": 8.772789,
    "hits": [{
        "_index": "movies",
        "_type": "_doc",
        "_id": "tt0800369",
        "_score": 8.772789,
        "_source": {
          "year": 2011,
          "release_date": "3812a72c6dd23eef3c750c2d99e205cbd260389461e19d610406847397ecb357",
          "plot": "The powerful but arrogant god Thor is cast out of Asgard to live amongst humans in Midgard (Earth), where he soon becomes one of their finest defenders.",
          "title": "Thor"
        }
      },
      ...
    ]
  }
}
```

The response has fewer hits and fewer fields for each hit\. Also, the `release_date` field is anonymized\. If a user with no permissions makes the same request, the cluster returns an error:

```
{
  "error": {
    "root_cause": [{
      "type": "security_exception",
      "reason": "no permissions for [indices:data/read/search] and User [name=limited-user, roles=[], requestedTenant=null]"
    }],
    "type": "security_exception",
    "reason": "no permissions for [indices:data/read/search] and User [name=limited-user, roles=[], requestedTenant=null]"
  },
  "status": 403
}
```

If a user provides invalid credentials, the cluster returns an `Unauthorized` exception\.

## Key concepts<a name="fgac-concepts"></a>

*Roles* are the core way of using fine\-grained access control\. In this case, roles are distinct from IAM roles\. Roles contain any combination of permissions: cluster\-wide, index\-specific, document level, and field level\.

After configuring a role, you *map* it to one or more users\. For example, you might map three roles to a single user: one role that provides access to Kibana, one that provides read\-only access to `index1`, and one that provides write access to `index2`\. Or you could include all of those permissions in a single role\.

*Users* are people or applications that make requests to the Elasticsearch cluster\. Users have credentials—either IAM access keys or a user name and password—that they specify when they make requests\. With fine\-grained access control on Amazon Elasticsearch Service, you choose one or the other for your *master user* when you configure your domain\. The master user has full permissions to the cluster and manages roles and role mappings\.
+ If you choose IAM for your master user, all requests to the cluster must be signed using AWS Signature Version 4\. For sample code, see [Signing HTTP requests to Amazon Elasticsearch Service](es-request-signing.md)\.

  We recommend IAM if you want to use the same users on multiple clusters, if you want to use Amazon Cognito to access Kibana, or if you have Elasticsearch clients that support Signature Version 4 signing\.
+ If you choose the internal user database, you can use HTTP basic authentication \(as well as IAM credentials\) to make requests to the cluster\. Most clients support basic authentication, including [curl](https://curl.haxx.se/)\. The internal user database is stored in an Elasticsearch index, so you can't share it with other clusters\.

  We recommend the internal user database if you don't need to reuse users across multiple clusters, if you want to use HTTP basic authentication to access Kibana \(rather than Amazon Cognito\), or if you have clients that only support basic authentication\. The internal user database is the simplest way to get started with Amazon ES\.

## Enabling fine\-grained access control<a name="fgac-enabling"></a>

Enable fine\-grained access control using the console, AWS CLI, or configuration API\. The console offers the simplest experience\. For steps, see [Creating and managing Amazon Elasticsearch Service domains](es-createupdatedomains.md)\. Here are the requirements for enabling fine\-grained access control:
+ Elasticsearch 6\.7 or later
+ [Encryption of data at rest](encryption-at-rest.md) and [node\-to\-node encryption](ntn.md) enabled
+ **Require HTTPS for all traffic to the domain** enabled

You can't enable fine\-grained access control on existing domains, only new ones\. After you enable fine\-grained access control, you can't disable it\.

## Accessing Kibana as the master user<a name="fgac-kibana"></a>

Fine\-grained access control has a Kibana plugin that simplifies management tasks\. You can use Kibana to manage users, roles, mappings, action groups, and tenants\. The Kibana sign\-in page and underlying authentication method differs, however, depending on how you manage users and configured your domain\.
+ If you want to use IAM for user management, use [Configuring Amazon Cognito authentication for Kibana](es-cognito-auth.md) to access Kibana\. Otherwise, Kibana shows a nonfunctional sign\-in page\. See [Limitations](#fgac-limitations)\.

  With Amazon Cognito authentication, one of the assumed roles from the identity pool must match the IAM role that you specified for the master user\. For more information about this configuration, see [\(Optional\) Configuring granular access](es-cognito-auth.md#es-cognito-auth-granular) and [Tutorial: IAM master user and Amazon Cognito](#fgac-walkthrough-iam)\.  
![\[Cognito sign-in page\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/cognito-auth.png)
+ If you choose to use the internal user database, you can sign in to Kibana with your master user name and password\. You must access Kibana over HTTPS\. Amazon Cognito and SAML authentication for Kibana both replace this login screen\.

  For more information about this configuration, see [Tutorial: Internal user database and HTTP basic authentication](#fgac-walkthrough-basic)\.  
![\[Basic authentication sign-in page\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/basic-auth-kibana.png)
+ If you choose to use SAML authentication, you can sign in using credentials from an external identity provider\. For more information, see [SAML authentication for Kibana](saml.md)\.

## Managing permissions<a name="fgac-access-control"></a>

As noted in [Key concepts](#fgac-concepts), you manage fine\-grained access control permissions using roles, users, and mappings\. This section describes how to create and apply those resources\. We recommend that you [sign in to Kibana as the master user](#fgac-kibana) to perform these operations\.

![\[Security home page in Kibana\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/kibana-fgac-home.png)

### Creating roles<a name="fgac-roles"></a>

You can create new roles for fine\-grained access control using Kibana or the `_opendistro/_security` operation in the REST API\. For more information, see the [Open Distro for Elasticsearch documentation](https://opendistro.github.io/for-elasticsearch-docs/docs/security/access-control/users-roles/#create-roles)\.

Fine\-grained access control also includes a number of [predefined roles](https://opendistro.github.io/for-elasticsearch-docs/docs/security/access-control/users-roles/#predefined-roles)\. Clients such as Kibana and Logstash make a wide variety of requests to Elasticsearch, which can make it hard to manually create roles with the minimum set of permissions\. For example, the `kibana_user` role includes the permissions that a user needs to work with index patterns, visualizations, dashboards, and tenants\. We recommend [mapping it](#fgac-mapping) to any user or backend role that accesses Kibana, along with additional roles that allow access to other indices\.

#### Cluster\-level security<a name="fgac-cluster-level"></a>

Cluster\-level permissions include the ability to make broad requests such as `_mget`, `_msearch`, and `_bulk`, monitor health, take snapshots, and more\. Manage these permissions using the **Cluster Permissions** section when creating a role\. For a list of cluster\-level action groups, see the [Open Distro for Elasticsearch documentation](https://opendistro.github.io/for-elasticsearch-docs/docs/security/access-control/default-action-groups/#cluster-level)\.

#### Index\-level security<a name="fgac-index-level"></a>

Index\-level permissions include the ability to create new indices, search indices, read and write documents, delete documents, manage aliases, and more\. Manage these permissions using the **Index Permissions** section when creating a role\. For a list of index\-level action groups, see the [Open Distro for Elasticsearch documentation](https://opendistro.github.io/for-elasticsearch-docs/docs/security/access-control/default-action-groups/#index-level)\.

#### Document\-level security<a name="fgac-document-level"></a>

Document\-level security lets you restrict which documents in an index a user can see\. When creating a role, specify an index pattern and an Elasticsearch query\. Any users that you map to that role can see only the documents that match the query\. Document\-level security affects [the number of hits that you receive when you search](#fgac-example)\.

For more information, see the [Open Distro for Elasticsearch documentation](https://opendistro.github.io/for-elasticsearch-docs/docs/security/access-control/document-level-security/)\.

#### Field\-level security<a name="fgac-field-level"></a>

Field\-level security lets you control which document fields a user can see\. When creating a role, add a list of fields to either include or exclude\. If you include fields, any users you map to that role can see only those fields\. If you exclude fields, they can see all fields *except* the excluded ones\. Field\-level security affects [the number of fields included in hits when you search](#fgac-example)\.

For more information, see the [Open Distro for Elasticsearch documentation](https://opendistro.github.io/for-elasticsearch-docs/docs/security/access-control/field-level-security/)\.

#### Field masking<a name="fgac-field-masking"></a>

Field masking is an alternative to field\-level security that lets you anonymize the data in a field rather than remove it altogether\. When creating a role, add a list of fields to mask\. Field masking affects [whether you can see the contents of a field when you search](#fgac-example)\.

**Tip**  
If you apply the standard masking to a field, Amazon ES uses a secure, random hash that can cause inaccurate aggregation results\. To perform aggregations on masked fields, use pattern\-based masking instead\.

### Creating users<a name="fgac-users"></a>

If you enabled the internal user database, you can create users using Kibana or the `_opendistro/_security` operation in the REST API\. For more information, see the [Open Distro for Elasticsearch documentation](https://opendistro.github.io/for-elasticsearch-docs/docs/security/access-control/users-roles/#create-users)\.

If you chose IAM for your master user, ignore this portion of Kibana\. Create IAM users and IAM roles instead\. For more information, see the [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)\.

### Mapping roles to users<a name="fgac-mapping"></a>

Role mapping is the most critical aspect of fine\-grained access control\. Fine\-grained access control has some predefined roles to help you get started, but unless you map roles to users, every request to the cluster ends in a permissions error\.

*Backend roles*, also called *external identities*, offer another way of mapping roles to users\. Rather than mapping the same role to dozens of different users, you can map the role to a single backend role, and then make sure that all users have that backend role\. Backend roles can be IAM roles or arbitrary strings\.
+ Specify users, IAM user ARNs, and Amazon Cognito user strings in the **Users** section\. Cognito user strings take the form of `Cognito/user-pool-id/username`\.
+ Specify backend roles and IAM role ARNs in the **External identities** section\.

![\[Role mapping screen\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/role-mapping-edit.png)

You can map roles to users using Kibana or the `_opendistro/_security` operation in the REST API\. For more information, see the [Open Distro for Elasticsearch documentation](https://opendistro.github.io/for-elasticsearch-docs/docs/security/access-control/users-roles/#map-users-to-roles)\.

### Creating action groups<a name="fgac-ag"></a>

Action groups are sets of permissions that you can reuse across different resources\. You can create new action groups using Kibana or the `_opendistro/_security` operation in the REST API, although the default action groups suffice for most use cases\. For more information about the default action groups, see the [Open Distro for Elasticsearch documentation](https://opendistro.github.io/for-elasticsearch-docs/docs/security/access-control/default-action-groups/)\.

### Kibana multi\-tenancy<a name="fgac-multitenancy"></a>

Tenants are spaces for saving index patterns, visualizations, dashboards, and other Kibana objects\. Kibana multi\-tenancy lets you safely share your work with other Kibana users \(or keep it private\)\. You can control which roles have access to a tenant and whether those roles have read or write access\. To learn more, see the [Open Distro for Elasticsearch documentation](https://opendistro.github.io/for-elasticsearch-docs/docs/security/access-control/multi-tenancy/#add-tenants)\.

**To view your current tenant or change tenants**

1. Navigate to Kibana and sign in\.

1. In the upper\-right, choose your user icon\.

1. Choose **Switch tenants**\.

1. Verify your tenant before creating visualizations or dashboards\. If you want to share your work with all other Kibana users, choose **Global**\. To share your work with a subset of Kibana users, choose a different shared tenant\. Otherwise, choose **Private**\.

## Recommended configurations<a name="fgac-recommendations"></a>

Due to how fine\-grained access control [interacts with other security features](#fgac-access-policies), we recommend several fine\-grained access control configurations that work well for most use cases\.


| Description | Master User | Domain Access Policy | 
| --- | --- | --- | 
|  Use IAM credentials for calls to the Elasticsearch APIs, and use [SAML authentication](saml.md) to access Kibana\. Manage fine\-grained access control roles using Kibana or the REST API\.  | IAM user or role |  <pre>{<br />  "Version": "2012-10-17",<br />  "Statement": [<br />    {<br />      "Effect": "Allow",<br />      "Principal": {<br />        "AWS": "*"<br />      },<br />      "Action": "es:ESHttp*",<br />      "Resource": "domain-arn/*"<br />    }<br />  ]<br />}</pre>  | 
|  Use IAM credentials or basic authentication for calls to the Elasticsearch APIs\. Manage fine\-grained access control roles using Kibana or the REST API\. This configuration offers a lot of flexiblity, especially if you have Elasticsearch clients that only support basic authentication\. If you have an existing identity provider, use [SAML authentication](saml.md) to access Kibana\. Otherwise, manage Kibana users in the internal user database\.  | User name and password |  <pre>{<br />  "Version": "2012-10-17",<br />  "Statement": [<br />    {<br />      "Effect": "Allow",<br />      "Principal": {<br />        "AWS": "*"<br />      },<br />      "Action": "es:ESHttp*",<br />      "Resource": "domain-arn/*"<br />    }<br />  ]<br />}</pre>  | 
|  Use IAM credentials for calls to the Elasticsearch APIs, and use Amazon Cognito to access Kibana\. Manage fine\-grained access control roles using Kibana or the REST API\.  | IAM user or role |  <pre>{<br />  "Version": "2012-10-17",<br />  "Statement": [<br />    {<br />      "Effect": "Allow",<br />      "Principal": {<br />        "AWS": "*"<br />      },<br />      "Action": "es:ESHttp*",<br />      "Resource": "domain-arn/*"<br />    }<br />  ]<br />}</pre>  | 
|  Use IAM credentials for calls to the Elasticsearch APIs, and block most access to Kibana\. Manage fine\-grained access control roles using the REST API\.  | IAM user or role |  <pre>{<br />  "Version": "2012-10-17",<br />  "Statement": [<br />    {<br />      "Effect": "Allow",<br />      "Principal": {<br />        "AWS": "*"<br />      },<br />      "Action": "es:ESHttp*",<br />      "Resource": "domain-arn/*"<br />    },<br />    {<br />      "Effect": "Deny",<br />      "Principal": {<br />        "AWS": "*"<br />      },<br />      "Action": "es:ESHttp*",<br />      "Resource": "domain-arn/_plugin/kibana*"<br />    }<br />  ]<br />}</pre>  | 

## Tutorial: IAM master user and Amazon Cognito<a name="fgac-walkthrough-iam"></a>

This tutorial covers a popular use case: an IAM master user with Amazon Cognito authentication for Kibana\. Although these steps use the Amazon Cognito user pool for authentication, this same basic process works for any Cognito authentication provider that lets you assign different IAM roles to different users\.

**Note**  
This tutorial assumes you have two existing IAM roles, one for the master user and one for more limited users\. If you don't have two roles, [create them](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html)\.

**To get started with fine\-grained access control**

1. [Create a domain](es-createupdatedomains.md) with the following settings:
   + Elasticsearch 7\.8
   + Public access
   + Fine\-grained access control enabled with an IAM role as the master user \(`IAMMasterUserRole` for the rest of this tutorial\)
   + [Amazon Cognito authentication for Kibana](es-cognito-auth.md) enabled
   + The following access policy:

     ```
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Principal": {
             "AWS": [
               "*"
             ]
           },
           "Action": [
             "es:ESHttp*"
           ],
           "Resource": "arn:aws:es:region:account:domain/domain-name/*"
         }
       ]
     }
     ```
   + HTTPS required for all traffic to the domain
   + Node\-to\-node encryption
   + Encryption of data at rest

1. Navigate to the IAM console, and then choose **Roles**\.

1. Choose `IAMMasterUserRole`, and then choose the **Trust relationships** tab\.

1. Choose **Edit trust relationship**, and ensure that the Amazon Cognito identity pool can assume the role\. You should see the following statement:

   ```
   {
     "Version": "2012-10-17",
     "Statement": [{
       "Effect": "Allow",
       "Principal": {
         "Federated": "cognito-identity.amazonaws.com"
       },
       "Action": "sts:AssumeRoleWithWebIdentity",
       "Condition": {
         "StringEquals": {
           "cognito-identity.amazonaws.com:aud": "identity-pool-id"
         },
         "ForAnyValue:StringLike": {
           "cognito-identity.amazonaws.com:amr": "authenticated"
         }
       }
     }]
   }
   ```

1. Choose **Update Trust Policy**\.

1. Add the same trust policy to a second IAM role \(`IAMLimitedUserRole` for the rest of this tutorial\)\.

1. Navigate to the Amazon Cognito console, and then choose **Manage User Pools**\.

1. Choose your user pool, and then choose **Users and groups**\.

1. Choose **Create user**, specify a user name of `master-user` and a password, and then choose **Create user**\.

1. Create another user named `limited-user`\.

1. Choose the **Groups** tab, and then choose **Create group**\.

1. Name the group `master-user-group`, choose `IAMMasterUserRole` in the **IAM role** dropdown list, and then choose **Create group**\.

1. Create another group named `limited-user-group` that uses `IAMLimitedUserRole`\.

1. Choose `master-user-group`, choose **Add users**, and then add `master-user`\.

1. Choose `limited-user-group`, choose **Add users**, and then add `limited-user`\.

1. Choose **App client settings** and note the app client ID for your domain\.

1. Choose **Federated Identities**, choose your identity pool, and then choose **Edit identity pool**\.

1. Expand **Authentication providers**, find your user pool ID and the app client ID for your domain, and then change **Use default role** to **Choose role from token**\.

1. For **Role resolution**, choose **DENY**\. With this setting, users must be in a group to receive an IAM role after authenticating\.

1. Choose **Save Changes**\.

1. Navigate to Kibana\.

1. Sign in with `master-user`\.

1. Choose **Try our sample data**\.

1. Add the sample flight data\.

1. Choose **Security**, **Roles**, **Create role**\.

1. Name the role `new-role`\.

1. For index permissions, specify `kibana_sample_data_fli*` for the index pattern\.

1. For the action group, choose **read**\.

1. For **Document Level Security Query**, specify the following query:

   ```
   {
     "match": {
       "FlightDelay": true
     }
   }
   ```

1. For field\-level security, choose **Exclude fields** and specify `FlightNum`\.

1. For **Anonymize fields**, specify `Dest`\.

1. Choose **Create**\.

1. Choose **Mapped users**, **Manage mapping**\. Then add the ARN for `IAMLimitedUserRole` as an external identity and choose **Map**\.

1. Return to the list of roles and choose **kibana\_user**\. Choose **Mapped users**, **Manage mapping**\. Then add the ARN for `IAMLimitedUserRole` as an external identity and choose **Map**\.

1. In a new, private browser window, navigate to Kibana, sign in using `limited-user`, and then choose **Explore on my own**\.

1. Choose **Dev Tools**, and then run the default search:

   ```
   GET _search
   {
     "query": {
       "match_all": {}
     }
   }
   ```

   Note the permissions error\. `limited-user` doesn't have permissions to run cluster\-wide searches\.

1. Run another search:

   ```
   GET kibana_sample_data_flights/_search
   {
     "query": {
       "match_all": {}
     }
   }
   ```

   Note that all matching documents have a `FlightDelay` field of `true`, an anonymized `Dest` field, and no `FlightNum` field\.

1. In your original browser window, signed in as `master-user`, choose **Dev Tools**, and then perform the same searches\. Note the difference in permissions, number of hits, matching documents, and included fields\.

## Tutorial: Internal user database and HTTP basic authentication<a name="fgac-walkthrough-basic"></a>

This tutorial covers another popular use case: a master user in the internal user database and HTTP basic authentication for Kibana\.

**To get started with fine\-grained access control**

1. [Create a domain](es-createupdatedomains.md) with the following settings:
   + Elasticsearch 7\.9
   + Public access
   + Fine\-grained access control with a master user in the internal user database \(`TheMasterUser` for the rest of this tutorial\)
   + Amazon Cognito authentication for Kibana *disabled*
   + The following access policy:

     ```
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Principal": {
             "AWS": [
               "*"
             ]
           },
           "Action": [
             "es:ESHttp*"
           ],
           "Resource": "arn:aws:es:region:account:domain/domain-name/*"
         }
       ]
     }
     ```
   + HTTPS required for all traffic to the domain
   + Node\-to\-node encryption
   + Encryption of data at rest

1. Navigate to Kibana\.

1. Sign in using `TheMasterUser`\.

1. Choose **Try our sample data**\.

1. Add the sample flight data\.

1. Choose **Security**, **Internal users**, **Create internal user**\.

1. Name the user `new-user` and specify a password\. Then choose **Create**\.

1. Choose **Roles**, **Create role**\.

1. Name the role `new-role`\.

1. For index permissions, specify `kibana_sample_data_fli*` for the index pattern\.

1. For the action group, choose **read**\.

1. For **Document Level Security Query**, specify the following query:

   ```
   {
     "match": {
       "FlightDelay": true
     }
   }
   ```

1. For field\-level security, choose **Exclude fields** and specify `FlightNum`\.

1. For **Anonymize fields**, specify `Dest`\.

1. Choose **Create**\.

1. Choose **Mapped users**, **Manage mapping**\. Then add `new-user` to **Users** and choose **Map**\.

1. Return to the list of roles and choose **kibana\_user**\. Choose **Mapped users**, **Manage mapping**\. Then add `new-user` to **Users** and choose **Map**\.

1. In a new, private browser window, navigate to Kibana, sign in using `new-user`, and then choose **Explore on my own**\.

1. Choose **Dev Tools** and run the default search:

   ```
   GET _search
   {
     "query": {
       "match_all": {}
     }
   }
   ```

   Note the permissions error\. `new-user` doesn't have permissions to run cluster\-wide searches\.

1. Run another search:

   ```
   GET kibana_sample_data_flights/_search
   {
     "query": {
       "match_all": {}
     }
   }
   ```

   Note that all matching documents have a `FlightDelay` field of `true`, an anonymized `Dest` field, and no `FlightNum` field\.

1. In your original browser window, signed in as `TheMasterUser`, choose **Dev Tools** and perform the same searches\. Note the difference in permissions, number of hits, matching documents, and included fields\.

## Limitations<a name="fgac-limitations"></a>

Fine\-grained access control has several important limitations:
+ The `hosts` aspect of role mappings, which maps roles to hostnames or IP addresses, doesn't work if the domain is within a VPC\. You can still map roles to users and backend roles\.
+ Users in the internal user database can't change their own passwords\. Master users \(or users with equivalent permissions\) must change their passwords for them\.
+ If you choose IAM for the master user and don't enable Amazon Cognito or SAML authentication, Kibana displays a nonfunctional sign\-in page\.
+ If you choose IAM for the master user, you can still create users in the internal user database\. Because HTTP basic authentication is not enabled under this configuration, however, any requests signed with those user credentials are rejected\.
+ If you use [SQL](sql-support.md) to query an index that you don't have access to, you receive a "no permissions" error\. If the index doesn't exist, you receive a "no such index" error\. This difference in error messages means that you can confirm the existence of an index if you happen to guess its name\.

  To minimize the issue, [don't include sensitive information in index names](es-indexing.md#es-indexing-naming)\. To deny all access to SQL, add the following element to your domain access policy:

  ```
  {
    "Effect": "Deny",
    "Principal": {
      "AWS": [
        "*"
      ]
    },
    "Action": [
      "es:*"
    ],
    "Resource": "arn:aws:es:us-east-1:123456789012:domain/my-domain/_opendistro/_sql"
  }
  ```

## Modifying the master user<a name="fgac-forget"></a>

If you forget the details of the master user, you can reconfigure it using the console, AWS CLI, or configuration API\.

**To modify the master user \(console\)**

1. Go to [https://aws\.amazon\.com](https://aws.amazon.com), and then choose **Sign In to the Console**\.

1. Under **Analytics**, choose **Elasticsearch Service**\.

1. Choose your domain\.

1. Choose **Actions**, **Modify authentication**\.

1. Choose either **Set IAM role as master user** or **Create new master user**\.
   + If you previously used an IAM master user, fine\-grained access control re\-maps the `all_access` role to the new IAM ARN that you specify\.
   + If you previously used the internal user database, fine\-grained access control creates a new master user\. You can use the new master user to delete the old one\.
   + Switching from the internal user database to an IAM master user does *not* delete any users from the internal user database\. Instead, it just disables HTTP basic authentication\. Manually delete users from the internal user database, or keep them in case you ever need to reenable HTTP basic authentication\.

1. Choose **Submit**\.

## Additional master users<a name="fgac-more-masters"></a>

You designate a master user when you create a domain, but if you want, you can use this master user to create additional master users\. You have two options: Kibana or the REST API\.
+ In Kibana, choose **Security**, **Role Mappings**, and then map the new master user to the `all_access` and `security_manager` roles\.  
![\[Role mapping page\]](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/images/new-master-users.png)
+ To use the REST API, send the following requests:

  ```
  PUT _opendistro/_security/api/rolesmapping/all_access
  {
    "backend_roles": [
      "arn:aws:iam::123456789012:role/fourth-master-user"
    ],
    "hosts": [],
    "users": [
      "master-user",
      "second-master-user",
      "arn:aws:iam::123456789012:user/third-master-user"
    ]
  }
  ```

  ```
  PUT _opendistro/_security/api/rolesmapping/security_manager
  {
    "backend_roles": [
      "arn:aws:iam::123456789012:role/fourth-master-user"
    ],
    "hosts": [],
    "users": [
      "master-user",
      "second-master-user",
      "arn:aws:iam::123456789012:user/third-master-user"
    ]
  }
  ```

  These requests *replace* the current role mappings, so perform `GET` requests first so that you can include all current roles in the `PUT` requests\. The REST API is especially useful if you can't access Kibana and want to map an IAM role from Amazon Cognito to the `all_access` role\.

## Manual snapshots<a name="fgac-snapshots"></a>

Fine\-grained access control introduces some additional complications with taking manual snapshots\. To register a snapshot repository—even if you use HTTP basic authentication for all other purposes—you must map the `manage_snapshots` role to an IAM role that has `iam:PassRole` permissions to assume `TheSnapshotRole`, as defined in [Prerequisites](es-managedomains-snapshots.md#es-managedomains-snapshot-prerequisites)\.

Then use that IAM role to send a signed request to the domain, as outlined in [Registering a manual snapshot repository](es-managedomains-snapshots.md#es-managedomains-snapshot-registerdirectory)\.

## Integrations<a name="fgac-integrations"></a>

If you use [other AWS services](es-aws-integrations.md) with Amazon ES, you must provide the IAM roles for those services with appropriate permissions\. For example, Kinesis Data Firehose delivery streams often use an IAM role called `firehose_delivery_role`\. In Kibana, [create a role for fine\-grained access control](#fgac-roles), and [map the IAM role to it](#fgac-mapping)\. In this case, the new role needs the following permissions:

```
{
  "cluster_permissions": [
    "cluster_composite_ops",
    "cluster_monitor"
  ],
  "index_permissions": [{
    "index_patterns": [
      "firehose-index*"
    ],
    "allowed_actions": [
      "create_index",
      "manage",
      "crud"
    ]
  }]
}
```

Permissions vary based on the actions each service performs\. An AWS IoT rule or AWS Lambda function that indexes data likely needs similar permissions to Kinesis Data Firehose, while a Lambda function that only performs searches can use a more limited set\.

## REST API differences<a name="fgac-rest-api"></a>

The fine\-grained access control REST API differs slightly depending on your Elasticsearch version\. Prior to making a `PUT` request, make a `GET` request to verify the expected request body\. For example, a `GET` request to `_opendistro/_security/api/user` returns all users, which you can then modify and use to make valid `PUT` requests\.

On Elasticsearch 6\.*x*, requests to create users look like this:

```
PUT _opendistro/_security/api/user/new-user
{
  "password": "some-password",
  "roles": ["new-backend-role"]
}
```

On Elasticsearch 7\.x, requests look like this:

```
PUT _opendistro/_security/api/user/new-user
{
  "password": "some-password",
  "backend_roles": ["new-backend-role"]
}
```

Further, tenants are properties of roles in Elasticsearch 6\.*x*:

```
GET _opendistro/_security/api/roles/all_access

{
  "all_access": {
    "cluster": ["UNLIMITED"],
    "tenants": {
      "admin_tenant": "RW"
    },
    "indices": {
      "*": {
        "*": ["UNLIMITED"]
      }
    },
    "readonly": "true"
  }
}
```

In Elasticsearch 7\.x, they are objects with their own URI:

```
GET _opendistro/_security/api/tenants

{
  "global_tenant": {
    "reserved": true,
    "hidden": false,
    "description": "Global tenant",
    "static": false
  }
}
```

For documentation on the 7\.*x* REST API, see the [Open Distro for Elasticsearch documentation](https://opendistro.github.io/for-elasticsearch-docs/docs/security/access-control/api/)\.

**Tip**  
If you use the internal user database, you can use [curl](https://curl.haxx.se/) to make requests and test your domain\. Try the following sample commands:  

```
curl -XGET -u 'master-user:master-user-password' 'domain-endpoint/_search'
curl -XGET -u 'master-user:master-user-password' 'domain-endpoint/_opendistro/_security/api/user'
```