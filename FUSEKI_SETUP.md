# Apache Jena: Fuseki
This file provides instructions for recreating the docker environment that we will use to
host our microservice related to SPARQL endpoints. To learn more about the software we 
will rely on, check out the [documentation](https://jena.apache.org/documentation/serving_data/).

## Steps
(1) Grab the image:
```
docker pull stain/jena-fuseki
```

(2) Start the server
```
docker run -d -p 8000:3030 stain/jena-fuseki
docker run -p 3030:3030 -e JVM_ARGS=-Xmx2g stain/jena-fuseki
```
*Notice the environment variable being set (JVM_ARGS) this instructs the JVM of the amount
of memory to allocate for our application. For large (or complicated) OWL files (like SNOMED), 
increase this setting to see quicker upload / performance speeds.* Also, going this route requires
the user to authenticate as admin to upload datasets (owl files).

(3) Start the server and automatically load a data set:
```
docker run -d -p 3030:3030 \
-e ADMIN_PASSWORD=thisisonpurpose \
-v $(PWD)/lung_ontology.owl:/jena-fuseki/lung_ontology.owl \
stain/jena-fuseki \
/jena-fuseki/fuseki-server \
--file=/jena-fuseki/lung_ontology.owl /ontology
```
If the container stops for some reason, please use the restart command to ensure data is still loaded:
```
docker restart fuseki_lungmap
```

### Production Environment
```
docker run -d -p 3030:3030 \
-e JVM_ARGS=-Xmx1g \
--restart=always \
-e ADMIN_PASSWORD=thisisonpurpose \
-v $(pwd)/lung_ontology.owl:/jena-fuseki/lung_ontology.owl \
stain/jena-fuseki \
/jena-fuseki/fuseki-server \
--file=/jena-fuseki/lung_ontology.owl /ontology
```

## Example Query using python
```
from SPARQLWrapper import SPARQLWrapper, JSON
sparql = SPARQLWrapper("http://localhost:3030/ontology/query")
uri = "http://localhost:3030/Hotel/query"
query = """
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX : <http://www.semanticweb.org/am175/ontologies/2017/1/untitled-ontology-79#>
SELECT ?p ?p_label WHERE {
    ?p rdfs:subClassOf :Protein .
    ?p rdfs:label ?p_label
}
"""

sparql.setQuery(query)
sparql.setReturnFormat(JSON)
results = sparql.query().convert()
for result in results["results"]["bindings"]:
    print(result)

```