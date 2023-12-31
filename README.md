<a name="readme-top"></a>

[![LinkedIn][linkedin-shield]][linkedin-url]



<!-- PROJECT LOGO -->
<br />
<div align="center">
  <img src="readme_files/logo.png" alt="Logo" width="80" height="80">
  <h3 align="center">Elasticsearch Deep Dive Case Study</h3>
</div>



<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
	<li><a href="#about-the-project">About The Project</a></li>
	<li><a href="#business-case">Business Case</a></li>
	<li>
	  <a href="#technical-documentation">Technical Documentation</a>
	  <ul>
		<li>
		  <a href="#1-installation-and-configuration">1. Installation and Configuration</a>
		</li>
		<li>
		  <a href="#2-security">2. Security</a>
		  <ul>
			<li>
			  <a href="#a-create-our-own-pki-public-key-infrastructure">a. Create Our Own PKI (Public Key Infrastructure)</a>
			</li>
			<li>
			  <a href="#b-encrypt-the-transport-network">b. Encrypt The Transport Network</a>
			</li>
			<li>
			  <a href="#c-set-built-in-user-passwords">c. Set Built-in User Passwords</a>
			</li>
			<li>
			  <a href="#d-encrypt-the-client-network">d. Encrypt The Client Network</a>
			</li>
			<li>
			  <a href="#e-define-roles">e. Define Roles</a>
			</li>
			<li>
			  <a href="#f-create-users">f. Create Users</a>
			</li>
		  </ul>
		</li>
		<li>
          <a href="#3-index">3. Index</a>
          <ul>
			<li>
			  <a href="#a-define-indices">a. Define Indices</a>
			</li>
			<li>
			  <a href="#b-bulk-index-data">b. Bulk Index Data</a>
			</li>
			<li>
			  <a href="#c-perform-crud-operations">c. Perform CRUD Operations</a>
			</li>
		  </ul>  
        </li>
		<li>
          <a href="#4-query">4. Query</a>
          <ul>
			<li>
			  <a href="#a-execute-non-analyzed-search-queries">a. Execute Non-Analyzed Search Queries</a>
			</li>
			<li>
			  <a href="#b-execute-analyzed-search-queries">b. Execute Analyzed Search Queries</a>
			</li>
			<li>
			  <a href="#c-define-metric-aggregations">c. Define Metric Aggregations</a>
			</li>
            <li>
			  <a href="#d-define-bucket-aggregations">d. Define Bucket Aggregations</a>
			</li>
			<li>
			  <a href="#e-define-sub-aggregations">e. Define Sub-Aggregations</a>
			</li>
			<li>
			  <a href="#f-define-pipeline-aggregations">f. Define Pipeline Aggregations</a>
			</li>
		  </ul>  
        </li>
		<li>
          <a href="#5-administration">5. Administration</a>
          <ul>
			<li>
			  <a href="#a-cat-apis">a. CAT APIs</a>
			</li>
			<li>
			  <a href="#b-monitor-your-cluster">b. Monitor Your Cluster</a>
			</li>
			<li>
			  <a href="#c-diagnose-and-repair-shard-issues">c. Diagnose and Repair Shard Issues</a>
			</li>
		  </ul>  
        </li>
	  </ul>
	</li>
	<li><a href="#contact">Contact</a></li>
	<li><a href="#references">References</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

<img src="readme_files/cover.jpg" alt="Cover Image">

* Project Name: Elasticsearch Deep Dive Case Study
* Version: v1.0.0
* Organization Department: Technology


### Business Case
In this project, I had a hands-on experience to deploy, secure, monitor, and 
troubleshoot an Elasticsearch cluster from scratch.

Following the Elasticsearch Deep Dive course on "A Cloud Guru", I went through the 
practical aspects of working with Elasticsearch. I deployed and secured a multi-node 
Elasticsearch cluster, learnt how to ingest data, and master search and 
aggregation techniques. 
I had also hands-on experience on essential administration tasks such as 
monitoring and troubleshooting.

<p align="right">(<a href="#readme-top">back to top</a>)</p>


## Technical Documentation

### 1. Installation and Configuration
I created an architecture of three distinct nodes:
* One master node. It also has an instance of Kibana on it.
* Two data nodes.

##### Master just added data nodes into the cluster:
<img src="readme_files/master-added-data-nodes.jpg">

