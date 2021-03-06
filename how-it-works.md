# How TritanDB Works

<!-- toc -->

## IoT Time-series workloads are different {#workloads}

_**Time-series workloads produced by streams from the Internet of Things \(IoT\) are different.**_ Time-series data is largely immutable. Writes primarily occur as new appends to recent time intervals, not as updates to existing rows. Both read and write workloads have a natural partitioning across both time and space. Hence, TritanDB utilises these properties to most efficiently store and process such IoT workloads.

> Of the total potential economic value the IoT enables, interoperability is required for 40 percent on average and for nearly 60 percent in some settings. - [Unlocking the potential of the IoT, McKinsey](http://www.mckinsey.com/business-functions/digital-mckinsey/our-insights/the-internet-of-things-the-value-of-digitizing-the-physical-world)

**Semantic interoperability is essential for achieving the economic and social impact of the IoT. **A rich model for representing metadata which contains the context of IoT time-series sensor data is required. Graph-based models are what almost every standards body and industry authority are working towards for representing metadata. TritanDB is built to support such rich models with the [Resource Description Framework \(RDF\)](https://www.w3.org/RDF/), state-of-the-art technology that drives the semantic web and machine-to-machine interoperability on the web.

## The Nature of Time-series IoT Data {#nature}

The majority IoT time-series data is flat, wide, numerical and either approximately periodic or non-periodic.

### Comparison of Flat and Complex Schema {#flat-complex}
A flat schema in JSON, as shown below, is a schema with only a single level of attribute-value pairs.
```json
{
    timestamp: 1501513833,
    temperature: 32.0,
    humidity: 10.5,
    light: 120.0,
    pressure: 28.0,
    wind_speed: 10.5,
    wind_direction: 0
}
```
A complex schema in JSON, as shown below, has multiple nested levels of data within a tree-like, hierarchical structure.
```json
{
    timestamp: 1501513833,
    temperature: {
       max: 22.0,
       current: 17.0,
       error : {
    	   percent: 5.0
       }
    }
}
```
### Width of Schema {#width}

The width of a schema refers to the number of attributes it contains besides the timestamp attribute. The flat JSON schema below has a width of 6 because it has a set containing 6 attributes besides the timestamp: (_temperature, humidity, light, pressure, wind_speed, wind_direction_).
```json
{
    timestamp: 1501513833,
    temperature: 32.0,
    humidity: 10.5,
    light: 120.0,
    pressure: 28.0,
    wind_speed: 10.5,
    wind_direction: 0
}
```
 A schema is considered wide if the width is 2 or more. This definition borrows from the _tall versus wide_ schema design philosophy in big data key-value stores for time-series such as in BigTable and [HBase](http://hbase.apache.org/book.html\#casestudies.schema).
 
### Data Types of Values within Schema {#value_type}

The value from an attribute-value pair within a schema can assume varying data types.There are 3 basic JSON data types _Number, String_ and _Boolean_. The _String_ attribute-value pairs can be of various categories: numerical strings, boolean strings, identifiers with a Universally Unique Identifier (UUID) format, categorical strings and test data. A study of public IoT data found that a majority of attributes besides the timestamp were numerical (87.2%). Numerical attributes include integers, floating point numbers and time values. 

### Periodicity of Streams {#period}

One of the fundamental characteristics of data streams that form a time-series is how frequently and evenly the observations are spaced in time. The definition of periodicity used is from the Oxford English Dictionary and refers to 'the quality of regular recurrence', in particular, the tendency for data observations in a stream to 'recur at regular intervals'. This should not be confused with the definition of periodicity as seasonality in some data mining literature. A period is the interval between subsequent observations in a time-series and a periodic stream is defined as one with exactly regular timestamp intervals or in other words, a constant period, while a non-periodic stream has unevenly-spaced intervals or a varying period. A periodic stream has maximum periodicity while a non-periodic stream could have high or low periodicity.

One of the differences between the Internet of Things and traditional wireless sensor networks is the potentially advantageous deployment of event-driven sensors within Things instead of sensors that record measurements at regular time intervals, determined by a constant sampling rate. For example, a smart light bulb that measures when a light is on can either send the signal only when the light changes state, i.e. is switched on or off, or send its state regularly every second. The former type of event-driven sensor gives rise to a non-periodic time series.

It was discovered that public IoT data was mainly approximately periodic or non-periodic (approximately in a 50-50 ratio).

## Quantum Re-ordering Buffer \(QRB\) {#qrb}

Time-series data for the IoT is immutable. However, sensor networks offer only best-effort delivery and hence data might arrive at TritanDB out-of-order. The Quantum Re-ordering Buffer \(QRB\) mitigates out-of-order delivery within a period of time called a quantum. No matter how bad the connections are, as long as the quantum is longer than the worst case delivery, TritanDB has no problem processing the data.

![](/assets/qrb - Page 1.png)

For example, if a quantum, _q_, of 6 is defined, once 6 rows of the time-series are inserted into the QRB, a quantum expiry occurs and an insertion sort is performed on the content of the QRB, by timestamp. The insertion sort is efficient as the QRB is already partially-ordered (think slightly out-of-ordered). A portion of the QRB determined by a flush coefficient, _a_, of _q_ is flushed from the QRB. The remainder, (1-a)*q, is left in the QRB. The flush coefficient, _a_, is a value between 0 and 1. No value smaller than the new minimum value (from the remainder) in the QRB can be added. A new value of timestamp 1496337890 is inserted successfully as it is larger than the new minimum timestamp of 1496335840.

## Tritan Tables \(TrTables\) {#trtables}

Tritan Tables \(TrTables\) are a novel data structured for time-series storage inspired by Google's work on BigTable and Sorted String Tables \(SSTables\). Many big data systems like Apache Cassandra and HBase use SSTables organised as a Log Structured Merge Tree \(LSM-tree\) for key-value storage. A continuous compaction process helps manage these SSTables in terms of size and traversal performance. TrTables, however, do away with this background compaction process and LSM-tree design which adds unnecessary complication and can be expensive. Instead, TrTables, together with the QRB and a memtable in between, are designed as flat storage, which performs the fastest as it matches the physical design of modern storage. The interface to retrieve anything is physically one-dimensional, you provide an offset and you access a block of storage, hence, the flat TrTable, with a flat index, flat QRB and flat memtable works best and fastest.

![](/assets/docs_tritandb_trtable - Page 1.png)

In the above figure, the memtable consists of an index entry, $$i$$, that stores values of the block timestamp, current TrTable offset and average, maximum, minimum and counts of the row data which are updated when elements from the QRB are inserted. It also stores a block entry, $$b$$, which contains the current compressed time-partitioned block data. The memtable gets flushed to a TrTable file once it reaches the time-partitioned block size, $$b_{size}$$. Each time-series has a memtable and corresponding TrTable file on-disk.

## Time-series Compression {#ts-compress}

Time-series can often be compressed very effectively. The difference between timestamps, the delta, is much smaller than the actual timestamp. Values from sensors are unlikely to fluctuate greatly all the time. Hence, by compressing timestamps and values with reference to the previous sequence of values is advantageous because it is faster to compress and decompress (in terms of disk and memory input-output) a smaller chunk than to store and retrieve a bigger uncompressed chunk (and the pace and improvement of CPU speeds have outstripped that of memory and disk latency).

State-of-the-art timestamp and value compression is performed in TritanDB for time-partitioned blocks. Timestamps are compressed using Facebook's Gorilla delta-delta compression algorithm for high precision timestamps and Adaptive Delta-Run-Length-Encoded Rice Encoding for low precision timestamps. Values are encoded using the Gorilla value compression algorithm, which improves on the best-of-class FPC floating point compression algorithm, when divided into 64KB blocks (empirically proven in experiments). This also utilises the block boundaries on disk for fast read performance.

### Where precision matters - Timestamp Compression {#ts-compress-precision}

TritanDB utilises the best algorithms for different precisions of timestamps. Timestamps with precision up to seconds are more efficiently encoded with our adaptive Delta-RLE-Rice algorithm while timestamps with higher precisions than seconds are encoded better with the Delta-of-Delta algorithm.

![](/assets/docs_tritandb_timeseries - Page 1.png)

#### Delta-of-Delta Timestamp Compression (High Precision) {#delta-delta}

Delta-of-Delta compression builds on the technique for compressing timestamps introduced by [Pelkonen et al. (2015)](#references) to support effective compression on varying timestamp precision. The header stores a full starting timestamp for the block in 64 bits and the next length of bits depends on the timespan of a block and the precision of the timestamps. In the example in the figure above, a 24 bit length is used to store the timestamp delta of '3602'. The 24 bit length is derived for a 4 hour block at millisecond precision with the first delta assumed to be positive. For the same 4 hour block, if the precision of timestamps during ingestion is determined to be in seconds a 14 bits length is calculated and used, while this length is 24 bits for millisecond precision and 44 bits for nanoseconds precision. 

$$\epsilon$$ is a 1 to 4 bit variable-length binary value that indicates the next number of bits to read for the first delta-of-delta value. '0' means the delta-of-delta ($$\Delta\Delta$$) is 0, while '10' means read the next 7 bits as the value is between '-63' and '64' (range of $$2^{7}$$), '110' the next 24 bits, '1110' the next 32 bits. Finally, an $$\epsilon$$ of '1111' means reading a 64 bit $$\Delta\Delta$$. The example follows with $$\Delta\Delta$$s of '-2' and '0' stored in just 10 bits which reflect the delta of the next two timestamps is '3600'. These lengths are adapted from the empirical measurement of the distribution of $$\Delta\Delta$$ values from the [Pelkonen et al. (2015)](#references) paper.

#### Delta-RLE-Rice Compression (Low Precision) {#delta-rice}

This proposal for Delta-RLE-Rice compression is inspired by the backward adaptation strategy from [Malvar (2006)](#references) for the run-length encoding method initially proposed by [Rice and Plaunt (1971)](#references). The backward adaptation strategy succeeds by tuning a $$k$$ parameter which allows the adaptive compression of timestamps and run-lengths of varying precision and periodicity respectively. Rice coding divides a value, $$u$$, into two parts based on $$k$$, giving a quotient $$q =\left \lfloor{u/2^{k}}\right \rfloor $$ and the remainder, $$r = u\%2^{k}$$. The quotient, $$q$$ is stored in unary coding, for example, the $$\Delta$$ value '3602' with a $$k$$ of 10 has a quotient of 3 and is stored as '1110'. The remainder, $$r$$, is binary coded in $$k$$ bits. Initial $$k$$ values of 2 and 10 are used in this example in the above figure and are adaptively tuned based on the previous value in the sequence so this can be reproduced during decoding. Three rules govern the tuning based on the value of $$q$$, allowing quick convergence on good $$k$$ values.

$$
   \text{if } q=
    \begin{cases}
      0, & k \to k-1 \\
      1, & \text{no change in } k \\
      >1, & k \to k+q
    \end{cases}
$$

This adaptive coding adjusts $$k$$ based on the actual data to be encoded so no other information needs to be retrieved on the side for decoding. It also has a fast learning rate that chooses good, though not necessarily optimal, $$k$$ values and does not have the delay of forward adaptation methods. $$k$$ is adapted from 2 and 10 to 1 and 13 respectively in the above figure.

## Value Compression {#value-compress}

Value compression, for speed, does not use predictors ([Burtscher and Ratanaworabhan's (2009)](#references) FPC) and instead compares the current value to only the previous value for speed, as detailed in the paper by [Pelkonen et al. (2015)](#references). After an XOR operation between the values, the result, $$r$$, is stored according to the output from a function _gor()_ described as follows, where '$$.$$' is an operator that appends bits together, $$p$$ is the previous XOR value, _lead()_ and _trail()_ return the number of leading and trailing zeroes respectively, _len()_ returns the length in bits and $$n$$ is the number of meaningful bits remaining within the value.

$$
    gor(r)=
    \begin{cases}
      '0', & \text{if } r = 0 \\
      '10'.n, & \text{if } lead(r) >= lead(p) \text{ and } trail(r) = trail(p)\\
      '11'.l.m.n, & \text{else, where } l = lead(r) \text{ and } m = len(n)
    \end{cases}
$$
  
The algorithm was designed for the fast encoding and decoding of blocks of time-series data as there is no condition within the _gor(r)_ function to reduce the number of significant bits stored in the sequence but only to increase them. However, each block resets the counter on significant bits and 'localises' the compression. 

## SPARQL Queries on S2SML Mappings {#s2sml}

SPARQL is a powerful query language for RDF graphs which is similar in terms of syntax to SQL but allows the querying of subgraph patterns. Time-series data is stored in the most compact and concise means with TrTables while metadata and other graph structures and relations are stored in an in-memory graph model represented in RDF with [S2SML](appendix/s2sml.md).

## References {#references}

Tuomas Pelkonen, Scott Franklin, Justin Teller, Paul Cavallaro, Qi Huang, Justin Meza, and Kaushik Veeraraghavan. [Gorilla: A Fast, Scalable, In-Memory Time Series Database](http://www.vldb.org/pvldb/vol8/p1816-teller.pdf). _In Proceedings of the 41st International Conference on Very Large Data Bases, 2015_

Henrique S Malvar. [Adaptive Run-Length / Golomb-Rice Encoding of Quantized Generalized Gaussian Sources with Unknown Statistics](http://ieeexplore.ieee.org/document/1607237/). _In Proceedings of the Data Com- pression Conference, 2006_

Robert Rice and James Plaunt. [Adaptive Variable-Length Coding for Efficient Compression of Spacecraft Television Data](http://ieeexplore.ieee.org/document/1090789/). _IEEE Transactions on Communications, 19 (6):889897, dec 1971._

Martin Burtscher and Paruj Ratanaworabhan. [FPC: A High-Speed Compressor for Double-Precision Floating-Point Data](http://ieeexplore.ieee.org/document/4589203/). _IEEE Transactions on Computers, 58(1):18 31, jan 2009._
