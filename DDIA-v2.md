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
- Can _vertical scale_ get a faster machine with more CPU cores, more RAM, and more disk space.
- Increasing parallelism on a single machine is called a _shared-memory architecture_ since it uses the same RAM.
  - The cost here grows faster than linearly!
- _Shared-disk architecture_ is several machines (each with their own RAM) that share disks, connected via some fast network.
  - Contention adn overhead of locking limit scalability
- _Shared-nothing architecture_ aka _horizontal scaling_ aka _scaling out_ where each machine has its own CPU, RAM, and disks. Coordination is done at the software level. Pros are:
  - It can scale linearly.
  - It can use any hardware, (cheaper price perhaps).
  - It can easier scale up and down (just increase number of machines).
  - Better fault tolerance by distributing geographically.
- Downsides of shared-nothing architectures are complexity and requiring explicit sharding

### Principles for Scalability
- Scaling architecture is very dependent on the specific application, no one-size-fits-all.
- Likely necessary to rethink architecture ever order of magnitude, usually best to not plan one order of magnitude in advance.
- Good idea generally to break a system into smaller components (microservices, sharding, stream processing, etc), with the challenge being knowing where to draw the line
- Also, keep it simple when possible.

## Maintainability
- The majority of cost in software is in its ongoing maintenance: bugs, operations, investigations, adapting to platforms, new features, tech debt.
- Maintenance can be complex: outdated tech, knowledge loss with people leaving, fixing others' mistakes.
- Try to design software with maintenance in mind, and it'll lower costs in the long run.

### Operability: Making Life Easy for Operations
- Good software cannot run reliably with bad operations.
- Manual operations will always be necessary, automation will never cover every case.
- Can have too much automation: can be harder to troubleshoot if it goes wrong.
- Can help operations by:
  - Having monitoring tools
  - Avoid dependency on individual machines
  - Documentation
  - Good default behavior, and allowing overrides
  - Self-healing (when appropriate)
  - Predictable behavior

### Simplicity: Managing Complexity
- Rather subjective.
- _Essential complexity_ is inherent to the domain/problem, _accidental complexity_ arises because of limitations of the tooling.
- Abstraction is the best tool we have against complexity.

### Evolvability: Making Change Easy
- Adaptibility is closely linked to simplicity and abstractions. Loosely coupled systems are easier to modify.
- Irreversibility is difficult (e.g. migrating from one database to another). Minimizing irreversibility improves flexibility.

# Chapter 3: Data Models and Query Languages
1. Real world is modeled in terms of objects or data structures adn APIs to manipulate them.
2. These objects are expressed in a general-purpose data model, like JSON/SML, DB tables, graph edges/verticies.
3. DB engineers decided on a way to represent document/relational/graph data as bytes/memory/disk/network.
4. Hardware engineers represent bytes as electrical currents, etc.
- A _declarative_ query language allows description of the pattern of the data (source, transformation, grouping, sorting, etc.) but **not how** this is achieved. This is different than most programming languages which describe how something should be done.

## Relational vs Document Models
- SQL is the most popular: data is organized into _relations_ aka _tables_ with each relation an unordered collection of _tuples_ aka _rows_.
- SQL became tool of choice in the mid-80s.
- SQL reigns supreme with many coup attempts. NoSQL in the 2010s popularlized the _document model_ which represents data as JSON.

### The Object-Relational Mismatch
- The translation layer between objects in an OOP and the database objects is called an _impedance mismatch_.

#### Object-relational mapping
- An _Object-relational mapping (ORM)_ is a framework that reduce the boilerplate code for this translation layer. E.g. Hibernate and ActiveRecord.
- Drawbacks of ORMs:
  - Complex, and can't completely hide differences, leaving a lot of work still for devs.
  - Generally used only for OLTP app development.
  - ORMs usually only work with relational OLTP dbs, lacking support for search engines, graph databases, and NoSQL systems.
  - Customizing the ORM's schema and query generation can be complex and negative.
  - ORMs make it easy to accidentally write inefficient queries, like a N+1 query problem.
- Benefits:
  - Reduces boilerplate code, even if complicated queries need to be handled outside the ORM
  - Caching features
  - Help with managing schema migrations

#### The document data model for one-to-many relationships
- JSON representation has better _locality_, i.e. fewer queries/joins needed.
- Best for actual _one-to-few_ relationships, where there is not massive branching.

