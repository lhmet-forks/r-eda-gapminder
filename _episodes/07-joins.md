---
# Please do not edit this file directly; it is auto generated.
# Instead, please edit 07-joins.md in _episodes_rmd/
title: "Joining tables"
teaching: 30
exercises: 20
questions:
- "How to join different tables together?"
- "How to identify mis-matches between tables?"
objectives: 
- "Apply the `*_join()` family of functions to merge two tables with each other."
- "Use `anti_join()` to identify non-matches between two tables."
- "Recognise when to use each kind of joinining operation."
keypoints:
- "Use `full_join()`, `left_join()`, `right_join()` and `inner_join()` to merge two tables together."
- "Specify the column(s) to match between tables using the `by` option."
- "Use `anti_join()` to identify the rows from the first table which _do not_ have a match in the second table."
source: Rmd
---






In this lesson we're going to learn how to use functions from the `dplyr` package 
(part of `tidyverse`) to help us combine different tables together.

As usual when starting an analysis on a new script, let's start by loading the 
packages and reading the data. In this case, let's use the clean dataset that we 
created in the last exercise of a 
[previous episode]({{ page.root }}{% link _episodes/05-manipulate_observations_dplyr.md %}).


~~~
# load the package
library(tidyverse)

# Read the data, specifying how missing values are encoded
gapminder_clean <- read_csv("data/processed/gapminder1960to2010_socioeconomic_clean.csv", 
                            na = "")
~~~
{: .language-r}

If you haven't completed that exercise, here's how you can recreate the clean dataset:


~~~
gapminder_clean <- read_csv("data/raw/gapminder1960to2010_socioeconomic.csv", na = "") %>% 
  select(-country_id) %>% 
  mutate(population_total = population_male + population_female,
         main_religion = str_to_lower(str_squish(main_religion)),
         life_expectancy_male = ifelse(life_expectancy_male == -999, NA, life_expectancy_male),
         life_expectancy_female = as.numeric(life_expectancy_female)) %>% 
  filter(!is.na(income_groups))
~~~
{: .language-r}




## Joining tables

A common task in data analysis is to bring different datasets together, so that we 
can combine columns from two (or more) tables together. 

This can be achieved using the _join_ family of functions in `dplyr`. There are 
different types of _joins_, which can be represented by a series of Venn diagrams:

![joins](../fig/07-dplyr_joins.svg)

<!--
This can be used to explain joins in class. Maybe add to instructor guidelines.

Let's start by looking at some simple examples using in-built data that come with R.
Take the following two tables:


~~~
band_members
~~~
{: .language-r}



~~~
# A tibble: 3 x 2
  name  band   
  <chr> <chr>  
1 Mick  Stones 
2 John  Beatles
3 Paul  Beatles
~~~
{: .output}



~~~
band_instruments
~~~
{: .language-r}



~~~
# A tibble: 3 x 2
  name  plays 
  <chr> <chr> 
1 John  guitar
2 Paul  bass  
3 Keith guitar
~~~
{: .output}

As you can see, we have information about individuals and the music band they play in 
on one table, and the instrument they play on the other table. We can also see that 
not all artists are present on both tables. 

Let's _join_ these two tables and retain only the artists common to both:


~~~
inner_join(band_members, band_instruments, by = "name")
~~~
{: .language-r}



~~~
# A tibble: 2 x 3
  name  band    plays 
  <chr> <chr>   <chr> 
1 John  Beatles guitar
2 Paul  Beatles bass  
~~~
{: .output}

Note that the option `by = "name"` tells the _join_ function what is the column 
(or columns) that should be used to match the different entries in the two tables. 
-->


In the [apendix exercises]({{ page.root }}{% link _episodes/90-appendix-exercises.md %}) 
we've been exploring a dataset related to energy consumption in different counties. 
Let's see how we can _join_ these two datasets together, so that we have all information 
in a single table. 

