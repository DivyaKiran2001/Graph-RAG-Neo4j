# GraphRAG: Three Core Retrieval Approaches with Neo4j

## Overview

GraphRAG combines a Large Language Model (LLM) with retrieval from a
Neo4j knowledge graph. Instead of relying only on the LLM's internal
knowledge, the application retrieves relevant information from Neo4j and
provides that information as context before generating the final answer.

This README focuses on three important approaches:

1.  **Vector Retrieval**
2.  **Text-to-Cypher**
3.  **Vector + Cypher Graph Retrieval**

------------------------------------------------------------------------

# 1. Vector Retrieval

## What is Vector Retrieval?

Vector retrieval finds information based on **semantic similarity**.

The data is converted into embeddings and stored in a Neo4j vector
index. When a user asks a question:

``` text
User Question
      ↓
Create Query Embedding
      ↓
Neo4j Vector Index
      ↓
Find Semantically Similar Content
      ↓
Retrieved Context
      ↓
LLM
      ↓
Answer
```

The important point is that the system searches for **meaning**, rather
than requiring an exact keyword match.

### Example

Stored information:

``` text
"OpenAI released GPT-4 for advanced language understanding."
```

User question:

``` text
"What model did OpenAI release for advanced language understanding?"
```

Vector similarity can identify the relevant content even when the
wording differs.

## When to use Vector Retrieval

Use it when the question is mainly about:

-   Semantic similarity
-   Unstructured text
-   Documents
-   Descriptions
-   Explanations
-   General contextual information
-   Finding relevant passages with different wording

### Example questions

``` text
"What is this technology used for?"

"Explain the main purpose of this document."

"What are the important details mentioned in this report?"
```

## Advantages

-   Good for natural-language questions
-   Handles different wording and synonyms
-   Useful for document-based RAG
-   Does not require the LLM to generate Cypher

## Limitation

Pure vector retrieval does not naturally handle complex graph
relationships.

For example:

``` text
"Which accused person is connected to FIR-101
through a victim who lives in Vijayawada?"
```

This requires following relationships in the graph.

------------------------------------------------------------------------

# 2. Text-to-Cypher

## What is Text-to-Cypher?

Text-to-Cypher converts a user's natural-language question into a
**Cypher query**.

The flow is:

``` text
User Question
      ↓
LLM
      ↓
Generate Cypher
      ↓
Execute Cypher in Neo4j
      ↓
Graph Results
      ↓
LLM
      ↓
Final Answer
```

### Example

User asks:

``` text
"Find all accused persons involved in FIR-101."
```

The LLM may generate:

``` cypher
MATCH (a:Accused)-[:INVOLVED_IN]->(f:FIR {id: "FIR-101"})
RETURN a
```

Neo4j executes the query and returns the graph records. The LLM then
uses those records to generate the final answer.

## When to use Text-to-Cypher

Use it when the question is mainly about:

-   Exact entities
-   Relationships
-   Filtering
-   Aggregation
-   Counts
-   Structured properties
-   Multi-hop graph queries
-   Graph-specific business rules

### Example questions

``` text
"How many FIRs are associated with this accused?"

"Which victims are connected to FIR-101?"

"Which accused persons are connected to the same location?"

"Find all FIRs registered in a particular police station."
```

## Advantages

-   Can answer structured graph questions
-   Can traverse multiple relationships
-   Supports filtering and aggregation
-   Flexible for different graph questions

## Limitations

The LLM must generate valid and appropriate Cypher.

Possible problems include:

-   Incorrect labels
-   Incorrect relationship names
-   Incorrect property names
-   Invalid Cypher
-   Hallucinated entities or properties
-   Inefficient queries

Therefore, production implementations should provide schema information
and apply appropriate validation, access controls, and query safeguards.

------------------------------------------------------------------------

# 3. Vector + Cypher Graph Retrieval

## What is Vector + Cypher Graph Retrieval?

This approach combines:

1.  **Vector similarity** to find relevant starting information
2.  **Cypher / graph traversal** to retrieve related graph information

The flow is:

``` text
User Question
      ↓
Create Query Embedding
      ↓
Vector Search
      ↓
Find Relevant Node(s)
      ↓
Cypher / Graph Traversal
      ↓
Retrieve Connected Entities
      ↓
Combined Graph Context
      ↓
LLM
      ↓
Final Answer
```

The key idea is:

> Use semantic search to find the relevant part of the graph, then use
> the graph structure to retrieve the relationships around it.

### Example

Suppose the graph contains:

