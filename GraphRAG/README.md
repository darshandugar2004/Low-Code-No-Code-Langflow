### Project Title: GraphRAG
**Hybrid Retrieval with Knowledge Graphs and Vector Stores**

### Image
<img src="https://github.com/darshandugar2004/Low-Code-No-Code-Langflow/blob/main/images/GraphRAG.png">

### Project Description

This project is an advanced, enterprise-ready **Hybrid Retrieval-Augmented Generation (RAG)** workflow for Langflow. It is designed to overcome the limitations of standard vector-search RAG by integrating a **Knowledge Graph (Neo4j)** for structured, factual data retrieval alongside a **Vector Store (Chroma)** for unstructured, semantic context.

This dual-retrieval approach allows the AI to answer complex questions with greater accuracy and context. It can understand not only *what* is being discussed (from vector search) but also *how* entities are specifically related (from the graph).

### Core Concept: The Hybrid RAG Pipeline

When a user asks a question, the workflow initiates a two-pronged retrieval process before generating an answer:

1.  **Vector Retrieval (Unstructured Context):** The user's question is first used to query the `Chroma DB Search` component. This fetches semantically similar document chunks (the "unstructured context") that provide general information.
2.  **Entity-Based Graph Retrieval (Structured Context):**
    * The text retrieved from Chroma is immediately passed to the `NER Extractor` node, which identifies and extracts key entities (e.g., people, organizations, locations).
    * These entities are then fed into the `Neo4j Query` node. This component queries your Neo4j database to find specific, factual relationships connected to those entities.
3.  **Hybrid Augmentation:** The final `Prompt Template` is intelligently "stuffed" with *both* sets of information: the general unstructured context from Chroma and the specific, factual relationships from Neo4j.
4.  **Generation:** This combined, context-rich prompt is sent to the `Language Model` (Gemini), which generates a final, comprehensive answer. The prompt explicitly instructs the model to prioritize the structured graph knowledge, ensuring answers are more factual.

### Key Components & Workflows

Your project is broken into two main parts: the **Data Ingestion** flows (one-time setup) and the **Query** flow (the live RAG pipeline).

#### 1. Data Ingestion (One-Time Setup)

Before the RAG pipeline can work, both databases must be populated.

* **Vector Store Ingestion (Chroma):**
    * A `File` component loads your source documents (e.g., `.txt`, `.pdf`).
    * `SplitText` breaks the documents into manageable chunks.
    * `Google Generative AI Embeddings` converts these chunks into vector embeddings.
    * `Chroma Ingest` saves these embeddings into a persistent Chroma database.
* **Knowledge Graph Ingestion (Neo4j):**
    * This flow is designed to load *pre-existing* graph data.
    * Two `File` components are used to load `entities.json` and `relationships.json`.
    * The `Neo4j JSON Importer` (a custom component) parses these files and populates your Neo4j database with the nodes and their relationships.
    * *(Note: The flow also includes a separate `GraphTripleExtractor` path, which provides an alternative LLM-based method for creating these JSON files from raw text.)*

#### 2. Query Workflow (The Live RAG Pipeline)

This is the main interaction flow described in the "Core Concept" section.

* `ChatInput`: User asks a question (e.g., "What is the relationship between Company X and Company Y?").
* `Chroma DB Search`: Retrieves documents mentioning "Company X" and "Company Y".
* `NERExtractorNode`: Extracts "Company X" and "Company Y" as entities from the retrieved text.
* `Neo4jQueryNode`: Queries Neo4j (e.g., `MATCH (a {name: 'Company X'})-[r]-(b {name: 'Company Y'}) RETURN r`).
* `Prompt Template`: Assembles the final prompt:
    * **Unstructured Context:** "Company X is a tech giant. Company Y is a startup."
    * **Structured Context:** "Company X *INVESTED_IN* Company Y."
    * **Question:** "What is the relationship between Company X and Company Y?"
* `LanguageModelComponent`: Generates the answer: "Company X has an investment relationship with Company Y; they are an investor in the startup."
* `ChatOutput`: Displays the final, fact-checked answer.

### Target Use Case

This template is ideal for **enterprises** and industries dealing with highly interconnected, factual data where relationships are critical. This includes:

* **Internal Knowledge Management:** Answering complex questions about corporate structure, projects, and personnel (e.g., "Who managed Project 'Titan' and which teams did they collaborate with?").
* **Financial Analysis:** Understanding relationships between companies, directors, and market events.
* **Medical/Scientific Research:** Finding pathways, drug interactions, and relationships between genes and proteins.
* **Secure, On-Premise AI:** By using local models and on-premise Neo4j/Chroma instances, this entire workflow can be run securely without sensitive data leaving the corporate network.
