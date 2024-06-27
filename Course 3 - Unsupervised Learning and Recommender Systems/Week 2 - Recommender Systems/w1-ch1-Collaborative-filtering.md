# Week 2: Recommender Systems

**Learning Objectives**
- Implement collaborative filtering recommender systems in TensorFlow
- Implement deep learning content based filtering using a neural network in TensorFlow
- Understand ethical considerations in building recommender systems

---

## Ch 1: Collaborative filtering

### Recommender Systems

A **Recommendation System** is a type of information filtering system which predicts the preferences of a user for specific items.
- It provides **personalized** recommendations to users based on their past preferences or other users' preferences for similar items.

> This is one of the topics that has received quite a bit of attention in academia. But the commercial impact and the actual number of practical use cases of recommended systems seems to me to be even vastly greater than the amount of attention it has received in academia. ~ _Andrew Ng_

#### Example: Movie Recommendation System

Say, we have some movies and some users who have rated these movies on a scale of $0 - 5$. We can represent this data in a matrix form where each row represents a movie and each column represents a user. 
The matrix (data) is sparse because not all users have rated all movies, meaning that there might be some missing values represented by $?$, which we need to predict.

| Movie          | User 1 | User 2 | User 3 | User 4 | User 5 |
| -------------- | :----: | :----: | :----: | :----: | :----: |
| *Avatar*       |   1    |   2    |   3    |   5    |   2    |
| *Brahmastra*   |   5    |   4    |   ?    |   0    |   0    |
| *Inception*    |   2    |   5    |   4    |   0    |   ?    |
| *Interstellar* |   4    |   ?    |   5    |   3    |   5    |
| *Titanic*      |   3    |   0    |   0    |   ?    |   4    |
| ...            |  ...   |  ...   |  ...   |  ...   |  ...   |

**Notations**:
- $n_u$ = number of users
- $n_m$ = number of movies
- $r(i, j) = 1$ if user $j$ has rated movie $i$ else $0$
- $y^{(i, j)}$ = rating given by user $j$ to movie $i$ (if rated)

##### Approach
One way we can approach this problem is to look at the movies that users have not rated and try to predict the ratings for those movies. We can then recommend the movies with the highest predicted ratings to the users.

---

### User-per Item Features

Let's say with different user ratings, we have some more features for each movie. For example: $x_1$ = Romance, $x_2$ = Action, etc... These features can be used to predict the ratings for the movies.

#### Data

| Movie          | User 1 | User 2 | User 3 | User 4 | User 5 | $x_1$ (Romance) | $x_2$ (Action) |
| -------------- | :----: | :----: | :----: | :----: | :----: | :-------------: | :------------: |
| *Avatar*       |   1    |   2    |   3    |   5    |   2    |       0.6       |      0.7       |
| *Brahmastra*   |   5    |   4    |   ?    |   0    |   0    |       0.8       |      0.9       |
| *Inception*    |   2    |   5    |   4    |   0    |   ?    |       0.4       |      0.9       |
| *Interstellar* |   4    |   ?    |   5    |   3    |   5    |       0.7       |      0.6       |
| *Titanic*      |   3    |   0    |   0    |   ?    |   4    |       0.9       |      0.2       |


#### Notations
- $n$ = number of features
- $x^{(i)}$ = feature vector for movie $i$
- $w^{(j)}, b^{(j)}$ = parameters for user $j$
- $r(i, j) = 1$ if user $j$ has rated movie $i$ else $0$
- $y^{(i, j)}$ = rating given by user $j$ to movie $i$ (if rated)
- $m^{(j)}$ = number of movies rated by user $j$


Here, we have $n=2$ features for each movie, where for each movie $i$, we have a feature vector $x^{(i)} \in \mathbb{R}^2$ and each feature is in the range $0-1$.

For movie $1$ - _Avatar_:

$$x^{(1)} = \begin{bmatrix} 0.6 \\ 0.7 \\ \end{bmatrix}$$

