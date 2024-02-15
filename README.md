# isc-iris-vector-search-eap
InterSystems IRIS Vector Search Early Access Program (EAP) materials. This repo contains information about how to get started using IRIS Vector Search, which brings the power of embedding vectors and the Retrieval Augmented Generation (RAG) applications to IRIS.

# IRIS Vector Search - Early Access Program

## Table of Contents

1. [Overview](#overview)
2. [IRIS "as a Vector Database"](#iris-as-a-vector-database)  
3. [The Early Access Program](#the-early-access-program)
4. [Early Access Program Materials](#early-access-program-materials)
   - [Kits and Containers:](#kits-and-containers)
   - [License Keys:](#license-keys)
   - [Documentation:](#documentation)
   - [Code Samples / Demos:](#code-samples)
5. [References](#references)
6. [Vectors as Indexes](#vectors-as-indexes)
7. [Vector Search Tutorial: Storing and Retrieving data using similarity](#vector-search-tutorial-storing-and-retrieving-data-using-similarity)
8. [Vector SQL datatype](#vector-sql-datatype)
   - [Using Vectors in SQL](#using-vectors-in-sql)
     - [VECTOR (type, length)](#vector-type-length)
       - [CREATE a table with vector columns:](#create-a-table-with-vector-columns)
       - [INSERT INTO a table with vector columns:](#insert-into-a-table-with-vector-columns)  
       - [SELECT FROM a table with vector columns:](#select-from-a-table-with-vector-columns)
   - [SQL Functions](#sql-functions) 
     - [TO_VECTOR (input, type, length)](#to_vector-input-type-length)
     - [VECTOR_COSINE (vec1, vec2)](#vector_cosine-vec1-vec2)
     - [VECTOR_DOT_PRODUCT (vec1, vec2)](#vector_dot_product-vec1-vec2)
   - [Nearest Neighbor Search](#nearest-neighbor-search)
     - [Getting the top 3 most similar vectors (to an input vector) from a table](#getting-the-top-3-most-similar-vectors-to-an-input-vector-from-a-table)
     - [Using Cosine Similarity:](#using-cosine-similarity)
     - [Using Dot Product:](#using-dot-product)

## Overview

## IRIS "as a Vector Database"  

First, a little about "vector databases" -- a niche within the database market that appeared when the first useful "language models" started being used to index unstructured (free text, natural language) data for search applications, in particular "recommendation engines" for "product search", and "sentiment analysis" use cases. The basic idea is that any bit of text could be converted to a dense numeric vector or array of some predetermined length, such as 128 dimensions, by a language model, such that text that had similar semantic meaning would be clustered "close together" in that large dimensional space. This content encoding was then very useful for recommending similar products, etc. Since traditional DBMSs did not support handling such large arrays as data types, vendors sprang up in the mid 2010's to fill this gap. Fast forward to today, and these language models are larger and more powerful, and vector search is a primary way to leverage this power for enterprise applications.  

InterSystems is not "bolting on" a separately engineered component -- vectors are "just another datatype" for us! Our recent addition of Columnar storage required a low-level ObjectScript $vector data type, which is the basis for the new vector datatype in SQL. For the 2024.1 release we are introducing as Experimental Features:  

- a new **[Vector SQL datatype](#vector-sql-datatype)** - vectors are lists of (typically) floats, that can be stored in a SQL table column, where all vectors in a given column are the same length. These vectors are the output of an embedding language model, and are used to represent the language model's estimation of the meaning of a passage of text. These vectors can then be compared with each other, and in the case of the RAG pattern, are compared to a vector derived from a query.

- new similarity functions that perform comparisons between 2 vectors:
   - [**VECTOR_DOT_PRODUCT()**](#VectorasaSQLdatatype-VECTOR_DOT_PRODUCT)
   - [**VECTOR_COSINE()**](#VectorasaSQLdatatype-VECTOR_COSINE) 

We are also updating our Community-developed ancillary components:
- Updated DB-API and SQLAlchemy drivers that will support python application development using the most popular LLM development frameworks and tools, including from Embedded Python, which will enable an easier path for InterSystems to build features  

- [LangChain VectorStore](https://github.com/langchain-ai/langchain/blob/master/libs/core/langchain_core/vectorstores.py) implementation -- LangChain is the most popular "LLM interoperability" framework, and our VectoreStore implementation will make using IRIS within these applications much simpler  

## The Early Access Program

Since this is exciting and new functionality in IRIS, naturally we want to get it into peoples' hands as quickly as possible, and we plan a comprehensive, phased EAP for this:  

| Phase | Goals (in priority order) | Audience | Availability | Timing |
| ------------- |:-------------|-------------:|-------------:|-------------:|
| 1 | - Identify defects and gaps in our implementation, before sharing publicly <br>- Solicit feedback on (initially limited) API surface and desirable extensions     | employees only <br>(focus on innovation teams) | internal-only kit, frequent builds | Week of Dec 18, 2023 |   
| 2 | - Identify real-world use cases and potential reference customer <br> - Solicit feedback on API surface and desirable extensions to feed 2024.2 roadmap <br> - Data points on performance | hands-on customers | 2024.1 developer preview, biweekly updates | mid-January, 2024 <br> once merged into DP stream |
| 3 | - Add reference customers <br> - Solicit feedback to finalize 2024.2 content <br> - Data points on performance | customers | 2024.1 GA release | align with 2024.1 release cycle |

## Early Access Program Materials  

### Kits and Containers: 
- 2024.1 Developer Preview kits and containers, both Enterprise and Community editions, support Vector Search
- "bleeding edge" kits and container images are available via download in the Early Access Program page

### License Keys:  

There are also license keys that you can use with these kits/containers, just download and apply the key for the version (kit or container) you want to install.   

### Documentation:  

- [Vector SQL datatype](#vector-sql-datatype) -- quick reference / how-to  

### Code Samples:  

- [Vector Search Tutorial: Storing and Retrieving data using similarity](#vector-search-tutorial-storing-and-retrieving-data-using-similarity)

- [Vectors as Indexes](#vectors-as-indexes)  

## References  

## Vectors as Indexes  

**Introduction:**  

This article provides [experimental] code and instructions on using embeddings as indexes.  

**With one line of code, you can generate embeddings as indexes for your table, enabling similarity search on your data.**

```
> Index ContentEmb On (Title, Description) As Embeddings.Index(CHUNKSIZE = 200, MODEL = "sentence-transformers/all-MiniLM-L12-v2", OVERLAP = 10);
```

This functionality is still in early development. The goal is for embeddings to be abstracted away as an index, such that users can easy run similarity search on their data.  

The code provided is only a proof of concept, and is not production code. The syntax will be improved and the code will be more optimized before release.   

For a more customizable version that stores vectors as rows in a table, refer to [Vector Search Tutorial: Storing and Retrieving data using similarity](#vector-search-tutorial-storing-and-retrieving-data-using-similarity)  

**Instructions:**  

1. Install the IRIS instance (use the latest VecDB kit http://kits-web/kits/unreleased/IRIS/2024.1.0VECDB/)  

2. Setup your VSCode workspace with the IRIS instance  

3. Install python packages:  

   a. For mac users:  

     i. pip3 install --target <path_to_IRIS>/mgr/python numpy sentence_transformers  

     ii. For example: pip3 install --target /Users/aryanput/InterSystems/IRIS_DEMO/mgr/python numpy sentence_transformers langchain

   b. For windows users
