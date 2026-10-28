# Musical Instruments Recommender

Popularity, user-based CF, item-based CF, SVD and a hybrid recommender, built and evaluated on Amazon Musical Instruments reviews with a time-based split.

## Results (test set, 2018)

4,249 users with history before 2018. Top-10 recommendations, items the user already reviewed excluded. A hit is an item the user later rated 4 or 5 stars.

| Model | Precision@10 | Recall@10 | Coverage |
|---|---|---|---|
| Popularity (baseline) | 0.0102 | 0.0237 | 0.0020 |
| User-based CF (k=500) | 0.0093 | 0.0307 | 0.2401 |
| Item-based CF | 0.0038 | 0.0146 | 0.8134 |
| SVD (20 factors) | 0.0012 | 0.0038 | 0.0340 |
| **Hybrid** (popularity if 2 or fewer reviews, else user-CF) | **0.0108** | **0.0334** | 0.1561 |

**Takeaways**
- Popularity is a strong baseline on sparse, positively skewed review data. No model clearly beat it on precision.
- The hybrid improves recall@10 by 41% over popularity (0.0334 vs 0.0237) at comparable precision, and raises catalog coverage from 0.2% to about 16%.
- Item-CF and SVD recommend a much wider range of products but are less accurate. SVD predicts ratings slightly better than guessing the average (validation RMSE 0.969 vs 1.003) but ranks poorly, which shows that predicting a rating and ranking items are different tasks.
- Absolute precision is low because each test user liked only 2 to 3 of about 10,000 products. What matters here is the comparison with the baseline.

## Data

[Amazon Review Data (2018)](https://cseweb.ucsd.edu/~jmcauley/datasets/amazon_v2), Musical Instruments, 5-core file (Jianmo Ni, UCSD).
- 231,392 reviews, reduced to user ID, item ID, rating and timestamp.
- Removed 12,236 duplicate (user, item) pairs, keeping the latest.
- Re-applied an iterative 5-core filter.
- Final data: 206,241 reviews, 24,780 users, 9,930 items, 99.92% sparse.

## Evaluation protocol

- **Time-based split:** train before 2017, validation in 2017, test in 2018 (up to September). No random splitting, so the model never sees the future.
- **Tuning on validation only:** user-CF neighbors (k=500), SVD factors and regularization (20 factors, 0.02), and the hybrid cutoff (2 reviews).
- **Test used once:** models were rebuilt on train + validation and scored once on the test set.
- **Metrics:** precision@10, recall@10 and catalog coverage, all written from scratch.
- **Rules:** items the user already reviewed are never recommended. Only users with history are scored. About 10% of test users (487) are new and are not scored; they would receive popularity.

## Models

- **Popularity:** the most-reviewed items in the training data.
- **User-based CF:** cosine similarity between users on a sparse matrix, weighted votes from the top-k neighbors.
- **Item-based CF:** cosine similarity between items, scored from the items a user has reviewed.
- **SVD:** matrix factorization with the Surprise library, ranked by predicted rating.
- **Hybrid:** popularity for users with 2 or fewer reviews, user-CF for everyone else.

## Limitations

- Reviews stand in for purchases, and many buyers never review.
- The data ends in September 2018.
- A breakdown of the test set by user activity suggested user-CF helps users with 6 or more reviews. This did not repeat on validation, so it is not claimed.

## Run it

Open `recommender.ipynb` in Google Colab and run all cells. The first cells download the data and install `scikit-surprise`; one cell saves the splits to Google Drive and asks for permission. The tuning cells take several minutes.
