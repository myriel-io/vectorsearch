---
date:
  created: 2024-12-01
categories:
  - Vector Search
  - Vector Databases
tags:
  - Qdrant
  - OpenAI
authors:
  - myriel
---

# Inverted Index 

## Definition  
An **inverted index** is a data structure that maps content (such as keywords or terms) to their locations within a dataset, enabling fast lookup and filtering operations. In the context of Qdrant, the inverted index is used to optimize filtering capabilities by allowing efficient retrieval of vectors based on specific payload conditions, such as filtering by metadata or tags associated with the stored vectors.

This mechanism is particularly useful in hybrid search scenarios where sparse (keyword-based) filtering is combined with dense (vector-based) similarity searches.

---

## Example in Qdrant  

Imagine you are building a recommendation engine for an e-commerce platform. Each vector represents a product, and payloads (metadata) include fields such as `category`, `price`, and `brand`.

### Creating a Collection with an Inverted Index

```json
POST /collections/products
{
  "vectors": {
    "size": 128,
    "distance": "Cosine"
  },
  "payload_schema": {
    "category": {
      "type": "keyword",
      "index": true
    },
    "price": {
      "type": "integer",
      "index": true
    },
    "brand": {
      "type": "keyword",
      "index": true
    }
  }
}
```

In this configuration:
- **`category` and `brand`** are indexed as `keyword`, allowing filtering by exact matches.
- **`price`** is indexed as `integer`, enabling range queries (e.g., products priced between $10 and $50).

---

## Query Example  

To retrieve vectors for all products in the `electronics` category with a price between $50 and $200:

```json
POST /collections/products/points/search
{
  "filter": {
    "must": [
      { "key": "category", "match": { "value": "electronics" } },
      { "key": "price", "range": { "gte": 50, "lte": 200 } }
    ]
  },
  "vector": [0.1, 0.2, 0.3, ...],
  "top": 10
}
```

### Result  
The inverted index ensures that the filter step is efficient, significantly reducing the search space before the similarity search is performed.

---

## Why It Matters  
An inverted index in Qdrant allows developers to create powerful, real-time search applications that combine metadata filtering and semantic similarity, optimizing both speed and relevance.

---

## Tabular Example  

Suppose we have the following dataset of products:

| **Product ID** | **Category**     | **Brand**       | **Price** |
|-----------------|------------------|-----------------|-----------|
| 1               | Electronics      | Samsung         | 150       |
| 2               | Electronics      | Apple           | 200       |
| 3               | Home Appliances | Samsung         | 300       |
| 4               | Electronics      | Sony            | 100       |
| 5               | Furniture        | IKEA            | 250       |

Based on this data, an **inverted index** could look like this:

| **Key**         | **Value**                              |
|------------------|---------------------------------------|
| `category:electronics` | Product IDs: [1, 2, 4]            |
| `category:home appliances` | Product IDs: [3]                |
| `category:furniture`     | Product IDs: [5]                |
| `brand:samsung`          | Product IDs: [1, 3]            |
| `brand:apple`            | Product IDs: [2]               |
| `brand:sony`             | Product IDs: [4]               |
| `brand:ikea`             | Product IDs: [5]               |
| `price_range:0-100`      | Product IDs: []                |
| `price_range:101-200`    | Product IDs: [1, 4]            |
| `price_range:201-300`    | Product IDs: [2, 5]            |
| `price_range:301-400`    | Product IDs: [3]               |

### Explanation:

- The inverted index maps keys (like `category:electronics` or `brand:samsung`) to a list of **Product IDs**.
- It can also include derived keys, such as `price_range`, which groups prices into ranges.
  
This structure allows efficient filtering, as you can quickly retrieve all product IDs for a specific category, brand, or price range without scanning the entire dataset.
