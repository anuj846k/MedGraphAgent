# **MedGraph: AI-Powered Healthcare Insights Navigator**

## **Overview**
MedGraph is an intelligent healthcare analytics system leveraging **GraphRAG**, **ArangoDB**, and **GPU-accelerated graph processing** to uncover deep insights from **synthetic patient data (Synthea)**. Designed for **healthcare providers, researchers, and administrators**, it facilitates:

✅ **Patient Pathway Analysis** – Understand patient journeys through diagnoses, treatments, and outcomes.  
✅ **Treatment Effectiveness Comparison** – Identify which treatments work best for similar patient cohorts.  
✅ **Healthcare Resource Optimization** – Detect inefficiencies in hospital operations.  
✅ **Medical Anomaly & Risk Detection** – Identify rare disease patterns, misdiagnoses, or adverse reactions.  
✅ **Personalized Care Recommendations** – Suggest data-driven care plans based on patient histories.  

---

## **How It Works (Data Flow)**  

1️⃣ **Dataset Loading** – The synthetic healthcare dataset from **Synthea** is parsed and loaded into **ArangoDB**.  
2️⃣ **Graph Model Creation** – Patient data is transformed into an interconnected **graph structure**.  
3️⃣ **Hybrid Query Execution** –  
   - **Simple Queries → AQL (ArangoDB Query Language)**  
   - **Complex Queries → GPU-accelerated Graph Algorithms (cuGraph/NetworkX)**  
4️⃣ **Insights Generation** – MedGraph analyzes relationships and patterns in patient data.  
5️⃣ **User Interface** – Results are visualized in **Gradio-powered dashboards**.  

---

## **🔗 Technologies Used**
- **ArangoDB** – Graph database for structured medical data.  
- **LangChain & GraphRAG** – Converts natural language queries into graph-based reasoning.  
- **NetworkX / cuGraph** – Performs complex graph analytics.  
- **Gradio** – Provides an intuitive web-based user interface.  

---

## **🛠 Implementation Breakdown**

### **1️⃣ Data Ingestion**
**Synthea** generates patient data in JSON format, including:  
- **Patients, Encounters, Conditions, Medications, Procedures, Immunizations.**  

#### **Loading Data into ArangoDB (Jupyter Notebook Cell)**
```python
from arango import ArangoClient

# Connect to ArangoDB
client = ArangoClient(hosts="http://localhost:8529")
db = client.db("medgraph_db", username="root", password="password")

# Load Patients Collection
def load_collection(collection_name, data):
    if not db.has_collection(collection_name):
        db.create_collection(collection_name)
    collection = db.collection(collection_name)
    collection.insert_many(data)

# Execute in Jupyter Notebook
load_collection("patients", patient_data)
```
✅ **Data is now structured in ArangoDB with vertex and edge relationships.**

---

### **2️⃣ Graph Construction**
The system builds an interconnected **graph model** linking medical entities:

#### **Creating Graph in ArangoDB (Jupyter Notebook Cell)**
```python
graph_name = "healthcare_graph"

edge_definitions = [
    {'edge_collection': 'patients_to_conditions', 'from_vertex_collections': ['patients'], 'to_vertex_collections': ['conditions']},
    {'edge_collection': 'patients_to_medications', 'from_vertex_collections': ['patients'], 'to_vertex_collections': ['medications']},
]

# Create Graph
if db.has_graph(graph_name):
    db.delete_graph(graph_name)
db.create_graph(graph_name, edge_definitions)
```
✅ **Now, patient records are interconnected for graph traversal and analytics.**

---

### **3️⃣ Query Execution & AI Analysis**
MedGraph supports **two types of queries**:

#### **🔹 AQL for Simple Queries (Jupyter Notebook Cell)**
Example: Find all patients with diabetes.
```python
aql_query = """
FOR patient IN patients
    FILTER patient.condition == 'Diabetes'
    RETURN patient
"""
cursor = db.aql.execute(aql_query)
for record in cursor:
    print(record)
```

#### **🔹 Graph Algorithms for Complex Queries (Jupyter Notebook Cell)**
Example: Identifying high-risk patients using **PageRank** in **NetworkX**.
```python
import networkx as nx

G = nx.Graph()
for condition in conditions:
    G.add_node(condition["patient_id"], risk_factor=condition["risk_score"])

# Compute risk-based PageRank
pagerank_scores = nx.pagerank(G, alpha=0.85)
pagerank_scores
```
✅ **Allows deeper insights into patient risk patterns.**

---

### **4️⃣ Visualization & User Interface**
To make insights accessible, MedGraph integrates **Gradio** for visual dashboards.

#### **Example: Interactive Patient Insights Dashboard (Jupyter Notebook Cell)**
```python
import gradio as gr

def search_patient(patient_id):
    result = db.collection("patients").get(patient_id)
    return result if result else "Patient Not Found"

# Launch in Jupyter Notebook
gr.Interface(fn=search_patient, inputs="text", outputs="json").launch()
```
✅ **Provides an interactive UI for easy query execution.**

---

## **📌 Setup Instructions**
### **1️⃣ Install Dependencies**
```bash
pip install arango gradio networkx langchain jupyter
```

### **2️⃣ Run MedGraph in Jupyter Notebook**
- Open Jupyter Notebook: `jupyter notebook`
- Load the notebook containing MedGraph code.
- Execute the cells sequentially to process data, build the graph, and interact with insights.

### **3️⃣ Access Web UI in Jupyter Notebook**
- After running the Gradio cell, access the UI at **`http://localhost:7860`**

---

## **📢 Final Thoughts**
MedGraph **transforms raw patient data into actionable healthcare intelligence**. By leveraging **graph-based AI and real-time analytics**, it aims to **redefine decision-making in healthcare**.

🔹 **Built with ArangoDB, NetworkX, LangChain, and Gradio**  
🔹 **Designed for Medical AI, Research, and Predictive Analytics**  

---

