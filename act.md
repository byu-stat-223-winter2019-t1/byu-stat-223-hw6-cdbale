act
================

Load
    libraries.

``` r
library(tidyverse)
```

    ## -- Attaching packages ---------------------------------------------------------------- tidyverse 1.2.1 --

    ## v ggplot2 3.1.0     v purrr   0.2.5
    ## v tibble  1.4.2     v dplyr   0.7.8
    ## v tidyr   0.8.2     v stringr 1.3.1
    ## v readr   1.3.1     v forcats 0.3.0

    ## -- Conflicts ------------------------------------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

-----

You are trying to decide between two ACT prep classes. Class 1 begins
with a general overview followed by an extensive number of practice
exams. Class 2 covers each topic in depth with practice questions
arranged by topic. Sample data for each class is available in the file
act.txt in your repository. Based on a permutation test with 10,000
replications, does Class 1 increase ACT scores on average? Answer the
same question for Class 2. Underneath your code, write a conclusion as
to which class (or, neither class) you would recommend. Justify your
answers.

-----

For our permutation test, our null hypothesis will be that there is no
difference in means for our pre and post class scores.

Read in data.

``` r
act_data <- read.table('act.txt', header = TRUE)
```

Calculate the average pre and post test scores for each class as well as
the difference in average pre and post scores for each class.

``` r
(class_stats <- act_data %>%
   group_by(class) %>%
   summarize(avg_pre = mean(pre), 
             avg_post = mean(post),
             difference = avg_post - avg_pre))
```

    ## # A tibble: 2 x 4
    ##   class  avg_pre avg_post difference
    ##   <fct>    <dbl>    <dbl>      <dbl>
    ## 1 class1    27.0     28.2      1.3  
    ## 2 class2    25.6     26.4      0.867

Store true differences in average pre and post test scores for each
class.

``` r
c1_diff <- class_stats %>%
  filter(class == 'class1') %>%
  select(difference) %>%
  pull()

c2_diff <- class_stats %>%
  filter(class == 'class2') %>%
  select(difference) %>%
  pull()
```

# Function

Create a function, ‘permuter’ which filters the ‘act\_data’ for the
specified class. The ‘nreps’ arguement has a default value of 10,000,
which is how many times we want to permute the data and calculate our
statistic of interest. After filtering for the specified class, the
function generates random draws between 0 and 1 from a uniform
distribution. Whether those draws are above or below 0.5 determines how
the values of ‘pre’ and ‘post’ test scores are permuted. Then, the
function calculates the average difference in test scores ‘pre’ and
‘post’ and returns that value.

``` r
permuter <- function(which_class, nreps = 10000) {
  
  new_data <- act_data %>%
    filter(class == which_class)
  
  mean_diffs <- map_df(1:nreps, function(x) {
    new_data %>%
      mutate(x = runif(n()),
             perm_pre = ifelse(x > 0.5, pre, post),
             perm_post = ifelse(x < 0.5, pre, post)) %>%
      summarize(mean_diff = mean(perm_post - perm_pre))
      
  })
  
  return(mean_diffs)
  
}
```

# Class 1

Calculate 10,000 estimates of the difference in mean score under the
null hypothesis for class one.

``` r
c1_perm_diffs <- permuter('class1')
```

Graph the estimates and include a verticle line to show the difference
for our class 1 sample.

``` r
c1_perm_diffs %>%
  ggplot(aes(x = mean_diff)) +
  geom_density() +
  geom_vline(xintercept = c1_diff, color = 'red') +
  labs(title = 'Null Distribution of Mean Score Diff. - Class 1',
       x = 'Mean Score Difference',
       y = 'Density')
```

![](act_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

Calculate the p-value of our estimate for class one.

``` r
c1_perm_diffs %>%
  summarize(P_Value = mean(mean_diff > c1_diff))
```

    ##   P_Value
    ## 1  0.0967

Based on this 10,000 replication permutation test, class one does not
increase scores on average.

# Class 2

Calculate 10,000 estimates of the difference in mean score under the
null hypothesis for class two.

``` r
c2_perm_diffs <- permuter('class2')
```

Graph the estimates and include a verticle line to show the difference
for our class 2 sample.

``` r
c2_perm_diffs %>%
  ggplot(aes(x = mean_diff)) +
  geom_density() +
  geom_vline(xintercept = c2_diff, color = 'red') +
  labs(title = 'Null Distribution of Mean Score Diff. - Class 2',
       x = 'Mean Score Difference',
       y = 'Density')
```

![](act_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

Calculate the p-value of our estimate for class two.

``` r
c2_perm_diffs %>%
  summarize(P_Value = mean(mean_diff > c2_diff))
```

    ##   P_Value
    ## 1  0.0318

Based on this 10,000 replication permutation test, class two does
increase scores on average.

-----

Based on this analysis, I would recommend that someone take class two.
The increase in average score that it produces is significant at the 5%
level. However, our point estimate of the increase in average score for
class 1 is larger than the point estimate for class 2. We only have 20
data points from class 1, so if we were to get more, we might be able to
show that the increase in score that class 1 produces is statistically
greater than zero. Until that point, I would recommend class 2.