##### Validate Elasticsearch on the master node:
<img src="readme_files/basic-setup-check-on-master.jpg">

##### All available "_cat" API endpoints:
<img src="readme_files/_cat_api_endpoints.jpg">

##### Nodes Endpoint:
<img src="readme_files/nodes_endpoint.jpg">

##### Nodes Endpoint Through Kibana Console:
<img src="readme_files/nodes_endpoint_kibana_console.jpg">

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### 2. Security

##### a. Create Our Own PKI (Public Key Infrastructure)

The first thing that we have to do before encrypting the networks of our clusters,
when need to create some certificates. We need to create our own PKI (Public Key
Infrastructure). Note that to have a series of certificates, one for each node,
all the certificates have to trust each other; so they all have to be signed by
the certificate authority.
So, what we will do is:
* Generate a certificate authority (CA). Also, protect the CA with a passphrase. That
way, anyone wants to sign node certificate with that CA will need to provide that
passphrase:

<img src="readme_files/generate-ca.jpg">

* Use the CA to sign node certificates for each node in our cluster, and we're going
to pass in the identifiable information (DNS and IP) of each node so that 
we can support full verification. That way, the certificate of master-1 node 
can't be used by anyone that want to join in our cluster; because it's only 
going to work on the master-1 node. Furthermore, we will protect the node certificate
with a passphrase. That way, not only does this certificate have to be used on a host
with this DNS and IP address, but in order to use it as an application, you have
to know the passphrase for this certificate. So this is by far, the most secure
way to use PKI. Now, we are sure that not only we're going to encrypt the 
communications using the certificate, but the actual use of the certificate is 
as protected as it can be.

<img src="readme_files/master-1-cer-1.jpg">

<img src="readme_files/master-1-cer-2.jpg">


* Generate certificates the same way as well for data-1 and data-2:

<img src="readme_files/elastic-certificates.jpg">


* Now, we will use this PKI (Public Key Infrastructure) that we just built to 
encrypt the transport network, which is the node-to-node communication network of
Elasticsearch, and encrypt as well the client network which is the network that
we interact with using the rest APIs.

##### b. Encrypt The Transport Network

* The Transport Network is the node-to-node communication of the cluster. So, we
as clients, don't have any direct interaction, with this network. However, this
traffic could be intercepted and interpreted to essentially leak data about the cluster.

* Encrypt Transport Network Configurations on The Master Node:

<img src="readme_files/encrypt-transport-on-master.jpg">

* We do the exact same for data-1 and data-2 as well.

* After encrypting the transport network and enabling the security pack, we
no longer can query the API without authentication as we did before:

<img src="readme_files/after-encrypt-transport.jpg">

* So, we need to set built-in user passwords to connect to our
cluster.

##### c. Set Built-in User Passwords

* I used the elasticsearch-reset-password utility to reset the password for all
available users.

<img src="readme_files/reset_users_password.jpg">

* Then set the elasticsearch user in kibana.yml configurations file to allow
Kibana to communicate with Elasticsearch with user "kibana".
* Now, Kibana can communicate with Elasticsearch using the user "kibana".
* We also can log into Kibana using user "elastic"

<img src="readme_files/kibana_login.jpg">


##### d. Encrypt The Client Network

* To enable the client Network Encryption and make the cluster listen on HTTPS
instead of HTTP, we will use the same certificates that we used to encrypt the 
transport network.

* We edit the HTTP SSL security configurations in elasticsearch.yml on the master node:

<img src="readme_files/client_network_encrypt_on_master.jpg">

* We do the same on the other data nodes as well.

<img src="readme_files/client_network_encrypt_on_data_nodes.jpg">

* We also need to edit Kibana configurations to make it communicate
with Elasticsearch over HTTPs:

<img src="readme_files/kibana_https_1.jpg">

<img src="readme_files/kibana_https_2.jpg">

* Now, our requests are fully encrypted, but since we used self-signed certificates,
they won't be trusted by any client by default.

<img src="readme_files/elasticsearch_https_request.jpg">

##### e. Define Roles

* Now that we have user access control enabled and protecting our cluster and 
kibana instance, we can now create our own custom roles and users.

* Create a read-only role called "read_only" which has a privilege to read all
indices.

