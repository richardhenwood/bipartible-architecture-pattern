# B*ip*art*ib*le Architecture Pattern

A system architecture pattern that transparently directs HTTP traffic over Inifiniband

## B*ip*art*ib*le - a pattern for transforming HTTP traffic into IB traffic

A cloud native software platform is characterized by several [key principals and technologies](https://en.wikipedia.org/wiki/Cloud-native_computing) including (but not limited to) containers, microservices architecture, dev-ops, and infrastructure-as-code. Intra- and Inter-platform communication is typically HTTP. HTTP uses the TCP/IP (IP) networking standard. HTTP is a particularly good choice as it is easy to route, inspect, and proxy. HTTP has a mature ecosystem of powerful and capable tools that are universally available and well understood.

Large scale computer systems typically support IP. In some cases the network will also include a remote direct memory access (RDMA) alternative network. RDMA systems typically deploy the InifiniBand (IB) networking standard. IB is often chosen when bandwidth and latency are important considerations. Both IP and IB networking standards can happily co-exist. Protocols that use these networking standards typically have a strong affinity for either IP or IB standards. For example: HTTP has a strong affinity for IP networks. Cloud native platforms often preference HTTP for communication and as a consequence HTTP traffic will typically travel over IP.

## B*ip*art*ib*le - consisting of two parts, IP and IB

The Bipartible Pattern applies in situations where a cloud native software platform includes remote direct memory access (RDMA). Typically this system would deploy the InfiniBand (IB) networking standard. Without this pattern cloud native platform traffic typically traverses the IP network and does not use the IB network. The Bipartible Pattern transparently redirects the data-heavy traffic over the IB network. The pattern works because the Bipartible service can execute at any location on the network and has an identical experience of data storage. The service is typically in a 'single-node' configuration and treats the data storage as local. This pattern was initially developed to address the performance challenges of deploying scientific software containers at scale (i.e. [E4S](https://e4s.io/) ) 

## System Requirements

 + Network: an IB network is present.
 + Service layer: a global file system i.e. [Lustre](https://www.lustre.org/).,
 + Service that supports Bipartible Pattern.

## Bipartible Service Requirements

 + IB Storage: the Bipartible service must use IB. (As a simplifying assumption, I am just working with POSIX storage in the form of Lustre).

### Bipartible Pattern

<img width="1051" height="425" alt="bipartible drawio(1)" src="https://github.com/user-attachments/assets/94836854-c8df-40e0-8fa5-8cb009442b11" />

A practical example of this setup is a Kubernetes cluster (the 'Host') with a containerized process ('Workload process'). An example of a HTTP service (Bipartible Service) is S3 or a container registry. An example of a POSIX global filesystem is Lustre.

With a naive setup a process running on a host makes a request for a HTTP service. The host uses the configured IP routes to direct the request over the IP network. The service endpoint on the IP network receives and processes the request. The service obtains the required data from a POSIX global filesystem. The data is returned back to the process on the host via the IP network. 

With the Bipartible pattern the process on the host makes a request for a HTTP service. The host uses the configured IP routes to direct the request to a service running on the host. The service endpoint on the host receives and processes the request. The service obtains the required data from a POSIX global filesystem available over IB. The data is returned to the process using in-memory communication on the host.

### Hybrid-Bipartible Pattern

<img width="1051" height="425" alt="bipartible-hybrid drawio(1)" src="https://github.com/user-attachments/assets/d1b52470-86c3-4673-996f-8d220602dd1a" /> 

With the Hybrid-Bipartible pattern the Bipartible Service requires additional storage beyond just a POSIX global filesystem. Bipartible Service uses POSIX global filesystem for the bulk data. Additional storage is accessed over IP and is used for lightweight metadata. The Hybrid-Bipartible pattern is identical to the Bipartible pattern with the exception that some traffic will travel from the Bipartible service over IP.

An example service that requires the Hybrid-Bipartible pattern is Project Quay container registry. Project Quay requires Redis and SQL storage in addition to bulk storage.

## Services Supporting Bipartible Pattern

### S3

#### Garage

[Garage](https://garagehq.deuxfleurs.fr/) is an open-source distributed object storage service tailored for self-hosting.

| Status | Scale test | Configuration | Notes |
| Works | Proof of concept | Single node | Bucket creation fails: `Error: ListBuckets returned InternalError (500): Internal error: Layout not ready` |

### Container Registry

#### Project Quay

[Project Quay](https://www.projectquay.io/) is the open source distribution of Red Hat Quay optimized for the secure distribution of container images no matter whether you are scaling.

| Status | Scale test | Configuration | Notes |
| TODO |  |  | |
