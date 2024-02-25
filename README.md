# Resource-Balance Workloads

## Overview
Cloud vendors offer platform-as-a-service (PaaS) OLAP databases that employ a range of pricing models. Two of the most common are pay-per-byte, where users pay for the amount of data their queries scan--such as Google BigQuery--and pay-per-compute, where users pay for the amount and duration of allocated compute resources--such as AWS Redshift. 

These pricing models are beneficial for different types of OLAP queries. An IO-bound query scanning large amounts of data but performing less computationally-intensive operators will be cheaper if billed per-compute rather than per-byte. Conversely, a CPU-bound query will be cheaper in a per-byte pricing model. 

The *balance* of CPU-bound and IO-bound queries in a workload can change what pricing model will be most effective for a workload, and could create an opportunity to execute a workload across multiple cloud databases to take advantage of different pricing models to save money. 

We create three workloads on the **TPC-DS** dataset that have different balances of CPU- and IO-bound queries to evaluate the money saving potential for multi-cloud execution of analytics SQL workloads. We name these workloads `CPU`, `IO`, and `Mixed`, referring to their relative balance of query types, i.e. `CPU` has more CPU-bound queries than IO-bound queries. 

## Usage Guide
### Adapting to Other SQL Syntaxes
These queries are written to be applicable to multiple databases SQL syntaxes, but there is unfortunately multiple conflicting SQL syntaxes present in the cloud. Notably, these queries will need some adjustment to execute in Google BigQuery. There are 2 primary text changes that need to happen, and we may push BigQuery compatible versions in the future:

1. BigQuery requires tables to be qualified with the *dataset* that data is stored in. For example, if your table `foo` is stored in a `tpcds` dataset, your SQL query must look like `SELECT * FROM tpcds.foo;`
2. For recursive CTEs, BigQuery requires that there not be schema on the CTE. For instance, `WITH foo(start, next, cnt) AS ....` would be invalid syntax. Instead users must write `WITH foo AS...` and name columns via explicit aliases on column names.  

## Workload Characteristics
There are queries that are present in multiple of the 3 workloads; however the three workloads are distinct. As discussed, the `CPU` workload is more CPU-bound as a whole because it contains more highly CPU-bound queries, whereas the `IO` workload contains more IO-bound queries and fewer CPU-bound queries. The `Mixed` workload tries to have a mix of heavily CPU- and IO-bound queries, as well as some queries that use a similar amount of both resources.

### What Queries Do and How They Were Created
We author queries which match customers based on purchase history and find neighborhoods of matching customers for recommendation tasks--either matching customers who purchased the same item or customers who purchased items similar in price to one another. We vary the channel used for matching--using either the `store_sales`, `catalog_sales`, or `web_sales` tables--and we vary other predicates on either the items or customers. We also author queries that look for triangles among customers based on purchase history.

These tasks require operators that are perform CPU-heavy operations on smaller sets of data, such as multiple self-joins, recursive CTEs, and complex, non-equality join predicates which will consume a lot of compute resources while remaining cheaper in a pay-per-byte pricing model.

Additionally, we use some existing TPC-DS queries, which are highly IO-bound, along with these CPU-bound queries to create the three Multi-Cloud Workloads. 