For movie $4$ - _Interstellar_:

$$x^{(4)} = \begin{bmatrix} 0.7 \\ 0.6 \\ \end{bmatrix}$$

#### Prediction

Now, to predict rating for a movie $i$ by user $j$, we can use the simple **Linear Regression** model:

$$f(x^{(i)})_{(w^{(j)}, b^{(j)})} = w^{(j)} \cdot x^{(i)} + b^{(j)}$$

where $x^{(i)}$ is the feature vector for movie $i$ and $w^{(j)}$ and $b^{(j)}$ are the parameters for user $j$.


Example: For user $3$ and movie $2$ - _Brahmastra_:

$$f(x^{(2)})_{(w^{(3)}, b^{(3)})} = w^{(3)} \cdot x^{(2)} + b^{(3)}$$

and say,

$$w^{(3)} = \begin{bmatrix} 0.9 \\ 0.3 \\ \end{bmatrix}, \quad b^{(3)} = 0.1$$

then,

$$f(x^{(2)})_{(w^{(3)}, b^{(3)})} = \begin{bmatrix} 0.9 \\ 0.3 \\ \end{bmatrix} \cdot \begin{bmatrix} 0.8 \\ 0.9 \\ \end{bmatrix} + 0.1 $$

$$f(x^{(2)})_{(w^{(3)}, b^{(3)})} = 0.72 + 0.27 + 3 = 3.99$$

So, the predicted rating for user $3$ and movie $2$ _Brahmastra_ is $3.99$.

#### Cost Function

The cost function for this model can be the **Mean Squared Error**:

$$ J(w^{(j)}, b^{(j)}) = \frac{1}{2m^{(j)}} \sum_{i:r(i, j)=1} \left( f(x^{(i)})_{(w^{(j)}, b^{(j)})} - y^{(i, j)} \right)^2 $$

which can be exapanded as:

$$ J(w^{(j)}, b^{(j)}) = \frac{1}{2m^{(j)}} \sum_{i:r(i, j)=1} \left( w^{(j)} \cdot x^{(i)} + b^{(j)} - y^{(i, j)} \right)^2 $$

Here, we are summing over all the movies rated by user $j$.

W can also add a regularization term - **L2 (*Ridge*) regularization** to the cost function to prevent overfitting. The cost function with regularization term is given by:

$$  J(w^{(j)}, b^{(j)}) = \frac{1}{2m^{(j)}} \sum_{i:r(i, j)=1} \left( w^{(j)} \cdot x^{(i)} + b^{(j)} - y^{(i, j)} \right)^2 + \frac{\lambda}{2m^{(j)}} \sum_{k=1}^n \left( w_k^{(j)} \right)^2 $$

