# SWIBRE (SWappable Interface for BGP Resolution Engines)

SWIBRE represents an interface to plugin any existing RDF store and SPARQL engine or a custom one to do BGP resolution for TritanDB. There are currently reference implementations for:

- Jena
- Sesame

The interface to implement is SwibreMatcher:

```java
package com.tritandb.engine;

public interface SwibreMatcher {	
	public void loadMapping(String filename);
	public ResultSet executeQuery(String queryStr, String dialect);
	public Boolean hasResults();
}
```