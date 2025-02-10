---
date:
  created: 2025-02-10
tags:
  - Qdrant
  - Anomaly Detection
authors:
  - myriel
hide:
  - navigation
---
![anomaly-detection-qdrant](/img/anomaly-detection-qdrant/social.png)

# Anomaly Detection at Billion-Vector Scale

Anomaly detection plays a crucial role in identifying unusual patterns or outliers in datasets across various industries. From detecting fraudulent transactions in finance to spotting manufacturing defects, the ability to identify anomalies efficiently is essential.

In this guide, we explore how vector databases, specifically Qdrant, can be leveraged for anomaly detection. Traditional methods often rely on statistical analysis or machine learning algorithms, which can be computationally expensive. Vector databases provide an alternative approach by enabling efficient similarity searches to detect anomalies.

## Understanding the Dataset

Blahblah
![anomaly-detection-qdrant](/img/anomaly-detection-qdrant/image1.png){ align=left }

For this guide, we use a synthetic dataset of customer transactions with the following fields:

- **Customer ID**: Unique identifier for customers
- **Transaction Amount**: Value of the transaction
- **Category**: Type of transaction (e.g., Clothing, Electronics, Travel)

We focus on detecting anomalies in these three key areas to identify unusual behavior or fraudulent activity.

## Implementation Steps

### Step 1: Setting Up the Environment

Install the necessary libraries:
```bash
pip install qdrant-client sentence-transformers
```

### Step 2: Importing Libraries and Loading Data

```python
import pandas as pd
file_path = '/content/updated_file.csv'
data = pd.read_csv(file_path)
```

### Step 3: Initializing Qdrant Client and Sentence Transformer

```python
from qdrant_client import QdrantClient
from qdrant_client.http import models
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('sentence-transformers/all-mpnet-base-v2')
client = QdrantClient("memory")
```

### Step 4: Preparing Data for Anomaly Detection

Convert relevant columns into vector embeddings:

```python
cust_ids = data['Customer ID'].astype(str).tolist()
transaction_amounts = data['Transaction Amount'].astype(str).tolist()
categories = data['Category'].tolist()

cust_id_vectors = model.encode(cust_ids).tolist()
transaction_vectors = model.encode(transaction_amounts).tolist()
category_vectors = model.encode(categories).tolist()
```

### Step 5: Creating Qdrant Collections

Define a function to create Qdrant collections and insert data:

```python
def create_and_insert(collection_name, vectors, items):
    client.create_collection(
        collection_name=collection_name,
        vectors_config=models.VectorParams(size=len(vectors[0]), distance=models.Distance.COSINE)
    )
    points = [
        models.PointStruct(id=i, vector=vector, payload={"item": item})
        for i, (vector, item) in enumerate(zip(vectors, items))
    ]
    client.upsert(collection_name=collection_name, points=points)

create_and_insert("customer_id_collection", cust_id_vectors, cust_ids)
create_and_insert("transaction_amount_collection", transaction_vectors, transaction_amounts)
create_and_insert("category_collection", category_vectors, categories)
```

### Step 6: Performing Anomaly Detection on Customer IDs

#### 6.1 Defining Normal Customer IDs

```python
positive_points = ["stu9101", "vwx3456", "rst6789", "stu7890", "lmn9101"]
positive_point_vectors = model.encode(positive_points).tolist()
```

#### 6.2 Querying Qdrant for Anomalies

```python
response = client.recommend(
    collection_name='customer_id_collection',
    negative=positive_point_vectors,
    limit=3,
    with_payload=True,
    strategy=models.RecommendStrategy.BEST_SCORE
)
for point in response:
    print(f"Anomaly: {point.payload['item']}, Score: {point.score}")
```

### Step 7: Identifying Unusual Categories

```python
positive_points = ["Clothing", "Electronics", "Travel", "Market", "Restaurant"]
positive_point_vectors = model.encode(positive_points).tolist()

response = client.recommend(
    collection_name='category_collection',
    negative=positive_point_vectors,
    limit=3,
    with_payload=True,
    strategy=models.RecommendStrategy.BEST_SCORE
)

for point in response:
    print(f"Anomaly: {point.payload['item']}, Score: {point.score}")
```

![anomaly-detection-qdrant](/img/anomaly-detection-qdrant/image2.png)

![anomaly-detection-qdrant](/img/anomaly-detection-qdrant/image3.png)

![anomaly-detection-qdrant](/img/anomaly-detection-qdrant/image4.png)

![anomaly-detection-qdrant](/img/anomaly-detection-qdrant/image5.png)


## Conclusion
![anomaly-detection-qdrant](/img/anomaly-detection-qdrant/image6.png)

This guide demonstrated how to use Qdrant for anomaly detection in a transaction dataset. By leveraging vector similarity search, we efficiently identified anomalies in customer IDs, transaction amounts, and transaction categories.

### Advantages of Using a Vector Database for Anomaly Detection
1. **Efficiency**: Fast querying even with large datasets.
2. **Flexibility**: Adaptable to different data types and anomaly patterns.
3. **No predefined rules needed**: Detects novel anomalies dynamically.
4. **Scalability**: Handles increasing data efficiently.

Using vector databases enhances anomaly detection, making it more effective for fraud detection, cybersecurity, and financial applications.
