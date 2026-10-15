# Chapter 1: Trade-Offs in Data Systems Architecture
- An application is _data-intensive_ if data management is a primary challenge. 
  - Different from _compute-intensive_ where the challenge is parallelizing large computation.
- Challenges include, for large amounts of data, storing and processing, managing changes, consistency during failures, concurrency, and availability.
- Many ways to do things, each with trade-offs.
- A key challenge is different people at the same organization need to do different things with the _same_ data. Different requirements from the data -> different approaches to handling/storing data.
- Some contrasting approaches (each with trade offs): Operational vs Analytical systems, Cloud vs On-Prem, Distributed vs Single-Node, Business Needs (Law, ethics, society, etc.)

## Operational vs Analytical Systems
- Engineers usually think in Operational Systems: reading and updating data for application purposes (users of the system).
- Business Analysists and Data Scientists just read (large amounts of) data for machine learning or business intelligence purposes.
- Thus _operational systems_ are the backend services and data infrastructure **where data is created**, and small operations of reading and writing occur (usually).
- _Analytical systems_ are read-only copies of data from the operational systems, and are optimized for data processing/reading.
- _Data engineers_ integrate operational and analytical systems, and have general organizational responsibility.
- _Analytics engineers_ model and transform data to make better use of them.

### Characterizing Transaction Processing and Analytics
- An operation in an operational system usually looks up a small number of records by some key (called a _point query_).
- Interactive applications are an access pattern known as _online transaction processing (OLTP)_.
- Analytics queries are called _Online analytical processing (OLAP)_
- OLTP systems mostly run a fixed set of queries baked into the application code (users can't just run random queries).
- However, analytical databases give users freedom to write arbitrary SQL queries by hand.

### Data Warehousing
- Reasons it's not great to query OLTP systems:
  - "data siloing", data spread out across non-joining systems.
  - Schemas good for OLTP are less good for Analytics.
  - Analytical queries are **expensive**
  - Security/compliance reasons
- Thus, port data over to a data warehouse, which stores data in a different way as to optimize analytics queries.
- Process of getting data into the data warehouse is known as _extract-transform-load (ETL)_, sometimes swapping "transform" and "load" steps (ELT).
- Especially useful when the OLTP system is an external SaaS product that cannot be queried in analytical ways.
- Some databases (_Hybrid Transactional/Analytical Processing (HTAP)_) offer both analytical and transactional processing, which is usually just two separate systems hidden behind a common interface.
- HTAP systems aren't an option when each microservice has its own database.... They only work well for certain cases that require large amounts of reading + updadting indivual records with low latency.

#### From data warehouse to data lake
- Data warehouse uses a relational model so data can be queried throguh SQL. A data lake is data in the form suitable for an ML model, e.g. vectorized.
- _features_ are a vector or matrix of numerical values that represent data.
- Transforming relational data into vectorized data is called _feature engineering_, and the goal is to maximize the performance of the trained model.
- A _data lake_ is a centralized data repository that holds a copy of any data useful for analytics. A data lake **simply contains files, without imposing format, model, or schema**. It's more flexible, and stored on regular, commoditized file storage like object stores.
- The datalake can be an intermediate stop from an operational system to a data warehouse through a generalized ETL process called a _data pipeline_. Thus a data contains the "raw" data.

#### Beyond the data lake
- Management has become more important, e.g. for governance, privacy, and compliance reasons (e.g. GDPR).
- Streams of events is important for analysis as well, e.g. time sensitive analytics (like fraud, abuse).
- Results of analytical systems can filter back into operational systems too, e.g. recommendation systems. Called _reverse ETL_.

### Systems of Record and Derived Data
- _System of record_ is aka the _source of truth_. Facts are represented exactly once, and often normalized.
- _Derived data_ is data transformed or processed from the source of truth, and **can be re-created from the original source**. Caches, denormalized values, indexes, materialized views, transformed data representations, and models all are derived data.
- Analytical systems are derived, operational systems usually have a mix of both (derived data here speeds up processing).
- Databases inherently are not defined as one category, it's about how it's used.

## Cloud vs Self-Hosting
- Things that are unique, or make your business/product **competitive** should be done in-house, non-core, routine, or commonplace items should be left to a vendor.
- Most control: in-house software/operations (e.g. application code).
- Medium control: Off-the-shelf software like self-hosted databases, frameworks.
- Least control: cloud services, SaaS.

### Pros and Cons of Cloud Hosting
- Cloud claims to be cheaper, but it can be cheaper to run things on-prem if you already know what you're doing and if your **load is predictable**.
- Outsourcing operations can be good because the vendor will have much more experience and it can be cheaper than hiring your own.
- Doing in house can allow for greater flexibility and tuning.
- Cloud services are excellent when load fluxuates a lot.
- Cloud services negative is loss of control in:  features, outages, debugging, performance and metrics, pricing, security, and politics even.

### Cloud Native System Architecture
- _Cloud native_ means an architecture designed to take advantage of cloud services, usually built from the ground up.

#### Layering of cloud services
- Most things need the same hardward: IP network, CPUs, RAM, a filesystem; thus most can run on a VM (or _instance_).
- Build upon low-level cloud services to create high-level services: e.g. a data warehouse (Snowflake) relies on S3 for data storage, and then other services rely on Snowflake
- If an existing higher-level solution exists, it's probably easier to just use that.

#### Separation of storage and compute
- Used to just use RAID disks; in the cloud, attached filesystems are ephemeral and disappear if the instance restarts, scales, drops, etc.
- Cloud providers do have "virtual disk storage" that can be detached from one instance and attached to a different one, which allows the running of traditional disk-based software (Amazon EBS, Azure managed disks, persistent disks in GCP). Can be hard to manage, and sensitive to network glitches.
- Usually just build on dedicated storage services (S3, GCS). 
- **This also means any processing on data requires transfering the data over network.**
- **This also means multiple processes could be reading/writing from the same data instance.**

### Operations in the Cloud Era
- Operations used to be managing individual machines, provisioning new machines, etc. Now operations uses APIs with the cloud to manage infrastructure.
- DevOps/SRE Philosophy places greater emphasis on:
  - Automating
  - Using ephemeral VMs and services
  - Enabling frequent application updates/deployments
  - Learning from incidents
  - Preserving organization's system knowledge
  - Choosing the correct resources to balance performance and cost and potential quotas.
  - Successfully integrating services.

## Distributed vs Single-Node Systems
- Any system that has several machines communicating over a network is distributed. Such as: two or more interacting users, different cloud services (cloud native / microservices).
- Benefits of distributed systems: fault tolerance, high availability, scalability, latency (deployed close to users), elasticity (scaling flexibility), specialized hardware, legal (country rules), sustainability (taking advantage of cheap power)

### Problems with Distributed Systems
- Any network call could fail, be dropped, or hang, and we don't know why (or if the receiver ever received it.)
- Network calls are inherently slower than anything in-process.
- Troubleshooting can be hard.
- Transactions are not atomic when spanning multiple microservices/databases
- Single machines are often much simpler and cheaper than a distributed system.

### Microservices and Serverless
- Software being divided into microservices has advantages: less coordination/conflicting when updating, abstracting (hiding implementation behind interfaces), and smaller software can be easier to handle individually.
- Complexity comes from microservices, like integration testing, and each service needs its own management/monitoring/on-call.
- Defining and changing contracts adds friction and difficulty, especially between teams
- _Serverless_ or _function as a service (FaaS)_ is code-execution charged on-demand.
  - Serverless infra providers usually impose a time limit and limit runtime environments, and "warming up" can be slow too.

### Cloud Computing vs Supercomputing
- Supercomputing is different from cloud computing:
  - Meant for computationally intense processes, like weather, rather than systems that service users with high availability
  - Supercomputing usually is a large batch job
  - Supercomputing maximizes speed wtih shared memory (less secure).
  - Supercomputing uses special network topoligies, not common IP and ethernet
  - Supercomputing isn't usually geographically distributed

## Data Systems, Law, and Society
- GDPR grants individuals the right to have data **erased** upon request, which needs to be considered in systems and presents new challenges.
- Cost-benefit needs to be considered when storing massive amounts of data, as well as risks of liability if hacked.
- Need to consider if the government demands data.
- All leads to sometimes considering _data minimization_ to simply not store some data to simplify things. But sometimes data needs to be stored for compliance/auditing reasons.

# Chapter 2: Defining Nonfunctional Requirements
- _Functional requirements_ are given, explicitly written down requirements. It is the functionality of the software, and what the software must do.
- _Nonfunctional requirements_ are not explicitly written down, or may "seem obvious". They are just as important as the app's actual functionality and not considering them can paralyze or break the app.

## Case Study; Social Network Home Timelines
- Simple case: social network.
  - 500 million posts per day (5800 posts per second)
    - Rate can spike to 150k posts per second
  - Average user follows 200 people
  - Average user has 200 followers (wide range though)

### Representing Users, Posts, and Follows
- If 10 million users are online at the same time, each "home" timeline query has to get, on average, 200 users info + latest 1000 posts. 
- If a timeline has to update every 5 seconds, that's 2 million queries per second. 
- This query gets 1200 rows (200 users + 1000 posts) so that's 2.4 billion lookups per second.

### Materializing and Updating Timelines
- Precompute timelines in a process called _materialization_: pre-computing and updating results. The timeline itself is called a _materialized view_.
- Materialization puts more work on writes.
- Balance is necessary to handle extreme cases (accounts with large number of followers, accounts following many people).

## Describing Performance
- _Response time_ is the elapsed time from the making of a request to the receiving of an answer (so + any latency).
- _Throughput_ is the number of requests per second the system is processing.
- When queueing is involved, response time can increase due to low throughput. As throughput approaches the max, queueing delays can sharply increase.
- Clients care about response time, engineers care about throughput (determines required computing resources / cost).
- A system is _scalable_ if maximum throughput can be significantly increased by adding computing resources.

#### Tools for handling an overloaded system
- A _retry storm_ can make the problem worse if long queue waits -> timeouts -> retries. 
- Tools to prevent overloading:
  - Exponential backoff.
  - A _circuit breaker_ on the client can tell the client to stop sending requests if known errors are being returned (closed: sends requests, open: not sending requests)
  - A _token bucket_ algorithm uses a bucket that has tokens added it to it at a regular interval, then tokens are removed upon processing/sending requests. If the bucket is empty, queue requests or reject them (until more tokens come in).
  - A server can start _load shedding_, i.e. proactively rejecting requests.
  - A server can request clients slow down (_backpressure_).

### Latency and Response Time
- The _response time_ is what the client sees.
- The _service time_ is the duration for which a service is actively processing a request.
- _Queueing delays_ can occur many places: in the process (e.g. waiting on IO or upstream), or networking queueing
- _Latency_ is the catchall term for all the time where a request is not being actively processed.
- Response time can be inconsistent due to many external (and sometimes random) factors: networks, garbage collection, cache miss, etc.
- A few slow requests can cause large queueing delays for many fast requests behind it, known as _head-of-line blocking_.
- It's important to measure response time from the client side.

### Average, Median, and Percentiles
- Best to describe response time in percentiles instead of mean/average.
- Median is the 50th percentile, better for describing "average" time. Often look at the _tail latencies_ like the 95th, 99th, or 99.9th percentiles (abbreviated _p95_, _p99_, and _p999_ respectively).

### Use of Response Time Metrics
- If a BFF calls multiple backend services, the response time of the BFF is the response time of the slowest backend service.
- A _Serice level objective (SLO)_ describes expected performance and availability of a service (e.g. 99th percentile under 1 second, and 99.9% of valid requests have a non-error response).
- A _Service level agreement (SLA)_ is a contract to describe what happens if the SLO isn't met (e.g. refund).

## Reliability and Fault Tolerance
- "Reliability" roughly means "continuing to work correctly when things go wrong".
- A _fault_ is when a certain _part_ of the system stops working correctly, like a single machine crashing or a single service having an outage.
- A _failure_ is when the system as a **whole** stops providing required service to the user. A failure is a breach of SLO.

### Fault Tolerance
- A system is _fault tolerant_ if it can provide service to users despite certain faults. Any part of a system where a fault causes a failure is a _single point of failure (SPOF)_.
- Fault tolerance has a limit, and it gets more expensive to be more fault tolerant.
- By introducing faults via _fault injection_ (like killing random processes without warning), you can continually test and exercise fault-handling parts of systems. This discipline is also known as _chaos engineering_.
- Sometimes need to prevent faults rather than tolerate them, like security, where unauthorized access cannot be tolerated ever.

### Hardwaree and Software Faults
- Expect your hardware to fail! Basically all hardware (CPUs, RAM, disks, network cables, electricity) can fail and will eventually. Natural causes can make other stuff fail too.

#### Tolerating hardware faults through redundancy
- First method to reduce failure rate of a system: have redundant individual hardware components.
- Best when component faults are independent (one fault doesn't trigger another fault).
- Cloud systems focus less on reliability of individual machines, and instead tolerate faulty nodes at the software level ("distributed" software).
- Even more effective when software is geographically separated / redundant.
- Systems that can tolerate the loss of entire machines can also handle planned downtime without any downtime, e.g. via a rolling upgrade.

#### Software faults
- All nodes will have the same bugs: high correlation.
- Software faults are harder to anticipate and can cause many more system failures.
- Types of software faults:
  - General bug in every node.
  - Runaway process that eats up a shared, limited resource.
  - Service becomes slow or unresponsive affects downstream services.
  - Integration bugs between services.
  - Cascading failures like overloading which brings down other components.
- No quick easy solution: best to: 
  - Carefully think about all assumptions and interactions.
  - Thoroughly test.
  - Ensure process isolation.
  - Allow processes to crash and restart (and assume they will).
  - Avoid feedback loops like retry storms.
  - Monitor and analyze behavior and metrics in production.

### Humans and Reliability
- Inevitable, and blaming doesn't solve anything.
- Best to try to just minimize the **impact**:
  - Rollback mechanisms.
  - Gradual rollouts of new code.
  - Detailed and clear monitoring and alerts.

## Scalability
- A new product with a small number of users will prioritize on system simplicity and flexibility rather than scalability.
- If an application becomes popular and load grows, **then** learn where the bottlenecks are and which dimensions you need to scale.

### Understanding Load
- Often measure a throughput: requests per second, GB of new data per day, etc. some _amount per time-unit_.
- Other statistics are important too, like:
  - Ratio of reads to writes in a db.
  - Hit rate on a cache.
  - Amount of data / items per user.
- Sometimes we care about the average load / case, sometimes peak/outlier/extreme/edge cases.
- When considering load increase, consider:
  - If load increases and system resources stay the same, how is the performance affected?
  - If load increases, how much resource increases is needed to keep performance the same?
- Need to balance these two: keep within SLAs while minimizing costs.
- _Linear scalability_ means adding x times of resources leads to x times higher throughput. Considered a good thing.
  - More likely, costs grow faster than linearly.

### Shared-Memory, Shared-Disk, and Shared-Nothing Architectures

### Principles for Scalability

## Maintainability

### Operability: Making Life Easy for Operations

### Simplicity: Managing Complexity

### Evolvability: Making Change Easy

# Chapter 3: Data Models and Query Languages
## Relational vs Document Models
### The Object-Relational Mismatch
### Normalization, Denormalization, and Joins
### Many-to-One and Manny-to-Many Relationships
### Stars and Snowflakes: Schemas for Analytics
### When to Use Which Model
## Graph-Like Data Models
### Property Graphs
### The Cypher Query Language
### Graph Queries in SQL
### Triple Stores and SPARQL
### Datalog: Recursive Relational Queries
### GraphQL
## Event Sourcing and CQRS
## DataFrames, Matrices, and Arrays

# Chapter 4: Storage and Retrieval

# Chapter 5: Encoding and Evolution

# Chapter 6: Replication

# Chapter 7: Sharding

# Chapter 8: Transactions

# Chapter 9: The Trouble with Distributed Systems

# Chapter 10: Consistency and Consensus

# Chapter 11: Batch Processing

# Chapter 12: Stream Processing

# Chapter 13: A Philosophy of Streaming Systems

# Chapter 14: Doing the Right Thing