First, let's start by reading the data into R 
(if you don't have these data <kbd>right-click</kbd> [this link](https://raw.githubusercontent.com/tavareshugo/r-eda-gapminder/gh-pages/_episodes_rmd/data/gapminder1990to2010_energy.tsv), choose "Save link as..." and save it on the `data/raw` folder of your project directory):


~~~
# it's a tab-separated file, so we use read_tsv
energy <- read_tsv("data/raw/gapminder1990to2010_energy.tsv", 
                   na = "")
~~~
{: .language-r}



A critical step when joining tables is to identify which columns are used to "match" 
the rows between them (in databases these are referred to as _key_ variables). 
In our case the columns `country`, `world_region` and `year` should, together, have a 
one-to-one match between our two tables. 

We can quickly check that not all countries from `gapminder_clean` are present in 
the `energy` table (here we use the base R method of accessing columns with `$`,
which returns a vector):


~~~
# the `all()` function checks whether all values of a logical vector are TRUE
all(gapminder_clean$country %in% energy$country)
~~~
{: .language-r}



~~~
[1] FALSE
~~~
{: .output}

And the same happens for year:


~~~
all(gapminder_clean$year %in% energy$year)
~~~
{: .language-r}



~~~
[1] FALSE
~~~
{: .output}

So, different types of _join_ operations (represented in the figure above) will 
result in different outcomes. 

Let's start by doing an _inner join_, which retains only the entries common to both tables:


~~~
inner_join(gapminder_clean, energy, 
           by = c("country", "world_region", "year"))
~~~
{: .language-r}



~~~
# A tibble: 3,946 x 23
   country world_region economic_organi… income_groups main_religion  year
   <chr>   <chr>        <chr>            <chr>         <chr>         <dbl>
 1 Afghan… south_asia   g77              low_income    muslim         1990
 2 Afghan… south_asia   g77              low_income    muslim         1991
 3 Afghan… south_asia   g77              low_income    muslim         1992
 4 Afghan… south_asia   g77              low_income    muslim         1993
 5 Afghan… south_asia   g77              low_income    muslim         1994
 6 Afghan… south_asia   g77              low_income    muslim         1995
 7 Afghan… south_asia   g77              low_income    muslim         1996
 8 Afghan… south_asia   g77              low_income    muslim         1997
 9 Afghan… south_asia   g77              low_income    muslim         1998
10 Afghan… south_asia   g77              low_income    muslim         1999
# … with 3,936 more rows, and 17 more variables: population_male <dbl>,
#   population_female <dbl>, income_per_person <dbl>, life_expectancy <dbl>,
#   life_expectancy_female <dbl>, life_expectancy_male <dbl>,
#   children_per_woman <dbl>, newborn_mortality <dbl>, child_mortality <dbl>,
#   school_years_men <dbl>, school_years_women <dbl>,
#   hdi_human_development_index <dbl>, population_total <dbl>,
#   country_id <chr>, yearly_co2_emissions <dbl>, energy_use_per_person <dbl>,
#   energy_production_per_person <chr>
~~~
{: .output}

The `by` option is used to tell the _join_ function which column(s) are used to "match" 
the rows of the two tables. We can see that the output has fewer rows than both of these 
tables, which makes sense given we're only keeping the "intersection" between the two. 
Schematically, this is what happened:

![inner_join](https://d33wubrfki0l68.cloudfront.net/3abea0b730526c3f053a3838953c35a0ccbe8980/7f29b/diagrams/join-inner.png)

The other types of _join_ functions work exactly the same as this one, the only thing 
that changes is what kind of output you get. Schematically:

![outer_joins](https://d33wubrfki0l68.cloudfront.net/9c12ca9e12ed26a7c5d2aa08e36d2ac4fb593f1e/79980/diagrams/join-outer.png)

One important thing to note is that when doing either of these types of _joins_, 
the functions will take care of filling in the table with missing values, `NA`, 
whenever a row in one table does not have a match in the other table. 


> ## Exercise
> 
> 1. Create a new table called `gapminder_all`, which includes all rows from both 
>    tables joined together by their common variable _keys_. Make a note of how many 
>    rows you have in the new table, and compare it with the original tables.
> 2. Use the `anti_join()` function to identify which rows in the `energy` dataset do 
>    NOT have a match in our original data. 
> 
> > ## Answer
> > 
> > **A1.**
> > 
> > Consulting the Venn diagrams at the top of the lesson, to retain _all_ data from 
> > both tables, we should use the `full_join()` function (corresponding to an union 
> > of the two tables):
> > 
> > 
> > ~~~
> > gapminder_all <- full_join(gapminder_clean, energy, by = c("country", "year", "world_region"))
> > 
> > # check how many rows
> > nrow(gapminder_all)
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > [1] 9814
> > ~~~
> > {: .output}
> > 
> > This table has more rows than either of the original tables. This must be because 
> > some data exists in one table but not the other.
> > 
> > **A2.**
> > 
> > Again, the Venn diagram at the top of the page illustrates the result of an `anti_join()`, 
> > which we can apply to investigate this issue:
> > 
> > 
> > ~~~
> > anti_join(energy, gapminder_clean, by = c("country", "year", "world_region"))
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > # A tibble: 22 x 7
> >    country_id country world_region  year yearly_co2_emis… energy_use_per_…
> >    <chr>      <chr>   <chr>        <dbl>            <dbl>            <dbl>
> >  1 bra        Brasil  america       1995         258347.              993.
> >  2 nru        Nauru   east_asia_p…  1990            125.               NA 
> >  3 nru        Nauru   east_asia_p…  1991            125.               NA 
> >  4 nru        Nauru   east_asia_p…  1992            121.               NA 
> >  5 nru        Nauru   east_asia_p…  1993            114.               NA 
> >  6 nru        Nauru   east_asia_p…  1994            110.               NA 
> >  7 nru        Nauru   east_asia_p…  1995            106.               NA 
> >  8 nru        Nauru   east_asia_p…  1996            103.               NA 
> >  9 nru        Nauru   east_asia_p…  1997            103.               NA 
> > 10 nru        Nauru   east_asia_p…  1998             99.0              NA 
> > # … with 12 more rows, and 1 more variable: energy_production_per_person <chr>
> > ~~~
> > {: .output}
> > 
> > We could further pipe (`%>%`) the output of the above operation to `distinct()` 
> > to check which countries were present in `energy` but missing in `gapminder_clean`:
> > 
> > 
> > ~~~
> > # take the energy data frame, and then...
> > energy %>% 
> >   # anti join it with gapminder_clean, and then...
> >   anti_join(gapminder_clean, by = c("country", "world_region", "year")) %>% 
> >   # get the distinct values of country
> >   distinct(country)
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > # A tibble: 2 x 1
> >   country
> >   <chr>  
> > 1 Brasil 
> > 2 Nauru  
> > ~~~
> > {: .output}
> > 
> > Note that if we dug a bit deeper into this, we would find that "Nauru" does not exist in
> > `gapminder_clean`, but "Brasil" does exist, but it's recorded as "Brazil", so it's a 
> > spelling inconsistency between the two datasets (in fact, the `energy` table has both 
> > "Brasil" and "Brazil", which we should correct if we were analysing these data further).
> {: .solution}
{: .challenge}


> ## Data tip: linking datasets
> 
> Combining datasets together is a very common task in data analysis, often referred 
> to as [data linkage](https://hdsr.mitpress.mit.edu/pub/8fm8lo1e). 
> 
> Although this task might seem easy at a first glance, it can be quite challenging 
> due to unforeseen properties of the data. For example: 
> 
> - which variables in one table have a correspondence in the other table? 
> (having the same column name doesn't necessarily mean they are the same)
> - are the values encoded similarly across datasets? 
> (for example, one dataset might encode _income groups_ as "low", "medium", "high" 
> and another as "1", "2", "3")
> - were the data recorded in a consistent manner? (for example, an individual's 
> _age_ might have been recorded as their date of birth or, due to confidentiality
> reasons, their age the last time they visited a clinic)
> 
> Thinking of these (and other) issues can be useful in order to avoid them when
> collecting your own data. 
> Also, when you share data, make sure to provide with _metadata_, explaining as 
> much as possible what each of your variables is and how it was measured. 
> 
> Finally, it's worth thinking that every observation of your data should have a 
> unique identifier (this is referred to as a "primary key"). 
> For example, in our data, a unique identifier could be created by the combination 
> of the `country` and `year` variables, as those two define our unit of study in 
> this case. 
{: .callout}
