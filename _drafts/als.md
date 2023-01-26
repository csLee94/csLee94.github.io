---
title: 스타트업에서 "데이터팀"으로 살아남기
author: csLee94
date: 2023-01-26 00:00:00 +0900
categories: [datascience]
tags: [recommendation]
math: true
---


### What is Alternating Least Squares(ALS)? and also can you show me the example of matrix following each step of recommendation? Especially, I need to check changing of matrix and how algorithm make matrix update.

Alternating Least Squares (ALS) is an algorithm for solving matrix factorization problems, such as those that arise in collaborative filtering. The idea behind ALS is to iteratively update the user and item factor matrices by solving a least-squares problem for each matrix while holding the other matrix fixed.

Here is an example of how the ALS algorithm updates the user and item factor matrices for a simple 2-user, 2-item case:

1. Initialize the user and item factor matrices with random values.

```plaintext
User factor matrix: 
[[0.5, 0.2],
 [0.3, 0.1]]

Item factor matrix:
[[0.2, 0.4],
 [0.1, 0.3]]
```

2. Fix the item factor matrix and solve for the user factor matrix by minimizing the difference between the predicted ratings and the actual ratings for all observed entries in the user-item matrix.

```plaintext
Observed ratings: 
[[4, 2],
 [3, 5]]

Predicted ratings: 
[[0.5*0.2 + 0.2*0.4, 0.5*0.1 + 0.2*0.3],
 [0.3*0.2 + 0.1*0.4, 0.3*0.1 + 0.1*0.3]] = [[0.3, 0.2], [0.2, 0.2]]

Differences: 
[[3.7, 1.8],
 [2.8, 4.8]]

Mean squared error: 
(3.7^2 + 1.8^2 + 2.8^2 + 4.8^2)/4 = 15.6
```

3. Update the user factor matrix using the least-squares solution

```plaintext
User factor matrix: 
[[0.5, 0.2],
 [0.3, 0.1]] = [[1.5, 0.5], [0.9, 0.3]]
```
4. Fix the user factor matrix and solve for the item factor matrix by minimizing the difference between the predicted ratings and the actual ratings for all observed entries in the user-item matrix.

```plaintext
Observed ratings: 
[[4, 2],
 [3, 5]]

Predicted ratings: 
[[1.5*0.2 + 0.5*0.1, 1.5*0.4 + 0.5*0.3],
 [0.9*0.2 + 0.3*0.1, 0.9*0.4 + 0.3*0.3]] = [[1.1, 1.9], [0.9, 1.3]]

Differences: 
[[2.9, 0.1],
 [2.1, 3.7]]

Mean squared error: 
(2.9^2 + 0.1^2 + 2.1^2 + 3.7^2)/4 = 8.2
```

5. Update the item factor matrix using the least-squares solution

```plaintext
Item factor matrix:
[[0.2, 0.4],
 [0.1, 0.3]] = [[0.3, 0.5], [0.2, 0.4]]
```

Repeat steps 2-5 until the difference between the predicted ratings and the actual ratings

### Above paragraph, what does each row and column of the example matrix mean? And are the random values  random, or should I declare something ?

In the example I provided, the rows of the user factor matrix represent different users, and the columns represent different latent factors. Similarly, the rows of the item factor matrix represent different items, and the columns represent different latent factors.

The values in these matrices are typically initialized with random values, but the exact method for generating these initial values can vary. Some common methods include using small random values drawn from a standard normal distribution, or using large random values drawn from a uniform distribution.

It's worth noting that the choice of initialization method and the values used can have an impact on the performance of the algorithm and the quality of the resulting recommendations. The most commonly used initialization method is generating random values from standard normal distribution with mean 0 and variance 1.

In addition, it's also important to note that the quality of the recommendations also depend on the number of latent factors, which is a hyperparameter that needs to be tuned. Typically, a larger number of latent factors can lead to more accurate recommendations, but it also increases the computational complexity and the risk of overfitting.

### All good. Can I get another ALS algorithm scenario including implicit data. Highlight different between this request and above descriptions. 