<img src="readme_files/create_read_only_role.jpg">

* Query the read_only role info:

<img src="readme_files/query_role_read_only.jpg">

##### f. Create Users

* Create a user with the custom role that we've created "read_only"
and a built-in role "kibana_user"; to allow the user to log into
Kibana

<img src="readme_files/create_user_with_roles.jpg">

* Query the user that we've just created it

<img src="readme_files/query_the_new_user.jpg">

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### 3. Index

#### a. Define Indices

* Create an index called "bank" and configure it to have one shard and one 
replica:

<img src="readme_files/create_bank_index.jpg">

* Create an index called "shakespeare", override the dynamic mapping of multiple
fields, and configure it to have one shard and one replica:

<img src="readme_files/shakespeare_index.jpg">

* Create an index called "logs" where we define a field of type geo point, which is
the longitude and latitude location of an IP address.

<img src="readme_files/logs_index.jpg">

* Check all the indices we have so far in our cluster

<img src="readme_files/query_all_indices.jpg">

#### b. Bulk Index Data

* Bulk indexing the bank index with a file of type New-Line Delimited JSON file:

<img src="readme_files/bulk_indexing_the_bank_index.jpg">

* Do the same for the shakespeare index:

<img src="readme_files/bulk_indexing_shakespeare_index_1.jpg">

<img src="readme_files/bulk_indexing_shakespeare_index_2.jpg">

* Do the same as well for the logs index:

<img src="readme_files/logs_index_1.jpg">

<img src="readme_files/logs_index_2.jpg">

* Check indices in the cluster

<img src="readme_files/indices_status_after_bulk_indexing.jpg">

#### c. Perform CRUD Operations

<img src="readme_files/crud_operations.jpg">


<p align="right">(<a href="#readme-top">back to top</a>)</p>

### 4. Query

#### a. Execute Non-Analyzed Search Queries

<img src="readme_files/non-analyzed-search.jpg">

#### b. Execute Analyzed Search Queries

* Standard Analyzer

<img src="readme_files/array_of_tokens_of_string.jpg">

* English Analyzer
  * The English analyzer removes unnecessary words like "the" and 
also normalizes some of these words

<img src="readme_files/english_analyzer.jpg">

* Analyzed Search vs. Non-Analyzed Search

<img src="readme_files/analyzed-vs-non-analyzed.jpg">

* Analyzed search with exact phrase

<img src="readme_files/analyzed_search_exact_phrase.jpg">

#### c. Define Metric Aggregations

* Aggregations Search

<img src="readme_files/aggs_search.jpg">

* Sum Aggregation Search

<img src="readme_files/sum_aggs_search.jpg">

#### d. Define Bucket Aggregations

<img src="readme_files/bucket_aggs.jpg">

* Another bucket aggregations type is **Date Histogram**. It's very useful for
time series data.

* If we look at the "logs" index that we created earlier, we can see that our
logs data is essentially time series data. So, we have an "@timestamp" field:

<img src="readme_files/logs_time_series_data.jpg">

* Let's figure out, per day, how many log events I had. To do that, we need to 
bucketize the data per day. In order to do that, we're going to do a date
histogram.

<img src="readme_files/date_histogram_bucket.jpg">

#### e. Define Sub-Aggregations'

* By now, we know how to do metric aggregations, and bucket aggregations. let's
combine the two into sub-aggregations or nested-aggregations.

<img src="readme_files/date_histogram_bucket.jpg">

* Sample nested aggregations

<img src="readme_files/cities_per_state_aggs.jpg">

* Let's take a look on the logs index

<img src="readme_files/logs_index_view.jpg">

* Let's figure out the sum of bytes transferred for each extension that's have
been requested.

<img src="readme_files/bytes_per_extension_aggs.jpg">

* Let's also figure out which one of these file extensions has the biggest average
file size.

<img src="readme_files/avg_file_size_per_extension.jpg">

#### f. Define Pipeline Aggregations

* Pipeline aggreations are aggregations that take as the input, the output of 
another aggregation.
* So, in order to have a pipeline aggregations, you have to first have a 
non-pipeline aggregation that is generation some kind of output.
* There are two types of pipeline aggregations:
  * 1. Parent pipeline aggregations
  * 2. Sibling pipeline aggregations

