# Musical Instruments Recommendation System

A comparative study of popularity-based, user-based collaborative filtering, item-based collaborative filtering, matrix factorization (SVD), and hybrid recommendation approaches using Amazon Musical Instruments reviews.

## Results

**Test set: 2018 | 4,249 users with historical review activity**

Each model generates 10 recommendations per user, excluding items the user has already reviewed. A recommendation is considered a hit if the user subsequently reviews that item with a rating of 4 or 5.

| Model | Precision@10 | Recall@10 | Catalog Coverage |
|---|---:|---:|---:|
| Popularity (Baseline) | 0.0102 | 0.0237 | 0.0020 |
| User-based CF (k=500) | 0.0093 | 0.0307 | 0.2401 |
| Item-based CF | 0.0038 | 0.0146 | 0.8134 |
| SVD (20 factors) | 0.0012 | 0.0038 | 0.0340 |
| **Hybrid** | **0.0108** | **0.0334** | **0.1561** |

The hybrid uses popularity-based recommendations for users with two or fewer historical reviews and user-based collaborative filtering for users with more history.

### Key Findings

- Popularity is a strong baseline on sparse, positively skewed review data. No model substantially outperforms it in precision.
- The hybrid improves Recall@10 by approximately 41% over popularity (0.0334 vs. 0.0237), while maintaining comparable precision (0.0108 vs. 0.0102).
- Hybrid catalog coverage reaches 15.61%, compared with 0.20% for popularity, indicating a substantially wider range of recommended products.
- Item-based CF achieves the highest catalog coverage (81.34%), but has lower precision and recall.
- SVD achieves a validation RMSE of 0.969 compared with 1.003 for an average-rating baseline, but performs poorly on ranking metrics. This illustrates the difference between predicting ratings accurately and producing effective top-N recommendations.

Absolute Precision@10 is low because the dataset is highly sparse and users have relatively few relevant held-out interactions among approximately 10,000 products. Results should therefore be interpreted alongside the popularity baseline and other ranking metrics.

## Dataset

**Source:** [Amazon Review Data (2018)](https://cseweb.ucsd.edu/~jmcauley/datasets/amazon_v2)

**Category:** Musical Instruments, 5-core review file (Jianmo Ni, UCSD).

### Preprocessing

- Started with 231,392 reviews.
- Retained user ID, item ID, rating and timestamp.
- Removed repeated user-item interactions, retaining the latest review.
- Re-applied an iterative 5-core filter to ensure users and items met the minimum interaction requirement.

**Final dataset:**
- 206,241 reviews
- 24,780 users
- 9,930 items
- 99.92% sparsity

## Evaluation Protocol

### 1. Chronological Data Split

- **Training:** Reviews before January 2017.
- **Validation:** Reviews from January to December 2017.
- **Testing:** Reviews from January to September 2018.

Review interactions are split chronologically, preventing future interactions from entering model training. However, applying the 5-core filter to the complete dataset before splitting introduces future-informed user and item selection bias.

### 2. Hyperparameter Tuning

Model configurations were selected using validation data only:

- User-based CF: 500 nearest neighbours.
- SVD: 20 latent factors with regularization of 0.02.
- Hybrid: Popularity fallback for users with two or fewer historical reviews.

### 3. Final Evaluation

After tuning, models were rebuilt using training and validation data and evaluated once on the held-out 2018 test set.

### 4. Evaluation Metrics

The following ranking metrics were implemented from scratch:

- **Precision@10:** Fraction of the top-10 recommendations that are relevant.
- **Recall@10:** Fraction of relevant held-out items retrieved in the top 10.
- **Catalog Coverage:** Fraction of the available item catalogue recommended across all evaluated users.

### 5. Recommendation Constraints

- Previously reviewed items are excluded from recommendations.
- Only users with historical review activity are evaluated.
- Approximately 10% of test users (487) have no training history and are excluded from collaborative filtering evaluation. These users would receive popularity-based recommendations as a cold-start fallback.

## Models Implemented

### 1. Popularity-Based Recommendation

Recommends the most-reviewed products in the training dataset. Serves as a non