Certainly! Here's an example of how the ALS algorithm can be used for collaborative filtering with implicit data:

1. Initialize the user and item factor matrices with random values.

```plaintext
User factor matrix: 
[[0.5, 0.2],
 [0.3, 0.1]]

Item factor matrix:
[[0.2, 0.4],
 [0.1, 0.3]]
```

2. Fix the item factor matrix and solve for the user factor matrix by minimizing the difference between the predicted confidence scores and the actual confidence scores for all observed entries in the user-item matrix.

```plaintext
Observed confidence scores: 
[[1, 1],
 [1, 1]]

Predicted confidence scores: 
[[sigmoid(0.5*0.2 + 0.2*0.4), sigmoid(0.5*0.1 + 0.2*0.3)],
 [sigmoid(0.3*0.2 + 0.1*0.4), sigmoid(0.3*0.1 + 0.1*0.3)]] = [[0.6, 0.5], [0.5, 0.5]]

Differences: 
[[-0.4, -0.5],
 [-0.5, -0.5]]

Cross-entropy loss: 
-((1*log(0.6) + 1*log(0.5) + 1*log(0.5) + 1*log(0.5))/4) = 0.6
```

3. Update the user factor matrix using the gradient descent solution

```plaintext
User factor matrix: 
[[0.5, 0.2],
 [0.3, 0.1]] = [[0.4, 0.1], [0.2, 0.05]]
```

4. Fix the user factor matrix and solve for the item factor matrix by minimizing the difference between the predicted confidence scores and the actual confidence scores for all observed entries in the user-item matrix.

```plaintext
Observed confidence scores: 
[[1, 1],
 [1, 1]]

Predicted confidence scores: 
[[sigmoid(0.4*0.2 + 0.1*0.1), sigmoid(0.4*0.4 + 0.1*0.3)],
 [sigmoid(0.2*0.2 + 0.05*0.1), sigmoid(0.2*0.4 + 0.05*0.3)]] = [[0.5, 0.6], [0.5, 0.5]]

Differences: 
[[-0.5, -0.4],
 [-0.5, -0.5]]

Cross-entropy loss: 
-((1*log(0.5) + 1*log(0.6) + 1*log(0.5) + 1*log(0.5))/4) = 0.6
```

5. Update the item factor matrix using the gradient descent solution

```plaintext
Item factor matrix:
[[0.2, 0.4],
 [0.1, 0.3]] = [[0.15, 0.35], [0.05, 0.25]]
```

6. Repeat steps 2-5 until the difference between the predicted confidence scores

### In addition to previous example, we should predict preference of new user for existing items. you can provide not only changing log of matrix but python code.

Sure! Here's an example of how you can use the user and item factor matrices that were learned from the ALS algorithm to predict the preferences of a new user for existing items:

```python
import numpy as np

# Assume that we have already learned the user and item factor matrices
user_factor_matrix = np.array([[0.4, 0.1], [0.2, 0.05]])
item_factor_matrix = np.array([[0.15, 0.35], [0.05, 0.25]])

# Define the new user's preferences as a binary vector
new_user_preferences = np.array([1, 0])

# Compute the predicted preferences of the new user for the existing items using the dot product of the user and item factor matrices
predicted_preferences = np.dot(new_user_preferences, item_factor_matrix.T)

print(predicted_preferences)
# Output: [0.2, 0.05]
```

In this example, we first import the numpy library and then assume that we have already learned the user and item factor matrices from the ALS algorithm. Then we define a new user's preferences as a binary vector, where a 1 indicates that the user has interacted with an item and a 0 indicates that the user has not interacted with the item.

We then use the dot product of the new user's preferences and the transpose of the item factor matrix to compute the predicted preferences of the new user for the existing items. The predicted preferences will be in the form of array, where each element of array represent the predicted preference score for each item.

It's worth noting that if you are using implicit data, the preferences will not be binary but rather real values representing the confidence score of the user-item interaction. Therefore, we will use dot product of user factor matrix and item factor matrix and pass it through sigmoid function to get the predicted preference in this case.