* Parent pipeline aggregations

  * Let's calculate the cumulative sum of unique clients per hour.
  
  * Step 1: Get a bucket per hour using date histogram aggregation
    
<img src="readme_files/avg_file_size_per_extension.jpg">
        
  * Step 2: Create a sub-aggregation to get unique clients per hour
    
<img src="readme_files/unique_clients_per_hour.jpg">
    
  * Step 3: Get the cumulative sum of unique clients per hour 
  
<img src="readme_files/cumulative_sum_of_parent_agg.jpg">

* Sibling pipeline aggregations

  * The sibling aggregation is outside the buckets. While the parent
  aggregations are adding metrics (data) to each bucket, the sibling aggregation
  is going to create its own separate aggregation that is a sibling to the 
  buckets.

<img src="readme_files/sibling_aggs.jpg">

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### 5. Administration

#### a. CAT APIs

<img src="readme_files/_cat_api_adminstration.jpg">

#### b. Monitor Your Cluster

* Enable self-monitoring through API

<img src="readme_files/enable_self_monitoring.jpg">

* Access monitoring dashboard in Kibana

<img src="readme_files/cover.jpg">


#### c. Diagnose and Repair Shard Issues

* In this section, I will actually break some of the indices, kind of deallocate
them in some way, and then show how to use the **cluster allocation explain API** to
basically tell us what's wrong.

* The **Explain API** basically does the troubleshooting for you.

* If we run the API without any parameters, it will automatically look for the 
first unallocated shard, and then explain why it's not allocated. Here we see an
error because we currently don't have any unallocated shards

<img src="readme_files/explain_api_no_unallocated.jpg">


* Here we see that the "logs" index is in a green state which means that all primary
and replica shards are allocated

<img src="readme_files/logs_index_status_green.jpg">


* Let's go ahead and allocate too many replicas for the log index

<img src="readme_files/allocate_too_many_shards.jpg">

* Now, if we check the shards status of the "logs" index, we will see that there is
an allocated primary and replica shards, and two unallocated shards. But, why?
Simply, because we have two data nodes. What are the maximum replicas that our 
cluster can support? One only! Because, the primary shard will be on a data node,
and the other replica shard will be on the other data node. We already know that 
Elasticsearch won't create a replica on the same server of the primary shard; because
it's pointless to have a primary and replica shards on the same servers. So, in our
case, Elasticsearch simply can't find enough data nodes to allocate the extra two
replica shards that we've just increased.

<img src="readme_files/unassigned_shards.jpg">

* If we check the explain API, it will tell us that for each data node we have,
there is already a copy of that shard that exists. So we now know that we can't
create that many shards for the index.

<img src="readme_files/explain_api_unallocated_shard.jpg">

* Also, if we check the health of the cluster now, we will find that the status
is Yellow. That means all primary shards are allocated. However, some replica
shards aren't allocated, which doesn't affect the cluster currently.

<img src="readme_files/cluster_health_yellow.jpg">

* Now, we can decrease the number of the replicas back to one, and elasticsearch
will be able to allocate the primary and replcia shards on the two available 
data nodes that we have.

<img src="readme_files/status_back_to_green.jpg">

* Another issue that we can deliberately make is to assign the index shard on a
node that doesn't exist.

<img src="readme_files/allocate_on_missing_node.jpg">

* If we use the explain API on the primary shard that we've just allocated,
it will us that it can't remain on the current specified node in the settings
we've specified (data-3). However, it can't also move it to another node as well.

<img src="readme_files/shard_can't_be_allocated.jpg">


<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTACT -->
## Contact

Mohamed AbdelGawad Ibrahim - [@m-abdelgawad](https://www.linkedin.com/in/m-abdelgawad/) - <a href="tel:+201069052620">+201069052620</a>

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- References -->
## References

* Elasticsearch Deep Dive Course: https://www.pluralsight.com/cloud-guru/courses/elasticsearch-deep-dive
* “Hot-Warm” architecture: https://www.elastic.co/fr/blog/hot-warm-architecture
* Elasticsearch Node Configuration: https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html
* Access Denied You are not authorized to access Monitoring: https://stackoverflow.com/questions/71552021/access-denied-you-are-not-authorized-to-access-monitoring

<p align="right">(<a href="#readme-top">back to top</a>)</p>


<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/m-abdelgawad/
