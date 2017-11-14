# SWIPE (SWappable Iterator for oPErations)

SWIPE represents an interface to modify the Operate process of TritanDB to extend the engine to support different languages, database and model tiers.

The implementation of the set of TritanDB operators is inspired by the work on an open and common relational algebra specification in Apache Calcite\footnote{https://calcite.apache.org/docs/algebra.html}. Table \ref{tab:operate_tritandb_operators} shows the conversion from a SPARQL algebra operator to the corresponding TritanDB operator. Each TritanDB operator is described as follows.

## Graph Patterns 

|SPARQL Algebra | TritanDB Operator |
|---	|---	|
| BGP, $$G$$ | _match_(BGP,map), _scan_(TS) |
| Join, $$\bowtie$$ | join(expr...) |
| LeftJoin, $$\ltimes$$ | semiJoin(expr) |
| Filter, $$\sigma$$ | filter(expr...) |
| Union, $\cup$	| union() |
|Graph	| setMap(map) |
|Extend, $$\rho$$	| extend(expr,var) |
|Minus	| minus() |
|Group/Aggregation, $$\gamma$$	| aggregate(groupKey, aggr) |

## Solution Modifiers


\begin{table}%
\begin{minipage}{\columnwidth}
\begin{center}
\begin{tabular}{l|r}
  \toprule
SPARQL Algebra & TritanDB Operator  \\
  	\midrule
\multicolumn{2}{c}{Graph Pattern}\\
  	\midrule
BGP, $G$				& match(BGP,map), scan(TS) \\
Join, $\bowtie$			& join(expr...) \\
LeftJoin, $\ltimes$		& semiJoin(expr) \\
Filter, $\sigma$			& filter(expr...) \\
Union, $\cup$			& union() \\
Graph					& setMap(map) \\
Extend, $\rho$			& extend(expr,var) \\
Minus					& minus() \\
Group/Aggregation, $\gamma$		& aggregate(groupKey, aggr) \\
  	\midrule
\multicolumn{2}{c}{Solution Modifiers}\\
\midrule
OrderBy					& sort(fieldOrdinal...) \\
Project, $\Pi$			& project(exprList [, fieldNames]) \\
Distinct, $\sigma_{D}$	& distinct() \\
Reduced	$\sigma_{R}$	& distinct() \\
Slice, $\sigma_{S}$		& limit(offset, fetch) \\
 \bottomrule  
\end{tabular}
\end{center}
\end{minipage}
\caption{SPARQL Algebra to TritanDB Operator Translation}
\label{tab:operate_tritandb_operators}
\end{table}%

\texttt{match} is described in Definition \ref{def:match} which matches a Basic Graph Pattern (BGP) from a query with a mapping to produce a binding $\mathbb{B}$. A set of time-series are referenced within $\mathbb{B}$. \texttt{scan} is an operator that returns an iterator over a time-series \texttt{TS}.

\texttt{join}, combines two time-series according to conditions specified as \texttt{expr} while \texttt{semiJoin} joins two time-series according to some condition, but outputs only columns from the left input.

\texttt{filter} modifies the input to return an iterator over points for which the conditions specified in \texttt{expr} evaluates to true. A common filter condition would a specification of a range of time for a time-series.

\texttt{union} returns the union of the input time-series and bindings $\mathbb{B}$. If the same time-series is referenced within inputs, only the bindings need to be merged. If two different time-series are merged, the iterator is formed in linear time by a comparison-based sorting algorithm, for example, the merge step within a merge sort, as the time-series are retrieved in sorted time order.

\texttt{setMap} is used to apply the specified mapping to its algebra tree leaf nodes for \texttt{match}.

\texttt{extend} allows the evaluation of an expression \texttt{expr} to be bound to a new variable \texttt{var}. This evaluation is performed only if \texttt{var} is projected. There are three means in SPARQL to produce the algebra: using \texttt{bind}, expressions in the \texttt{select} clause or expressions in the \texttt{group by} clause.

\texttt{minus} returns the iterator of first input excluding points from the second input.

\texttt{aggregate} produces an iteration over a set of aggregated results from an input. To calculate aggregate values for an input, the input is first divided into one or more groups by the \texttt{groupKey} field and the aggregate value is calculated for the particular \texttt{aggr} function for each group. The \texttt{aggr} functions supported are \texttt{count}, \texttt{sum}, \texttt{avg}, \texttt{min}, \texttt{max}, \texttt{sample} and \texttt{groupconcat}.

\texttt{sort} imposes a particular sort order on its input based on a sequence consisting of \texttt{fieldOrdinals}, each defining the time-series field index (zero-based) and specifying a positive ordinal for ascending and negative for descending order.

\texttt{project} computes the set of chosen variables to `select' from its input, as specified by \texttt{exprList}, and returns an iterator to the result containing only the selected variables. The default name of variables provided can be renamed by specifying the new name within the \texttt{fieldNames} argument.

\texttt{distinct} eliminates all duplicate records while \texttt{reduced}, in the TritanDB implementation, performs the same function. The SPARQL specification defines the difference being that \texttt{distinct} ensures duplicate elimination while \texttt{reduced} simply permits duplicate elimination. Given that time-series are retrieved in sorted order of time, the \texttt{distinct} function works the same for \texttt{reduced} as well and eliminates immediately repeated duplicate result rows.

\texttt{limit} computes a window over the input returning an iterator over results that are of a maximum size (in rows) of \texttt{fetch} and are a distance of \texttt{offset} from the start of the result set.

