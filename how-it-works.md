> ### IoT Time-series workloads are different.

_**Time-series workloads produced by streams from the Internet of Things \(IoT\) are different.**_ Time-series data is largely immutable. Writes primarily occur as new appends to recent time intervals, not as updates to existing rows. Both read and write workloads have a natural partitioning across both time and space. Hence, TritanDB utilises these properties to most efficiently store and process such IoT workloads.

> #### Of the total potential economic value the IoT enables, interoperability is required for 40 percent on average and for nearly 60 percent in some settings. - [Unlocking the potential of the IoT, McKinsey](http://www.mckinsey.com/business-functions/digital-mckinsey/our-insights/the-internet-of-things-the-value-of-digitizing-the-physical-world)

**Semantic interoperability is essential for achieving the economic and social impact of the IoT. **A rich model for representing metadata which contains the context of IoT time-series sensor data is required. Graph-based models are what almost every standards body and industry authority are working towards for representing metadata. TritanDB is built to support such rich models with the [Resource Description Framework \(RDF\)](https://www.w3.org/RDF/), state-of-the-art technology that drives the semantic web and machine-to-machine interoperability on the web.

<!-- toc -->

## The Nature of Time-series IoT Data

The majority IoT time-series data is flat, wide, numerical and either approximately periodic or non-periodic.

## Quantum Re-ordering Buffer \(QRB\)

Time-series data for the IoT is immutable. However, sensor networks offer only best-effort delivery and hence data might arrive at TritanDB out-of-order. The Quantum Re-ordering Buffer \(QRB\) mitigates out-of-order delivery within a period of time called a quantum. No matter how bad the connections are, as long as the quantum is longer than the worst case delivery, TritanDB has no problem processing the data.

![](/assets/qrb - Page 1.png)

For example, if a quantum, _q_, of 6 is defined, once 6 rows of the time-series are inserted into the QRB, a quantum expiry occurs and an insertion sort is performed on the content of the QRB, by timestamp. The insertion sort is efficient as the QRB is already partially-ordered (think slightly out-of-ordered). A portion of the QRB determined by a flush coefficient, _a_, of _q_ is flushed from the QRB. The remainder, (1-a)*q, is left in the QRB. The flush coefficient, _a_, is a value between 0 and 1. No value smaller than the new minimum value (from the remainder) in the QRB can be added. A new value of timestamp 1496337890 is inserted successfully as it is larger than the new minimum timestamp of 1496335840.

## Tritan Tables \(TrTables\)

Tritan Tables \(TrTables\) are a novel data structured for time-series storage inspired by Google's work on BigTable and Sorted String Tables \(SSTables\). Many big data systems like Apache Cassandra and HBase use SSTables organised as a Log Structured Merge Tree \(LSM-tree\) for key-value storage. A continuous compaction process helps manage these SSTables in terms of size and traversal performance. TrTables, however, do away with this background compaction process and LSM-tree design which adds unnecessary complication and can be expensive. Instead, TrTables, together with the QRB and a memtable in between, are designed as flat storage, which performs the fastest as it matches the physical design of modern storage. The interface to retrieve anything is physically one-dimensional, you provide an offset and you access a block of storage, hence, the flat TrTable, with a flat index, flat QRB and flat memtable works best and fastest.

## Time-series Compression

Time-series can often be compressed very effectively. The difference between timestamps, the delta, is much smaller than the actual timestamp. Values from sensors are unlikely to fluctuate greatly all the time. Hence, by compressing timestamps and values with reference to the previous sequence of values is advantageous because it is faster to compress and decompress (in terms of disk and memory input-output) a smaller chunk than to store and retrieve a bigger uncompressed chunk (and the pace and improvement of CPU speeds have outstripped that of memory and disk latency).

State-of-the-art timestamp and value compression is performed in TritanDB for time-partitioned blocks. Timestamps are compressed using Facebook's Gorilla delta-delta compression algorithm for high precision timestamps and Adaptive Delta-Run-Length-Encoded Rice Encoding for low precision timestamps. Values are encoded using the Gorilla value compression algorithm, which improves on the best-of-class FPC floating point compression algorithm, when divided into 64KB blocks (empirically proven in experiments). This also utilises the block boundaries on disk for fast read performance.

### Where precision matters - Timestamp Compression

TritanDB utilises the best algorithms for different precisions of timestamps. Timestamps with precision up to seconds are more efficiently encoded with our adaptive Delta-RLE-Rice algorithm while timestamps with higher precisions than seconds are encoded better with the Delta-of-Delta algorithm.

![](/assets/docs_tritandb_timeseries - Page 1.png)

## SPARQL Queries on S2SML Mappings

SPARQL is a powerful query language for RDF graphs which is similar in terms of syntax to SQL but allows the querying of subgraph patterns. Time-series data is stored in the most compact and concise means with TrTables while metadata and other graph structures and relations are stored in an in-memory graph model represented in RDF with S2SML.

