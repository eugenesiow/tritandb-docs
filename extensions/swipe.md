# SWIPE (SWappable Iterator for oPErations)

SWIPE represents an interface to modify the Operate process of TritanDB to extend the engine to support different languages, database and model tiers.

The implementation of the set of TritanDB operators is inspired by the work on an open and common relational algebra specification in [Apache Calcite](https://calcite.apache.org/docs/algebra.html). The tables show the conversion from a SPARQL algebra operator to the corresponding TritanDB operator. Each TritanDB operator is described as follows.

## Graph Patterns 

|SPARQL Algebra | TritanDB Operator |
|---	|---	|
| BGP, $$G$$ | match(BGP,map), scan_(TS) |
| Join, $$\bowtie$$ | join(expr...) |
| LeftJoin, $$\ltimes$$ | semiJoin(expr) |
| Filter, $$\sigma$$ | filter(expr...) |
| Union, $$\cup$$	| union() |
|Graph	| setMap(map) |
|Extend, $$\rho$$	| extend(expr,var) |
|Minus	| minus() |
|Group/Aggregation, $$\gamma$$	| aggregate(groupKey, aggr) |

_match_ which matches a Basic Graph Pattern (BGP) from a query with a mapping to produce a binding $$\mathbb{B}$$. A set of time-series are referenced within $$\mathbb{B}$$. _scan_ is an operator that returns an iterator over a time-series _TS_.

_join_, combines two time-series according to conditions specified as _expr_ while _semiJoin_ joins two time-series according to some condition, but outputs only columns from the left input.

_filter_ modifies the input to return an iterator over points for which the conditions specified in _expr_ evaluates to true. A common filter condition would a specification of a range of time for a time-series.

_union_ returns the union of the input time-series and bindings $$\mathbb{B}$$. If the same time-series is referenced within inputs, only the bindings need to be merged. If two different time-series are merged, the iterator is formed in linear time by a comparison-based sorting algorithm, for example, the merge step within a merge sort, as the time-series are retrieved in sorted time order.

_setMap_ is used to apply the specified mapping to its algebra tree leaf nodes for _match_.

_extend_ allows the evaluation of an expression _expr_ to be bound to a new variable _var_. This evaluation is performed only if _var_ is projected. There are three means in SPARQL to produce the algebra: using _bind_, expressions in the _select_ clause or expressions in the _group by_ clause.

_minus_ returns the iterator of first input excluding points from the second input.

_aggregate_ produces an iteration over a set of aggregated results from an input. To calculate aggregate values for an input, the input is first divided into one or more groups by the _groupKey_ field and the aggregate value is calculated for the particular _aggr_ function for each group. The _aggr_ functions supported are _count_, _sum_, _avg_, _min_, _max_, _sample_ and _groupconcat_.

## Solution Modifiers

|SPARQL Algebra | TritanDB Operator |
|---	|---	|
|OrderBy	| sort(fieldOrdinal...) |
|Project, $$\Pi$$	| project(exprList [, fieldNames]) |
|Distinct, $$\sigma_{D}$$	| distinct() |
|Reduced	$$\sigma_{R}$$	| distinct() |
|Slice, $$\sigma_{S}$$	| limit(offset, fetch) |

_sort_ imposes a particular sort order on its input based on a sequence consisting of _fieldOrdinals_, each defining the time-series field index (zero-based) and specifying a positive ordinal for ascending and negative for descending order.

_project_ computes the set of chosen variables to 'select' from its input, as specified by _exprList_, and returns an iterator to the result containing only the selected variables. The default name of variables provided can be renamed by specifying the new name within the _fieldNames_ argument.

_distinct_ eliminates all duplicate records while _reduced_, in the TritanDB implementation, performs the same function. The SPARQL specification defines the difference being that _distinct_ ensures duplicate elimination while _reduced_ simply permits duplicate elimination. Given that time-series are retrieved in sorted order of time, the _distinct_ function works the same for _reduced_ as well and eliminates immediately repeated duplicate result rows.

_limit_ computes a window over the input returning an iterator over results that are of a maximum size (in rows) of _fetch_ and are a distance of _offset_ from the start of the result set.

