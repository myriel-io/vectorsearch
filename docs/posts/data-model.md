---
date:
  created: 2024-11-30
categories:
  - Vector Search
  - Vector Databases
tags:
  - Qdrant
  - OpenAI
authors:
  - myriel
---

# What is a Data Model for Unstructured Data?

Here is an example of a data model in the context of unstructured data: 

## Data Model Example: Document Search System

This is an example of a data model tailored for unstructured data in the context of a document search system, such as one built using a vector database like Qdrant.

---

## Entity: Document

| Field Name        | Data Type         | Description                                                                                  |
|--------------------|-------------------|----------------------------------------------------------------------------------------------|
| `id`              | String (UUID)     | Unique identifier for the document.                                                         |
| `title`           | String            | The title of the document.                                                                  |
| `content_vector`  | Float Array       | Dense vector representation of the document content, generated using a pre-trained language model (e.g., OpenAI, BERT). |
| `metadata`        | Object (JSON)     | Key-value pairs storing metadata about the document (e.g., author, date, tags).             |
| `categories`      | Array of Strings  | List of categories the document belongs to (e.g., "contract law", "intellectual property"). |
| `created_at`      | DateTime          | Timestamp when the document was created.                                                    |
| `updated_at`      | DateTime          | Timestamp when the document was last updated.                                               |

---

## Example JSON Representation

```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "title": "Copyright Law in the Digital Age",
  "content_vector": [0.123, 0.987, 0.456, ...], 
  "metadata": {
    "author": "Jane Doe",
    "publish_date": "2024-01-15",
    "language": "English"
  },
  "categories": ["copyright law", "digital media"],
  "created_at": "2024-01-15T10:00:00Z",
  "updated_at": "2024-11-30T12:00:00Z"
}
```
## Why This Model?

1.	Flexibility: The unstructured content_vector enables similarity search, while structured metadata supports filtering and faceting.
2.	Extensibility: You can add new fields (e.g., “related documents”) without major schema changes.
3.	Efficiency: Vector-based retrieval is efficient for unstructured text, while metadata aids precise filtering.