``` text
FIR
 ├── HAS_VICTIM ──> Victim
 ├── INVOLVES ────> Accused
 └── OCCURRED_AT ─> Location
```

The user asks:

``` text
"Find information about the incident involving the person
described in this report and tell me who else was connected
to that incident."
```

Vector search can identify the relevant incident or starting node.

Then graph traversal can retrieve:

``` text
Relevant FIR
   ├── Victim
   ├── Accused
   └── Location
```

The retrieved graph context is then given to the LLM.

## When to use Vector + Cypher

Use this approach when the question requires both:

-   Semantic understanding
-   Graph relationships

It is particularly useful when the user does not provide an exact
identifier, but the application still needs to identify the correct
entity and explore its relationships.

### Example

``` text
"Find the incident related to this description
and tell me everyone and every location connected to it."
```

## Advantages

-   Combines semantic retrieval and graph relationships
-   Useful for multi-hop questions
-   Reduces dependence on generating an entire graph query from scratch
-   Provides relationship-aware context
-   Useful for Knowledge Graph + RAG applications

## Limitations

-   More complex than simple vector retrieval
-   Requires vector indexes and graph modeling
-   Retrieval quality depends on semantic search and graph traversal
-   The graph model must accurately represent required relationships

------------------------------------------------------------------------

# Comparison of the Three Approaches

  -------------------------------------------------------------------------
  Approach          Main Retrieval    Best For            Graph
                    Method                                Relationships
  ----------------- ----------------- ------------------- -----------------
  Vector Retrieval  Embedding         Semantic/document   Limited
                    similarity        questions           

  Text-to-Cypher    LLM-generated     Structured graph    Strong
                    Cypher            questions           

  Vector + Cypher   Vector search +   Semantic +          Strong
                    graph traversal   relationship        
                                      questions           
  -------------------------------------------------------------------------

------------------------------------------------------------------------

# How to Choose

## Use Vector Retrieval when

The question is primarily:

``` text
"Find information with similar meaning."
```

Example:

``` text
"What is this technology used for?"
```

------------------------------------------------------------------------

## Use Text-to-Cypher when

The question is primarily:

``` text
"Query the graph structure or properties."
```

Example:

``` text
"Which accused persons are connected to FIR-101?"
```

------------------------------------------------------------------------

## Use Vector + Cypher when

The question is primarily:

``` text
"First find the relevant information semantically,
then explore its graph relationships."
```

Example:

``` text
"Find the incident related to this description
and tell me everyone connected to it."
```

------------------------------------------------------------------------

# Overall Architecture

``` text
                         USER QUESTION
                              |
                              v
                       +--------------+
                       |      LLM     |
                       +--------------+
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
      VECTOR RETRIEVAL   TEXT-TO-CYPHER   VECTOR + CYPHER
             |                |                |
             v                v                v
      Vector Index       Generate Cypher    Vector Search
             |                |                |
             v                v                v
       Relevant Text      Neo4j Query      Relevant Nodes
                              |                |
                              v                v
                         Graph Results     Graph Traversal
             |                |                |
             +----------------+----------------+
                              |
                              v
                     Retrieved Context
                              |
                              v
                         FINAL LLM
                              |
                              v
                           ANSWER
```

------------------------------------------------------------------------

# Quick Decision Guide

``` text
                         User Question
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
      Semantic text?    Structured graph?   Both?
             |                |                |
             v                v                v
       Vector RAG       Text-to-Cypher   Vector + Cypher
```

### Simple rule

**Vector Retrieval**

> "Find relevant content by meaning."

**Text-to-Cypher**

> "Query the graph using structured relationships and properties."

**Vector + Cypher**

> "Find the relevant graph entity semantically, then explore its
> relationships."

------------------------------------------------------------------------

# Key Takeaway

These three approaches solve different retrieval problems:

``` text
Vector Retrieval
    → Semantic understanding

Text-to-Cypher
    → Structured graph querying

Vector + Cypher
    → Semantic discovery + graph relationships
```

They are not necessarily mutually exclusive. A real GraphRAG application
can combine retrieval strategies depending on the type of user question.

------------------------------------------------------------------------

# Official Neo4j References

-   Neo4j GraphRAG Python package:
    https://neo4j.com/docs/neo4j-graphrag-python/current/

-   Neo4j GraphRAG RAG guide:
    https://neo4j.com/docs/neo4j-graphrag-python/current/user_guide_rag.html

-   Neo4j Vector Index documentation:
    https://neo4j.com/docs/cypher-manual/current/indexes/semantic-indexes/vector-indexes/

-   Neo4j Cypher documentation:
    https://neo4j.com/docs/cypher-manual/current/
