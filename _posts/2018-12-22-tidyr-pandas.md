---
layout: post
title: Tidyr pandas
feature-img: "assets/img/pexels/pandas.jpg"
tags: [Python, R, tidyr, pandas, data wrangling]
---

As a data science enthusiast, and in my work, I often use R and Python which are two of the most exciting interpreted languages out there today.

In R, the [`tidyverse`](https://www.tidyverse.org/) packages collection has quickly become a reference for data science.
It offers data import, wrangling and visualisation packages, amongst which `dplyr`, `tidyr` and `ggplot2`.

In Python, the [`pandas`](http://pandas.pydata.org/) library implements the data frame structure and offers many functions for data import and wrangling, etc.

In this blog post, I will focus on the `tidyr` R package, and try to replicate the behavior of some of two of its most useful functions with `pandas` in Python.
I hope people with an R background and trying to learn python will find it useful.

In particular, I won't add to the debate asking which one of R or Python is better. For me, it is like asking whether French is better than Spanish or English better than German. It simply does not make sense.

The way I see it is that each has its advantages, and can be best suited to certain tasks. It can also be a matter of personal preference.

Ideally, one should learn and use both, especially that there are more and more ways to combine both.

For example, it's possible now to use Jupyter Notebooks with an R kernel. It is also possible to include python code blocks in Rmarkdown documents and seamlessly access python objects from R code blocks and vice versa thanks to the [`reticulate`](https://rstudio.github.io/reticulate/) package.

I might dedicate one or more blog posts to this in the future.

## Data frames

First, what's a data frame? It is basically a table that can contain columns with different data type. 

In R, the `data.frame` structure is natively available, with some extensions / enhancements available such as the [`tibble`](https://tibble.tidyverse.org/).

To instanciate a data frame, simply give the columns names and values:

```r
df <- data.frame(Key1 = c(1, 1, 2, 2, 3, 3, 4, 4, 4),
                 Key2 = c('a', 'b', 'a', 'b', 'a', 'b', 'a', 'b', 'c'),
                 Value = c(100, 150, 300, 200, 340, 450, 120, 470, 210))
df
```

| __Key1__ | __Key2__ | __Value__ |
|----------|----------|-----------|
|   1      | a   	  | 100       |
|   1      | b   	  | 150       |
|   2      | a   	  | 300       |
|   2      | b   	  | 200       |
|   3      | a   	  | 340       |
|   3      | b   	  | 450       |
|   4      | a   	  | 120       |
|   4      | b   	  | 470       |
|   4      | c   	  | 210       |

The pandas library implements the `DataFrame` data structure in Python.
Here is the same example as above, the instanciation is done using a (key, value) dictionary:

```python
import pandas as pd
df = pd.DataFrame({'Key1':  [1, 1, 2, 2, 3, 3, 4, 4, 4],
                   'Key2':  ['a', 'b', 'a', 'b', 'a', 'b', 'a', 'b', 'c'],
                   'Value': [100, 150, 300, 200, 340, 450, 120, 470, 210]})
```

## The pipe operator `%>%`

A quick digression to introduce the pipe operator that I will be using in the next sections.

One of the things I think make R an attractive language is the ability to define operators very easily. The pipe operator `%>%` provided by the `magrittr` package (also part of the `tidyverse`) is definitely one of the most useful.

What it does is take the result of the expression before it and give it as an input to the next one, which greatly simplifies some otherwise unreadable expressions by turning them into pipes.

For example, this expression with nested functions:
```r
library(dplyr)
summary(mutate(filter(df, Key2 == 'a'), Value = Value + 1)$Value)
```

becomes much clearer (we filter, then add one to the Value column, then take this column and display its summary):
```r
df %>% filter(Key2 == 'a') %>% mutate(Value = Value + 1) %>% .$Value %>% summary
```

## `spread` vs. `pivot_table`
As its name indicates, the `spread` function of the `tidyr` package "spreads" a single columns over many columns.
In other words, it turns a "long" data frame into a "wide" one.

Its arguments are intuitive:

- `key     `: name of the "key" column to be spread;
- `value   `: name of the "value" column from which the values will be taken;
- `fill    `: value to use to fill the non existing combinations (default is `NA`).

Let's apply it to our data frame (we don't need to load `magrittr` as it will be done automatically when we load `tidyr`):

```r
library(tidyr)
spread_df <- df %>% spread(Key1, Value, fill = 0)
spread_df
```

|__Key2__ |	__1__	|__2__	  |__3__	| __4__   |
|---------|---------|---------|---------|---------|
|  a  	  |100      |300      |340	    | 120     |
|  b   	  |150	    |200      |450	    | 470     |
|  c      |	0	    |0	      |  0	    | 210     |

We can achieve the same thing in `pandas` using the `pivot_table` function, taking similar arguments:

- `index     `: name of the "index" column (column not to be spread); 
- `columns   `: name of the "key" column to be spread;
- `values    `: name of the "value" column from which the values will be taken;
- `fill_value`: value to use to fill the non existing combinations (default is `NaN`).

```python
pivoted_df = pd.pivot_table(df, index = 'Key2', columns = 'Key1', values = 'Value', fill_value = 0)
```

|__Key1__ |	__1__	|__2__	  |__3__	| __4__   |
|__Key2__ |         |         |         |         |
|---------|---------|---------|---------|---------|
|__a__	  |100      |300      |340	    | 120     |
|__b__ 	  |150	    |200      |450	    | 470     |
|__c__    |	0	    |0	      |  0	    | 210     |

This results in an indexed data frame, with a columns axis named __Key1__, which is the name of the column that was "spread".

To get an identical result as R, we can simply get rid of the columns axis name, and reset the index in this way:

```python
pivoted_df = pd.pivot_table(df, index = 'Key2', columns = 'Key1', values = 'Value',
                            fill_value = 0).rename_axis(None, 1).reset_index()
```

## `gather` vs. `melt`
The `gather` function is the inverse operation, it "gathers" many columns into a single one, or turns a "wide" data frame into a "long" one.

Its arguments are:

- `key     `: name of the "key" column (the one that won't be "gathered");
- `value   `: name of the "value" column;
- `columns `: selection of columns to gather (or the columns not to with a minus sign).

For our example, key = __Key1__, value = __Value__ and we want to gather everything except the __Key2__ column:

```r
gathered_df <- spread_df %>% gather(Key1, Value, -Key2) 
gathered_df 
```

| __Key2__ | __Key1__ | __Value__ |
| a 	   | 1        | 100       |
| b 	   | 1        | 150       |
| c 	   | 1        | 0         |
| a 	   | 2        | 300       |
| b 	   | 2        | 200       |
| c 	   | 2        | 0         |
| a 	   | 3        | 340       |
| b 	   | 3        | 450       |
| c 	   | 3        | 0         |
| a 	   | 4        | 120       |
| b 	   | 4        | 470       |
| c 	   | 4        | 210       |

The result is identical to the original data frame, with some exceptions:

- There are extra rows where value is zero (resulting from the fill value of zero);
- The columns order is not the same

If we want to stay in the `tidyverse`, we can use the `filter` and `select` functions of the `dplyr` package to remove those extra columns, and change the column order:
```r
library(dplyr)
gathered_df %>% select(Key1, Key2, Value) %>% filter(Value != 0)
```

| __Key1__ | __Key2__ | __Value__ |
|----------|----------|-----------|
|   1      | a   	  | 100       |
|   1      | b   	  | 150       |
|   2      | a   	  | 300       |
|   2      | b   	  | 200       |
|   3      | a   	  | 340       |
|   3      | b   	  | 450       |
|   4      | a   	  | 120       |
|   4      | b   	  | 470       |
|   4      | c   	  | 210       |

In python, the same result can be achieved using the `melt` function that has the following arguments:

- `id_vars    `: the column(s) to use as identifier, i.e. the one(s) that won't be "gathered";
- `var_name   `: name to give to the variable column (the one that will take the values in the columns);
- `value_name `: name to give to the value column (the one that will take the content of the table).

For our example:

```python
melted_df = pd.melt(pivoted_df, id_vars = ['Key2'], var_name = 'Key1',  value_name = 'Value')
melted_df
```

Then to reorder the columns and filter out the values equal to zero:

```python
melted_df[['Key1', 'Key2', 'Value']].query("Value != 0")
```

| __Key1__ | __Key2__ | __Value__ |
|----------|----------|-----------|
|   1      | a   	  | 100       |
|   1      | b   	  | 150       |
|   2      | a   	  | 300       |
|   2      | b   	  | 200       |
|   3      | a   	  | 340       |
|   3      | b   	  | 450       |
|   4      | a   	  | 120       |
|   4      | b   	  | 470       |
|   4      | c   	  | 210       |

## Conclusion

Data manipulation and wrangling constitutes a very important part of the data scientists work.
The `tidyverse` in R and `pandas` in Python are formidable tools that make this part much easier.

In this blog post, we have seen how to replicate `tidyr`'s `gather` and `spread` functions using `pandas` `pivot_table` and `melt` functions.