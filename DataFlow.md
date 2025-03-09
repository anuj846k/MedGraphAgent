# Healthcare Dataset to ArangoDB Graph Flow

## 1. Dataset Selection and Preparation
The Synthea dataset was chosen because it provides realistic synthetic healthcare data that mimics real-world patient records without privacy concerns:

- **Data Source**: Pre-generated Synthea dataset hosted on Amazon S3
- **Format**: JSON files for both vertex (entity) and edge (relationship) collections
- **Structure**: 15 vertex collections and 26 edge collections representing the complete healthcare ecosystem
- **Dataset Structure**: 145515 nodes and 311702 edges


## 2. ArangoDB Connection Setup
```python
from arango import ArangoClient

# ArangoDB Connection
client = ArangoClient(hosts='https://81e2abc62b00.arangodb.cloud:8529')
db = client.db('synthea_db', username='root', password='your_password')
```
This establishes a connection to your ArangoDB cloud instance, which provides the graph database capabilities needed for complex healthcare data relationships.

## 3. Data Source Mapping
```python
vertex_urls = {
    'patients': 'https://arangodb-dataset-library-ml.s3.amazonaws.com/synthea_p100/vertex_collection/patients_3495d5d84b3bedeb4377cfc005bfbe52.data.json',
    'conditions': 'https://arangodb-dataset-library-ml.s3.amazonaws.com/synthea_p100/vertex_collection/conditions_b5b399ec1fcfe753f58dbafa197efdc1.data.json',
    # ... additional vertex collections
}

edge_urls = {
    'patients_to_conditions': 'https://arangodb-dataset-library-ml.s3.amazonaws.com/synthea_p100/edge_collection/patients_to_conditions_a9d75e31a814f611b70ca1f88ef20c17.data.json',
    'patients_to_medications': 'https://arangodb-dataset-library-ml.s3.amazonaws.com/synthea_p100/edge_collection/patients_to_medications_a5a78867f3dbe02223fb17dc428bdebc.data.json',
    # ... additional edge collections
}
```
This mapping provides a clear structure of all healthcare entities and their relationships, forming the foundation of your graph model.

## 4. Data Loading Process
```python
def download_and_load_collection(collection_name, url, is_edge=False):
    try:
        # Download data from S3
        response = requests.get(url)
       
        if response.status_code != 200:
            print(f"Failed to download {collection_name} from {url}")
            return False
       
        # Create collection with appropriate type (edge or vertex)
        if is_edge:
            if not db.has_collection(collection_name):
                db.create_collection(collection_name, edge=True)
        else:
            if not db.has_collection(collection_name):
                db.create_collection(collection_name)
       
        collection = db.collection(collection_name)
       
        # Parse and insert documents
        documents = [json.loads(line) for line in response.text.split('\n') if line.strip()]
       
        # Bulk insert for efficiency
        if documents:
            collection.insert_many(documents)
            print(f"Loaded {len(documents)} documents into {collection_name}")
       
        return True
   
    except Exception as e:
        print(f"Error loading {collection_name}: {str(e)}")
        return False
```
This function handles:
- Downloading data from S3
- Creating appropriate collections in ArangoDB
- Parsing JSON documents
- Bulk inserting data for efficiency
- Error handling and reporting

## 5. Orchestrated Loading Process
```python
def load_vertices():
    print("Loading Vertex Collections...")
    for vertex, url in vertex_urls.items():
        download_and_load_collection(vertex, url, is_edge=False)

def load_edges():
    print("Loading Edge Collections...")
    for edge, url in edge_urls.items():
        download_and_load_collection(edge, url, is_edge=True)

def load_synthea_dataset():
    load_vertices()
    load_edges()
    print("Synthea dataset loading complete!")
```
This ensures that all vertex collections are loaded before edge collections, which is critical since edges need to reference existing vertices.

## 6. Graph Model Creation
```python
# Graph name
graph_name = 'complete_synthea_healthcare_graph'

# Edge definitions connecting vertices
edge_definitions = [
    {
        'edge_collection': 'patients_to_conditions',
        'from_vertex_collections': ['patients'],
        'to_vertex_collections': ['conditions']
    },
    # ... 25 more edge definitions
]

# Create the graph
try:
    # Drop existing graph if it exists
    if db.has_graph(graph_name):
        db.delete_graph(graph_name)
   
    # Create new graph
    db.create_graph(graph_name, edge_definitions)
    print(f"Graph {graph_name} created successfully.")
   
    # Verify graph creation
    graph = db.graph(graph_name)
    print("\nGraph Details:")
    print("Edge Collections:", graph.edge_collections())
    print("Vertex Collections:", graph.vertex_collections())

except Exception as e:
    print(f"Error creating graph: {e}")
```
This creates a formal graph definition in ArangoDB that:
- Defines all relationships between healthcare entities
- Enables graph traversals and algorithms
- Provides a unified view of the healthcare data ecosystem

![imag1](https://i.imgur.com/yxFODCF.png)
![imag1](https://i.imgur.com/iub5e4k.png)
![imag1](https://i.imgur.com/iGT1DPd.png)
![imag1](https://i.imgur.com/MQnbR1A.png)


## 7. Graph Verification
```python
def verify_graph_connections():
    print("\nVerifying Graph Connections:")
    try:
        # Sample query to check traversal
        sample_query = '''
        FOR v, e, p IN 1..2 OUTBOUND 'patients/1'
        GRAPH 'complete_synthea_healthcare_graph'
        RETURN {
            vertex: v,
            edge: e,
            path: p
        }
        '''
        cursor = db.aql.execute(sample_query)
        print("Sample graph traversal successful.")
        print("Sample paths found:", len(list(cursor)))
    except Exception as e:
        print("Error verifying graph connections:", e)
```
This ensures that:
- The graph is traversable
- Relationships are correctly established
- The data can be queried using graph operations

## 8. Integration with Agent Architecture
Once the graph is created and verified, it is integrated with the agent architecture:
1. **Graph Database Access**: The agent tools connect to this graph database
2. **AQL Queries**: The `text_to_aql_to_text` tool generates and executes AQL queries against this graph
3. **Graph Algorithms**: The `text_to_nx_algorithm_to_text` tool uses NetworkX to run algorithms on subgraphs
4. **Hybrid Analysis**: The `hybrid_healthcare_analysis` tool combines both approaches

## 9. Data Model Benefits for Healthcare Analytics
This graph model provides significant advantages for healthcare analytics:
1. **Relationship-First Approach**: Healthcare data is inherently connected (patients to conditions to treatments)
2. **Temporal Analysis**: The graph preserves time relationships for progression analysis
3. **Pattern Detection**: Graph algorithms can identify patterns invisible in traditional databases
4. **Flexible Querying**: Both structured (AQL) and algorithmic (NetworkX) approaches are supported
5. **Holistic Patient View**: All patient data is connected in a single unified graph

**THANK YOU**

**Team: The Nucleus**