where $\lambda$ is the regularization parameter and regularization is happening over all the parameters $w_k^{(j)}$, where $k=1, 2, \ldots, n$ (# features).

#### Modification for Multiple Users

Here, in the cost function, we need to do a little modification, i.e. rather than summing over all the movies rated by user $j$, we need to sum over all the users who have rated movie $i$.

Then the cost function for multiple users can be given by:

$$ J \begin{pmatrix} w^{(1)} & w^{(2)} & \cdots & w^{(n_u)} \\ b^{(1)} & b^{(2)} & \cdots & b^{(n_u)} \end{pmatrix}  = \frac{1}{2} \sum_{j=1}^{n_u} \sum_{i:r(i, j)=1} \left( w^{(j)} \cdot x^{(i)} + b^{(j)} - y^{(i, j)} \right)^2 $$

And the regularization term can be added as:

$$ J \begin{pmatrix} w^{(1)} & w^{(2)} & \cdots & w^{(n_u)} \\ b^{(1)} & b^{(2)} & \cdots & b^{(n_u)} \end{pmatrix}  = \frac{1}{2} \sum_{j=1}^{n_u} \sum_{i:r(i, j)=1} \left( w^{(j)} \cdot x^{(i)} + b^{(j)} - y^{(i, j)} \right)^2 + \frac{\lambda}{2} \sum_{j=1}^{n_u} \sum_{k=1}^n \left( w_k^{(j)} \right)^2 $$


Now, instead of optimizing the parameters for each user separately, we can optimize all the parameters together using **Gradient Descent**.

---

### Collaborative Filtering Algorithm

The **Collaborative Filtering Algorithm** is a method to make automatic predictions (filtering) about the interests of a user by collecting preferences from many users (collaborating).

#### Motivation

Let's take the same dataset we had earlier:

| Movie          | User 1 | User 2 | User 3 | User 4 | User 5 | $x_1$ | $x_2$ |
| -------------- | :----: | :----: | :----: | :----: | :----: | :---: | :---: |
| *Avatar*       |   1    |   2    |   3    |   5    |   2    |   ?   |   ?   |
| *Brahmastra*   |   5    |   4    |   ?    |   0    |   0    |   ?   |   ?   |
| *Inception*    |   2    |   5    |   4    |   0    |   ?    |   ?   |   ?   |
| *Interstellar* |   4    |   ?    |   5    |   3    |   5    |   ?   |   ?   |
| *Titanic*      |   3    |   0    |   0    |   ?    |   4    |   ?   |   ?   |

But this time, we don't have the actual features, rather we have created $2$ dummy features $x_1$ and $x_2$ for each movie. We only have the ratings given by the users for the movies.

Suppose, we have the optimal weights $w^{(j)}$ and biases $b^{(j)}$ for each user $j$.

Example: For $1^{st}$ movie and we have following optimal weights and biases respective for each user:

$$w^{(1)} = \begin{bmatrix} 0.9 \\ 0.3 \\ \end{bmatrix}, \quad b^{(1)} = 0$$
$$w^{(2)} = \begin{bmatrix} 0.2 \\ 0.8 \\ \end{bmatrix}, \quad b^{(2)} = 0$$
$$w^{(3)} = \begin{bmatrix} 0.5 \\ 0.7 \\ \end{bmatrix}, \quad b^{(3)} = 0$$
$$w^{(4)} = \begin{bmatrix} 0.4 \\ 0.6 \\ \end{bmatrix}, \quad b^{(4)} = 0$$
$$w^{(5)} = \begin{bmatrix} 0.6 \\ 0.5 \\ \end{bmatrix}, \quad b^{(5)} = 0$$

And we not consider the bias term for simplicity, so $b^{(j)} = 0$ for all $j$.

Now, using these optimal weights, we can optimize the features for each movie $i$ using the same **Linear Regression** equation:

$$f(x^{(i)})_{(w^{(j)}, b^{(j)})} = w^{(j)} \cdot x^{(i)} + b^{(j)}$$

And the cost function for this model with given $w^{(j)}$ and $b^{(j)}$ to learn $x^{(i)}$ for each movie $i$ can be given by:

$$ J(x^{(i)}) = \frac{1}{2} \sum_{j:r(i, j)=1} \left( w^{(j)} \cdot x^{(i)} - y^{(i, j)} \right)^2 + \frac{\lambda}{2} \sum_{k=1}^n \left( x_k^{(i)} \right)^2 $$

And to learn all the features for all the movies, it can be given by:

$$ J \begin{pmatrix} x^{(1)} & x^{(2)} & \cdots & x^{(n_m)} \end{pmatrix} = \frac{1}{2} \sum_{i=1}^{n_m} \sum_{j:r(i, j)=1} \left( w^{(j)} \cdot x^{(i)} - y^{(i, j)} \right)^2 + \frac{\lambda}{2} \sum_{i=1}^{n_m} \sum_{k=1}^n \left( x_k^{(i)} \right)^2 $$

we can optimize the cost function using **Gradient Descent**.

> By optimizing the features for each movie, we can get the optimal features for each movie which can be used to predict the ratings for the movies.

Now, we have $2$ cost functions:

1. Cost function to optimize the parameters $w^{(j)}$ and $b^{(j)}$ for each user $j$.

$$ J \begin{pmatrix} w^{(1)} & w^{(2)} & \cdots & w^{(n_u)} \\ b^{(1)} & b^{(2)} & \cdots & b^{(n_u)} \end{pmatrix}  = \frac{1}{2} \sum_{j=1}^{n_u} \sum_{i:r(i, j)=1} \left( w^{(j)} \cdot x^{(i)} + b^{(j)} - y^{(i, j)} \right)^2 + \frac{\lambda}{2} \sum_{j=1}^{n_u} \sum_{k=1}^n \left( w_k^{(j)} \right)^2 $$

2. Cost function to optimize the features $x^{(i)}$ for each movie $i$.

$$ J \begin{pmatrix} x^{(1)} & x^{(2)} & \cdots & x^{(n_m)} \end{pmatrix} = \frac{1}{2} \sum_{i=1}^{n_m} \sum_{j:r(i, j)=1} \left( w^{(j)} \cdot x^{(i)} - y^{(i, j)} \right)^2 + \frac{\lambda}{2} \sum_{i=1}^{n_m} \sum_{k=1}^n \left( x_k^{(i)} \right)^2 $$

If we combine these two cost functions, we get:

$$ J \begin{pmatrix} w^{(1)} & w^{(2)} & \cdots & w^{(n_u)} \\ b^{(1)} & b^{(2)} & \cdots & b^{(n_u)} \\ x^{(1)} & x^{(2)} & \cdots & x^{(n_m)} \end{pmatrix} = \frac{1}{2}  \sum_{(i, j):r(i, j)=1} \left( w^{(j)} \cdot x^{(i)} + b^{(j)} - y^{(i, j)} \right)^2 + \frac{\lambda}{2} \sum_{j=1}^{n_u} \sum_{k=1}^n \left( w_k^{(j)} \right)^2 + \frac{\lambda}{2} \sum_{i=1}^{n_m} \sum_{k=1}^n \left( x_k^{(i)} \right)^2 $$

where we have $2$ regularization terms, one for the parameters $w^{(j)}$ and other for the features $x^{(i)}$.

#### Optimization

We can optimize the cost function using **Gradient Descent**.

Earlier, we optimized the parameters $w^{(j)}$ and $b^{(j)}$ for each user $j$ separately. But now, we can optimize all the parameters together.

So, our gradient descent update rule for all the parameters can be given by:

$$ w_k^{(j)} := w_k^{(j)} - \alpha \frac{\partial J(w, b, x)}{\partial w_k^{(j)}}  $$

$$ b^{(j)} := b^{(j)} - \alpha \frac{\partial J(w, b, x)}{\partial b^{(j)}} $$

$$ x_k^{(i)} := x_k^{(i)} - \alpha \frac{\partial J(w, b, x)}{\partial x_k^{(i)}} $$

where $\alpha$ is the learning rate and now $x$ is also a parameter to be optimized.

#### Algorithm

1. Initialize $x^{(1)}, x^{(2)}, \ldots, x^{(n_m)}$ randomly
2. Use **Linear Regression** to learn $w^{(j)}, b^{(j)}, x^{(i)}$ for all $i, j$
3. Minimize $J(w, b, x)$ using **Gradient Descent** to learn $w^{(j)}, b^{(j)}, x^{(i)}$ for all $i, j$
4. Given a new user with parameters $w^{(new)}$, predict the ratings for the movies using the learned features $x^{(i)}$.

> The algorithm we derived is called **collaborative filtering**, and the name **collaborative filtering** refers to the sense that because multiple users have rated the same movie collaboratively, given you a sense of what this movie maybe like, that allows you to guess what are appropriate features for that movie, and this in turn allows you to predict how other users that haven't yet rated that same movie may decide to rate it in the future. ~ _Andrew Ng_

---
