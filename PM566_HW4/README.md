HW04
================
Yating Zeng
2022-11-18

# 1.HPC

## Problem 1: Make sure your code is nice

Rewrite the following R functions to make them faster. It is OK (and
recommended) to take a look at Stackoverflow and Google

``` r
# Total row sums
fun1 <- function(mat) {
  n <- nrow(mat)
  ans <- double(n) 
  for (i in 1:n) {
    ans[i] <- sum(mat[i, ])
  }
  ans
}


fun1alt <- function(mat) {
  # MY CODE HERE
  library(matrixStats)
  rowSums(mat)
}

# Cumulative sum by row
fun2 <- function(mat) {
  n <- nrow(mat)
  k <- ncol(mat)
  ans <- mat
  for (i in 1:n) {
    for (j in 2:k) {
      ans[i,j] <- mat[i, j] + ans[i, j - 1]
    }
  }
  ans
}

fun2alt <- function(mat) {
  # MY CODE HERE
  library(matrixStats)
  rowCumsums(mat)
}


# Use the data with this code
set.seed(2315)
dat <- matrix(rnorm(200 * 100), nrow = 200)

# Test for the first
microbenchmark::microbenchmark(
  fun1(dat),
  fun1alt(dat), check = "equivalent"
)
```

    ## Warning: package 'matrixStats' was built under R version 4.1.2

    ## Unit: microseconds
    ##          expr     min       lq      mean   median       uq      max neval cld
    ##     fun1(dat) 311.775 404.4740 454.67329 437.1165 473.7855  786.548   100   b
    ##  fun1alt(dat)  56.862  63.8805  98.98776  72.7365  81.0435 2348.432   100  a

``` r
# Test for the second
microbenchmark::microbenchmark(
  fun2(dat),
  fun2alt(dat), check = "equivalent"
)
```

    ## Unit: microseconds
    ##          expr      min       lq      mean   median        uq      max neval cld
    ##     fun2(dat) 2024.777 2088.581 2184.0297 2116.657 2221.4340 3142.152   100   b
    ##  fun2alt(dat)  125.943  138.885  188.8373  155.960  188.9085 2434.783   100  a

``` r
#The last argument, check = “equivalent”, is included to make sure that the functions return the same result.
```

Based on the result above, we could notice that alternative function 1&2
are both lessen the time used than original function 1&2.

## Problem 2: Make things run faster with parallel computing

The following function allows simulating PI

``` r
sim_pi <- function(n = 1000, i = NULL) {
  p <- matrix(runif(n*2), ncol = 2)
  mean(rowSums(p^2) < 1) * 4
}

# Here is an example of the run
set.seed(156)
sim_pi(1000) # 3.132
```

    ## [1] 3.132

In order to get accurate estimates, we can run this function multiple
times, with the following code:

``` r
# This runs the simulation a 4,000 times, each with 10,000 points
set.seed(1231)
system.time({
  ans <- unlist(lapply(1:4000, sim_pi, n = 10000))
  print(mean(ans))
})
```

    ## [1] 3.14124

    ##    user  system elapsed 
    ##   2.972   0.781   3.767

Rewrite the previous code using parLapply() to make it run faster. Make
sure you set the seed using clusterSetRNGStream():

``` r
# MY CODE HERE
system.time({
  # MY CODE HERE
  ans <- # YOUR CODE HERE
  print(mean(ans))
  # YOUR CODE HERE
})
```

    ## [1] 3.14124

    ##    user  system elapsed 
    ##       0       0       0

``` r
library(parallel)
cl <- makePSOCKcluster(4)   
clusterSetRNGStream(cl, 1231) # Equivalent to `set.seed(123)`
clusterExport(cl, "sim_pi")
system.time({
  ans <- unlist(parLapply(cl, 1:4000, sim_pi, n = 10000))
  print(mean(ans))
})
```

    ## [1] 3.141578

    ##    user  system elapsed 
    ##   0.004   0.000   1.071

``` r
stopCluster(cl)
```

Based on the results, the time used was decreased significantly by
parallel with a Socket Cluster.