### Normalization, Denormalization, and Joins
- _Normalization_ is storing data as an ID which links to some other centralized data: ID will be used everywhere while the data stays just in one location / one copy.
- Benefits of normalization: consistent style/spelling, unambiguity, ease of updating, localization support, search functionality.
- Downside to normalization: extra lookup(s) required, less human readable.

#### Trade-offs of normalization
- Normalized data is faster to write but slower to query. Denormalized data is faster to query but much more expensive to write (usually).
- Not all databases offer atomicity across multiple documents
- Normalization usually better for OLTP systems, denormalization better for analytical systems (since read queries is the main concern).
- Normalization also better for small to moderate systems, but for very large systems joins become problematic.
- Denormalization is also difficult if the data changes often (like likes/comments on a post that has been denormalized into a timeline)

### Many-to-One and Manny-to-Many Relationships
- Usually represented with an _associative table_ or _join table_, which has a record for each connection.
- Many-to-one and many-to-many relationships don't work well in the Document model.

### Stars and Snowflakes: Schemas for Analytics
- A _star schema_ has a "center" _fact table_ where each "fact" represents an event. Each fact then has foreign keys to other _dimension tables_. Values in the fact record should only be values unique to that event.
- A _snowflake schema_ is still a star schema, but broken down even further (a dimension table is normalized). Star schemas are easier to work with usually but larger (less normalized).
- These schemas consist of mostly many-to-one relationships.
- The schema can be further denormalized and have no dimension tables (i.e. precomputing joins), known as _one big table (OBT)_. More space required, but sometimes faster queries.
- Denormalization is easier in analytics because the data is a log and not going to be overwritten.

### When to Use Which Model
- In favor of document model: schema flexibility, locality performance increases, more close to the application's object model (sometimes).
- In favor of relationship model: better support for joins, many-to-one, and many-to-many relationships.
- Easier to reference "nested" items in a relationship model.
- If a record has an ordered list of nested items, sometimes document model is easier for that (possible in relationship model, but a little more complicated).

#### Schema flexibility in the document model
- Document model can be thought of as _schema on read_ where schema has to be enforced upon reading a record.
- Relationship model can be thought of as _schema on write_ where schema is enforced upon writing to the db.
- When modifying a schema, its easier for document model, but the application code will have to support previous versions. For the relationship model, a migration is needed, which is normally easy but can be slow on large tables, sometimes requiring a lot of copying.
  - E.g. splitting a column, instead of having a migration do it (and update every record at migration time), could have a write-on-read in code.
- Schemaless is best if documents don't have the same structure

#### Data locality for reads and writes
- If typically need to load an entire document (with nested items), locality can make a big difference.
- Locality is a burden if writing is common **and** documents are large.
- SOme relational dbs offer the same, like Google's Spanner, allowing the schema to declare that a table's rows should be interleaved (nested) within a parent table. Oracle has the same with "multi-table index cluster tables". "wide-column" data model in Google's Bigtable. "column families" in HBase and Accumulo.

#### Convergence of document and relational databases
- They've converged over time: relational dbs adding JSON types and query operators and the ability to index properties inside documents. Document dbs add support for joins, secondary indexes, and declarative query languages.

## Graph-Like Data Models
- Can be better if data has many many-to-many relationships.
- A _node_, _entity_, or _vertice_ is a record like we'd think of in a relational/document db.
- An _edge_, _relationship_, or _arc_ is a connection between nodes/entities/vertices.

### Property Graphs
- A vertex contains: 
  - an ID
  - a label (to describe the type of object)
  - a **set of outgoing edges**
  - a **set of incoming edges**
  - Some collection of properties
- An edge contains:
  - an ID
  - starting vertex (_tail vertex_)
  - ending vertex (_head vertex_)
  - label to describe the kind of relationship
  - Some collection of properties
- Aspects of this model:
  - Any vertex can have an edge with any other vertex, no restrictions
  - Easy to find both incoming and outgoing edges and thus traverse the graph both forward and backward
  - Using labels we can store different kinds of vertices and different kinds of relationships in a single graph and maintain a clean model.
- This allows for quite a lot of flexibility in data modeling, and unlimited extensibility (add more facts/relationships).

