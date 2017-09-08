# How it works

> ### IoT Time-series workloads are different.

_**Time-series workloads produced by streams from the Internet of Things \(IoT\) are different.**_ Time-series data is largely immutable. Writes primarily occur as new appends to recent time intervals, not as updates to existing rows. Both read and write workloads have a natural partitioning across both time and space. Hence, TritanDB utilises these properties to most efficiently store and process such IoT workloads.

> #### Of the total potential economic value the IoT enables, interoperability is required for 40 percent on average and for nearly 60 percent in some settings. - [Unlocking the potential of the IoT, McKinsey](http://www.mckinsey.com/business-functions/digital-mckinsey/our-insights/the-internet-of-things-the-value-of-digitizing-the-physical-world)

**Semantic interoperability is essential for achieving the economic and social impact of the IoT. **A rich model for representing metadata which contains the context of IoT time-series sensor data is required. Graph-based models are what almost every standards body and industry authority are working towards for representing metadata. TritanDB is built to support such rich models with the [Resource Description Framework \(RDF\)](https://www.w3.org/RDF/), state-of-the-art technology that drives the semantic web and machine-to-machine interoperability on the web.

## The Nature of Time-series IoT Data

The majority IoT time-series data is flat, wide, numerical and either approximately periodic or non-periodic.

## Quantum Re-ordering Buffer \(QRB\)

## Tritan Tables \(TrTables\)

## Time-series Compression

## SPARQL Queries on S2SML Mappings



