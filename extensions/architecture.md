# Architecture

## Ingestion and Query Pipeline

The ingestion and query pipeline makes use of a disruptor pattern with an in-memory ring buffer to offer extremely high input rates.

![](/assets/docs_tritandb_ringbuffer.png)

## Query Processor
The query processor of TritanDB, as shown in the figure below, receives a query, $$q$$, from a ring buffer slot. A pre-compiled parser component with a specific query grammar parses the query to an abstract syntax tree (AST), a parse tree, which is passed to the next Match and Operate steps.

![](/assets/docs_tritandb_query_engine - Page 1.png)