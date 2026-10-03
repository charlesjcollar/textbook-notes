# Designing Data-Intensive Applications
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

## Characterizing Transaction Processing and Analytics
- An operation in an operational system usually looks up a small number of records by some key (called a _point query_).