# SQL

Setup a temporary database by running the following chunk

``` r
# install.packages(c("RSQLite", "DBI"))

library(RSQLite)
```

    ## Warning: package 'RSQLite' was built under R version 4.1.2

``` r
library(DBI)
```

    ## Warning: package 'DBI' was built under R version 4.1.2

``` r
# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")

# Download tables
film <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film.csv")
film_category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film_category.csv")
category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/category.csv")

# Copy data.frames to database
dbWriteTable(con, "film", film)
dbWriteTable(con, "film_category", film_category)
dbWriteTable(con, "category", category)
```

When you write a new chunk, remember to replace the r with sql,
connection=con. Some of these questions will require you to use an inner
join. Read more about them here
<https://www.w3schools.com/sql/sql_join_inner.asp>

## Question 1

How many many movies is there available in each rating category.

``` sql
  SELECT  rating,
    COUNT(*) AS count
  FROM film
  GROUP BY rating
```

| rating | count |
|:-------|------:|
| G      |   180 |
| NC-17  |   210 |
| PG     |   194 |
| PG-13  |   223 |
| R      |   195 |

5 records

As the results shown, teh number of movies for AG is 180; for NC-17 is
210; for PG is 194; for PG-13 is 223; for R is 195.

## Question 2

What is the average replacement cost and rental rate for each rating
category.

``` sql
  SELECT  rating,
    AVG(replacement_cost) AS avg_replacement_cost, 
    AVG(rental_rate) AS avg_rental_rate
  FROM film
  GROUP BY rating
```

| rating | avg_replacement_cost | avg_rental_rate |
|:-------|---------------------:|----------------:|
| G      |             20.12333 |        2.912222 |
| NC-17  |             20.13762 |        2.970952 |
| PG     |             18.95907 |        3.051856 |
| PG-13  |             20.40256 |        3.034843 |
| R      |             20.23103 |        2.938718 |

5 records

The average replacement cost and rental rate for each rating category
are shown above, with ave_replacement_cost represents the average
replacement cost and avg_rental_rate represents the average rental rate.

## Question 3

Use table film_category together with film to find the how many films
there are with each category ID

``` sql
  SELECT  category_id,
    COUNT(*) AS count
  FROM film AS a INNER JOIN film_category AS b
  ON a.film_id = b.film_id
  GROUP BY category_id
```

| category_id | count |
|:------------|------:|
| 1           |    64 |
| 2           |    66 |
| 3           |    60 |
| 4           |    57 |
| 5           |    58 |
| 6           |    68 |
| 7           |    62 |
| 8           |    69 |
| 9           |    73 |
| 10          |    61 |

Displaying records 1 - 10

For each category ID, the number of films are listed on the second
column above.

## Question 4

Incorporate table category into the answer to the previous question to
find the name of the most popular category.

``` sql
  SELECT  category_id, 
    COUNT(category_id) AS count
  FROM film AS a INNER JOIN film_category AS b 
  ON a.film_id = b.film_id
  GROUP BY category_id
  ORDER BY count DESC
```

| category_id | count |
|------------:|------:|
|          15 |    74 |
|           9 |    73 |
|           8 |    69 |
|           6 |    68 |
|           2 |    66 |
|           1 |    64 |
|          13 |    63 |
|           7 |    62 |
|          14 |    61 |
|          10 |    61 |

Displaying records 1 - 10

``` sql
  SELECT  b.category_id, b.name
  FROM film_category AS a INNER JOIN category AS b 
  ON a.category_id = b.category_id
  WHERE b.category_id = 15
```

| category_id | name   |
|------------:|:-------|
|          15 | Sports |
|          15 | Sports |
|          15 | Sports |
|          15 | Sports |
|          15 | Sports |
|          15 | Sports |
|          15 | Sports |
|          15 | Sports |
|          15 | Sports |
|          15 | Sports |

Displaying records 1 - 10

Based on the results above, we know that the id of the most popular
category is 15, whose name is “Sports”. Thus, “Sports” is the most
popular category in this case.
