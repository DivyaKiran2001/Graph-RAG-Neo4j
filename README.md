

## Overview

GraphRAG combines a Large Language Model (LLM) with retrieval from a Neo4j knowledge graph. Instead of relying only on the LLM's internal knowledge, the application retrieves relevant information from Neo4j and provides that information as context before generating the final answer.

This README focuses on three important approaches:

1. **Vector Retrieval**
2. **Text-to-Cypher**
3. **Vector + Cypher Graph Retrieval**

---

# 1. Vector Retrieval

## What is Vector Retrieval?

Vector retrieval finds information based on **semantic similarity**.

The data is converted into embeddings and stored in a Neo4j vector index. When a user asks a question:

```text
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
