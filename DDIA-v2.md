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
- 

### Operations in the Cloud Era

## Distributed vs Single-Node Systems

### Problems with Distributed Systems

### Microservices and Serverless

### Cloud Computing vs Supercomputing

## Data Systems, Law, and Society

# Chapter 2: Defining Nonfunctional Requirements

# Chapter 3: Data Models and Query Languages

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