### The Cypher Query Language
- Supported by Memgraph, KùzuDB, Amazon Neptune, Apache AGE (with storeage in PostgreSQL).
- Each vertex has a symbolic name (id), e.g. `usa` or `idaho`,.
- Vertices look like `(idaho :Location {name:'Idaho', type:'state'})` (label is Idaho, has 2-field property set).
- Edges use arrow notation like `(idaho) -[:WITHIN] -> (usa)` (edge has label of WITHIN).
- E.g. how to find people who emigrated from the US to Europe:
```
MATCH
  (person) -[:BORN_IN]->  () -[:WITHIN*0..]-> (:Location {name:'United States'}),
  (person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (:Location {name:'Europe'})
RETURN person.name
```
  - `MATCH` says "find all vertices that meet all of the below requirements"
  - Person that is BORN_IN any vertex `()` that connects via any number of `WITHIN` edges (zero to infinity) to some vertex labeled "Location" with property `name:'United States'` (and lives in "Europe").
  - For all matching vertices, return the `person.name`.

### Graph Queries in SQL
- In theory could query the same way with SQL in a relational database, but usually relational DB queries know which joins are needed (and how many), whereas graph querying excels when the number of edges/joins is not fixed/known in advance.
- Can be done in SQL with the `WITH RECURSIVE` syntax, e.g.
```SQL
WITH RECURSIVE
  in_usa(vertex_id) AS (
      SELECT vertex_id FROM vertices
        WHERE label = 'Location' AND properties->>'name' = 'United States'
    UNION
      SELECT edges.tail_vertex FROM edges
        JOIN in_usa ON edges.head_vertex = in_usa.vertex_id
        WHERE edges.label = 'within'
  ),

  in_europe(vertex_id) AS (
      SELECT vertex_id FROM vertices
        WHERE label = 'location' AND properties->>'name' = 'Europe'
    UNION
      SELECT edges.tail_vertex FROM edges
        JOIN in_europe ON edges.head_vertex = in_europe.vertex_id
        WHERE edges.label = 'within'
  ),

  born_in_usa(vertex_id) AS (
    SELECT edges.tail_vertex FROM edges
      JOIN in_usa ON edges.head_vertex = in_usa.vertex_id
      WHERE edges.label = 'born_in'
  ),

  lives_in_europe(vertex_id) AS (
    SELECT edges.tail_vertex FROM edges
      JOIN in_europe ON edges.head_vertex = in_europe.vertex_id
      WHERE edges.label = 'lives_in'
  )
SELECT vertices.properties->>'name'
FROM vertices
JOIN born_in_usa     ON vertices.vertex_id = born_in_usa.vertex.id
JOIN lives_in_europe ON vertices.vertex_id = lives_in_europe.vertex_id;
```
- The `in_usa` first finds the `vertex_id` of the record that has property `'name' = 'United States'`, then selects/unions recursively on outgoing edges (places in the US) to come up with a list of **all** the locations within the US.
- Does the same for Europe.
- The `born_in_usa` then gets the set of `vertex_id`s for all people `born_in` the set of vertices from `in_usa`.
- Does the same for Europe, but with `lives_in`.
- Then finally, with all people `born_in_usa` and all people `lives_in_europe`, select the name of vertices that inner join on both.
- So... way more complicated with SQL.
- The Graph Query Language (GQL) ISO standard, based on Cypher is not yet widely adopted but hopefully will bring greater uniformity in coming years.

### Triple Stores and SPARQL
- A record has 3 pieces of information, very similar to the property graph model: `subject`, `predicate`, `object` (i.e. `tail_vertex_id`, `relationship_label`, `head_vertex_id`).
- The `subject` is always a vertex in a graph, the `object` can be another vertex, or a primitive datatype.
  - If a primitive datatype, basically then like the property set in a vertex in the property graph model.
- Can be expressed in the _Turtle_ language (a subset of _Notation3 (N3)_). (Not covered here).
- The Turtle language encodes data in a model called the _Resource Description Framework_. (Not covered here).

#### The SPARQL query language
- Pronounced "sparkle".
- Same example as above looks like:
```SPARQL
PREFIX : <urn:example:>

SELECT ?personName WHERE {
  ?person :name ? personName.
  ?person :bornIn / :within* / :name "United States".
  ?person :livesIn / :within* / :name "Europe".
}
```
- SPARQL is supported by Amazon Neptune, AllegroGraph, Blazegraph, OpenLink Virtuoso, Apache Jena, and other triple stores.

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