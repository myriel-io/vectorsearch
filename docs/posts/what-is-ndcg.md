---
date:
  created: 2023-05-15
tags:
  - Vector Search
  - Information Retrieval
  - Evaluation Metrics
authors:
  - myriel
hide:
  - navigation
---
# What is NDCG?

![ndcg-metric](/img/what-is-ndcg/social.png)

> Note: An image needs to be created for this article. The SVG has been created in `/img/what-is-ndcg/social.svg` but needs to be converted to PNG.

Let's break down the idea of NDCG (Normalized Discounted Cumulative Gain) using a small, easy-to-follow example:

## The scenario

Imagine we have a simple "search engine" returning four results for a query. Each result has a relevance grade (like a "score" or "star rating"), typically assigned by some human judgment or another method.

Let's say the relevance scores for the four items (in the ideal, best order) are:
1. Result A: Relevance = 3
2. Result B: Relevance = 2
3. Result C: Relevance = 2
4. Result D: Relevance = 1

In an ideal world, the search engine would show these results in the order: A, B, C, D.

## The question

What if your actual ranking (the order your system outputs) is slightly different or not perfect? How do we measure how good your ranking is compared to the best possible?

## Step 1: Discounted Cumulative Gain (DCG)

DCG rewards having high-relevance items near the top of the list. It does so by:
1. Adding up relevance scores of items in the order you ranked them.
2. Discounting (reducing) the score for items that appear lower in the list, so top-ranked items "count" more.

Mathematically, one common version of the formula for DCG at position i uses:

$$\frac{2^{\text{rel}_i} - 1}{\log_2(i+1)}$$

But if that looks too complicated, you can think of it in words:
- Take the relevance score of the item.
- Divide by something that gets bigger as i increases (like the log of the rank).

Hence, items in rank 1, 2, or 3 contribute more to the overall score than those in rank 10, 11, or 12.

## Step 2: Compute Ideal DCG (IDCG)

The Ideal DCG ($\text{IDCG}$) is what you get if you rank items in the absolute best order (the one that puts the highest relevance first, second-highest relevance second, etc.). This serves as the highest possible DCG you could get.

## Step 3: NDCG

Finally, Normalized DCG (NDCG) just divides the DCG you got by the best (ideal) DCG. That way, your ranking gets a score between 0 and 1:

$$\text{NDCG} = \frac{\text{DCG of your ranking}}{\text{DCG of the ideal ranking}}$$

- If your ranking is perfect (it exactly matches the ideal order), NDCG = 1.
- If your ranking is completely upside down (worst items at the top), NDCG will be much lower, closer to 0.

## A small, concrete example

Let's imagine your system returns these four items in this order:
1. B (relevance = 2)
2. A (relevance = 3)
3. D (relevance = 1)
4. C (relevance = 2)

### (a) Calculate DCG for your system's order:

We'll label each position i and use the (simplified) formula for DCG at rank i:

$$\text{DCG}_i = \frac{2^{\text{relevance of item at rank } i} - 1}{\log_2(i+1)}$$

- Rank 1: Item B (relevance = 2)
  $$\frac{2^2 - 1}{\log_2(1+1)} = \frac{3}{\log_2(2)} = \frac{3}{1} = 3$$

- Rank 2: Item A (relevance = 3)
  $$\frac{2^3 - 1}{\log_2(2+1)} = \frac{7}{\log_2(3)} \approx \frac{7}{1.585} \approx 4.41$$

- Rank 3: Item D (relevance = 1)
  $$\frac{2^1 - 1}{\log_2(3+1)} = \frac{1}{\log_2(4)} = \frac{1}{2} = 0.5$$

- Rank 4: Item C (relevance = 2)
  $$\frac{2^2 - 1}{\log_2(4+1)} = \frac{3}{\log_2(5)} \approx \frac{3}{2.32} \approx 1.29$$

Now sum these up for the total DCG:
$$\text{DCG} = 3 + 4.41 + 0.5 + 1.29 \approx 9.20$$

### (b) Calculate Ideal DCG (IDCG) for the perfect order:

The perfect order is: A (3), B(2), C(2), D(1). Let's do the same calculation.

- Rank 1: A (3)
  $$\frac{2^3 - 1}{\log_2(1+1)} = \frac{7}{1} = 7$$

- Rank 2: B (2)
  $$\frac{2^2 - 1}{\log_2(2+1)} = \frac{3}{\log_2(3)} \approx \frac{3}{1.585} \approx 1.89$$

- Rank 3: C (2)
  $$\frac{2^2 - 1}{\log_2(3+1)} = \frac{3}{2} = 1.5$$

- Rank 4: D (1)
  $$\frac{2^1 - 1}{\log_2(4+1)} = \frac{1}{\log_2(5)} \approx \frac{1}{2.32} \approx 0.43$$

Now sum these:
$$\text{IDCG} = 7 + 1.89 + 1.5 + 0.43 \approx 10.82$$

### (c) Calculate the NDCG:

$$\text{NDCG} = \frac{\text{DCG}}{\text{IDCG}} \approx \frac{9.20}{10.82} \approx 0.85$$

So your system's ranking yields an NDCG of about 0.85.
- If your system matched the ideal exactly, the NDCG would be $\frac{10.82}{10.82} = 1$.
- If your system were much worse, the DCG would be lower, making NDCG closer to 0.

## Key takeaway
- NDCG tells you how close your ranking is to the "best possible" ranking by measuring how well you place the most relevant items at the top.
- A score near 1 means you're doing very well; a score near 0 means you're doing poorly.

## Why NDCG Matters for Vector Search

In vector search systems:

1. **Relevance isn't binary** - Documents can be partially relevant to a query
2. **User attention is limited** - Users focus primarily on top results
3. **Comparison is essential** - You need to compare different embedding models and retrieval approaches

NDCG addresses all these concerns, making it an excellent metric for evaluating and optimizing vector search systems.

## Implementing NDCG in Python

Here's a simple Python implementation using NumPy:

```python
import numpy as np

def dcg(relevance_scores, k=None):
    """Calculate Discounted Cumulative Gain"""
    if k is not None:
        relevance_scores = relevance_scores[:k]
    
    return np.sum(relevance_scores / np.log2(np.arange(2, len(relevance_scores) + 2)))

def ndcg(relevance_scores, ideal_relevance_scores, k=None):
    """Calculate Normalized Discounted Cumulative Gain"""
    if k is not None:
        relevance_scores = relevance_scores[:k]
        ideal_relevance_scores = ideal_relevance_scores[:k]
    
    dcg_value = dcg(relevance_scores)
    ideal_dcg_value = dcg(ideal_relevance_scores)
    
    if ideal_dcg_value == 0:
        return 0.0
    
    return dcg_value / ideal_dcg_value

# Example usage
actual_relevance = np.array([3, 2, 3, 0, 1])
ideal_relevance = np.array([3, 3, 2, 1, 0])  # Sorted in descending order

print(f"NDCG: {ndcg(actual_relevance, ideal_relevance)}")
```

## Conclusion

NDCG is an essential metric for evaluating ranking systems, particularly in vector search applications. It effectively balances the twin concerns of relevance and position, giving you a reliable measure of your search quality.

When optimizing your vector search system, NDCG should be one of your primary evaluation metrics, helping you quantify improvements as you refine your embeddings, similarity metrics, and retrieval algorithms. 