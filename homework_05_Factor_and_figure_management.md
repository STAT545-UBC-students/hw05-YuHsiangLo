Homework 05: Factor and figure management
================
Roger Yu-Hsiang Lo
2018-10-19

-   [Bring rectangular data in](#bring-rectangular-data-in)
-   [Part 1: Factor management](#part-1-factor-management)
    -   [Data sorting using `arrange()` on numeric variables](#data-sorting-using-arrange-on-numeric-variables)
    -   [Data sorting using `arrange()` on factor variables](#data-sorting-using-arrange-on-factor-variables)
    -   [Factor and level dropping](#factor-and-level-dropping)
    -   [Reordering levels of factor variables](#reordering-levels-of-factor-variables)

Bring rectangular data in
-------------------------

-   Load `Gapminder` and `tidyverse`:

``` r
library(gapminder)
library(tidyverse)
```

-   Some sanity check to make sure the `Gapminder` data was loaded properly:

``` r
head(gapminder) %>%
  knitr::kable()
```

| country     | continent |  year|  lifeExp|       pop|  gdpPercap|
|:------------|:----------|-----:|--------:|---------:|----------:|
| Afghanistan | Asia      |  1952|   28.801|   8425333|   779.4453|
| Afghanistan | Asia      |  1957|   30.332|   9240934|   820.8530|
| Afghanistan | Asia      |  1962|   31.997|  10267083|   853.1007|
| Afghanistan | Asia      |  1967|   34.020|  11537966|   836.1971|
| Afghanistan | Asia      |  1972|   36.088|  13079460|   739.9811|
| Afghanistan | Asia      |  1977|   38.438|  14880372|   786.1134|

Part 1: Factor management
-------------------------

We first check some general information about the `Gapminder` dataset before manipulating it.

``` r
gapminder %>%
  str()
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    1704 obs. of  6 variables:
    ##  $ country  : Factor w/ 142 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ continent: Factor w/ 5 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
    ##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
    ##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
    ##  $ pop      : int  8425333 9240934 10267083 11537966 13079460 14880372 12881816 13867957 16317921 22227415 ...
    ##  $ gdpPercap: num  779 821 853 836 740 ...

It is easy to see that the variables `country` and `continent` are both factors with 142 and 5 levels respectively.

### Data sorting using `arrange()` on numeric variables

We first explore the effects of the `arrange()` functions. In the following code, we attempt to sort the data by the mean life expectancy in 2007 for each continent:

``` r
gap_meanLifeExp_1 <- gapminder %>%
  filter(year == 2007) %>%
  group_by(continent) %>%
  summarize(meanLifeExp = mean(lifeExp)) %>%
  arrange(meanLifeExp)  # Arrange the row according to meanLifeExp

knitr::kable(gap_meanLifeExp_1)
```

| continent |  meanLifeExp|
|:----------|------------:|
| Africa    |     54.80604|
| Asia      |     70.72848|
| Americas  |     73.60812|
| Europe    |     77.64860|
| Oceania   |     80.71950|

From the resulting table, it seems that the `arrange()` function does have an effect on the output data frame, as the order of the continents is now following the increasing order of the mean life expectancy.

However, if we use the resulting data frame for a figure, it is obvious that the arragement of the bars follows the alphabetical order of the names of the continents, rather than the order of the mean life expectancy:

``` r
gap_meanLifeExp_1 %>%
  ggplot(aes(x = continent, y = meanLifeExp)) +
  geom_bar(stat = "identity")
```

<img src="homework_05_Factor_and_figure_management_files/figure-markdown_github/unnamed-chunk-5-1.png" style="display: block; margin: auto;" />

### Data sorting using `arrange()` on factor variables

Instead of sorting the data using a numeric variable, we now attempt to sort the data using a categorical variable (i.e., a factor). To demonstrate, let us sort the data by continents in the reverse alphabetical order. This can be done by using `arrange()` in combination with `desc()`, as shown in the following code:

``` r
gap_meanLifeExp_2 <- gapminder %>%
  filter(year == 2007) %>%
  group_by(continent) %>%
  summarize(meanLifeExp = mean(lifeExp)) %>%
  arrange(desc(continent))

knitr::kable(gap_meanLifeExp_2)
```

| continent |  meanLifeExp|
|:----------|------------:|
| Oceania   |     80.71950|
| Europe    |     77.64860|
| Asia      |     70.72848|
| Americas  |     73.60812|
| Africa    |     54.80604|

It appears that the data frame now is ordered as intented, but when we feed this data frame into a figure, the intented order is not reflected in the resulting figure:

``` r
gap_meanLifeExp_2 %>%
  ggplot(aes(x = continent, y = meanLifeExp)) +
  geom_bar(stat = "identity")
```

<img src="homework_05_Factor_and_figure_management_files/figure-markdown_github/unnamed-chunk-7-1.png" style="display: block; margin: auto;" />

### Factor and level dropping

From the `Gapminder`, we now drop the observations associated with Oceania in variable `continent`. Using the `str()` function, we can see that even after dropping these observations, the levels in the factor varialbes `continent` and `country` remain the same as in the original data set (5 and 142 levels respectively):

``` r
gap_no_oceania <- gapminder %>%
  filter(continent != "Oceania")
  
str(gap_no_oceania)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    1680 obs. of  6 variables:
    ##  $ country  : Factor w/ 142 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ continent: Factor w/ 5 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
    ##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
    ##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
    ##  $ pop      : int  8425333 9240934 10267083 11537966 13079460 14880372 12881816 13867957 16317921 22227415 ...
    ##  $ gdpPercap: num  779 821 853 836 740 ...

However, the total number of rows in the data frame is now different (1680 vs. 1704).

Now we explore the effects of two similar functions: `droplevels()` and `fct_drop()`. With `droplevels()`, the function goes through each factor variables and eliminate the levels that do not appears in any observations. In this case, `continent` now only has 4 levels (i.e., Oceania is now gone), and `country` now has 140 levels (i.e., Australia and New Zealand are dropped).

``` r
gap_no_oceania %>%
  droplevels() %>%
  str()
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    1680 obs. of  6 variables:
    ##  $ country  : Factor w/ 140 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ continent: Factor w/ 4 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
    ##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
    ##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
    ##  $ pop      : int  8425333 9240934 10267083 11537966 13079460 14880372 12881816 13867957 16317921 22227415 ...
    ##  $ gdpPercap: num  779 821 853 836 740 ...

With `fct_drop()`, however, we can specify the variable whose levels we want to drop. In the following example, we drop the used level in `continent`. Notice that `continent` now has only 4 levels, but `country` still has 142 levels.

``` r
gap_no_oceania %>%
  mutate(continent = fct_drop(continent)) %>%
  str()
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    1680 obs. of  6 variables:
    ##  $ country  : Factor w/ 142 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ continent: Factor w/ 4 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
    ##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
    ##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
    ##  $ pop      : int  8425333 9240934 10267083 11537966 13079460 14880372 12881816 13867957 16317921 22227415 ...
    ##  $ gdpPercap: num  779 821 853 836 740 ...

### Reordering levels of factor variables

We have seen that, if we want to present data in a tabular format, we can using `arrange()` to do this. However, if we want the same order to appear in a plot, we need to use `fct_reorder()` or `fct_reorder2()`. To illustrate the functions, we make a plot that shows the growth in life expectancy from 1952 to 2007 in Asian countries, with the order of the countries based on the amount of growth.

First, we create a data frame that contains a variable `growth` showing the difference in life expectancy between 2007 and 1952. Furthermore, we arrange the order of `country` with reference to `growth` using `fct_reorder`.

``` r
gap_growth <- gapminder %>%
  filter(year %in% c(1952, 2007), continent == "Asia") %>%
  # Rename 1952 to "earlier" and 2007 to "later" because we are going to spread year
  # and the resulting new columns with numerals as the names will cause trouble for following manipulations
  mutate(year = if_else(year == 1952, "earlier", "later")) %>%
  select(country, year, lifeExp) %>%
  spread(key = year, value = lifeExp) %>%
  mutate(growth = later - earlier, country = fct_reorder(country, growth))

knitr::kable(gap_growth)
```

| country            |  earlier|   later|  growth|
|:-------------------|--------:|-------:|-------:|
| Afghanistan        |   28.801|  43.828|  15.027|
| Bahrain            |   50.939|  75.635|  24.696|
| Bangladesh         |   37.484|  64.062|  26.578|
| Cambodia           |   39.417|  59.723|  20.306|
| China              |   44.000|  72.961|  28.961|
| Hong Kong, China   |   60.960|  82.208|  21.248|
| India              |   37.373|  64.698|  27.325|
| Indonesia          |   37.468|  70.650|  33.182|
| Iran               |   44.869|  70.964|  26.095|
| Iraq               |   45.320|  59.545|  14.225|
| Israel             |   65.390|  80.745|  15.355|
| Japan              |   63.030|  82.603|  19.573|
| Jordan             |   43.158|  72.535|  29.377|
| Korea, Dem. Rep.   |   50.056|  67.297|  17.241|
| Korea, Rep.        |   47.453|  78.623|  31.170|
| Kuwait             |   55.565|  77.588|  22.023|
| Lebanon            |   55.928|  71.993|  16.065|
| Malaysia           |   48.463|  74.241|  25.778|
| Mongolia           |   42.244|  66.803|  24.559|
| Myanmar            |   36.319|  62.069|  25.750|
| Nepal              |   36.157|  63.785|  27.628|
| Oman               |   37.578|  75.640|  38.062|
| Pakistan           |   43.436|  65.483|  22.047|
| Philippines        |   47.752|  71.688|  23.936|
| Saudi Arabia       |   39.875|  72.777|  32.902|
| Singapore          |   60.396|  79.972|  19.576|
| Sri Lanka          |   57.593|  72.396|  14.803|
| Syria              |   45.883|  74.143|  28.260|
| Taiwan             |   58.500|  78.400|  19.900|
| Thailand           |   50.848|  70.616|  19.768|
| Vietnam            |   40.412|  74.249|  33.837|
| West Bank and Gaza |   43.160|  73.422|  30.262|
| Yemen, Rep.        |   32.548|  62.698|  30.150|

Notice that this time the data frame appears to be arranged differently than expected: it is still ordered by the variable `country` rather than by `growth`. However, when we use the same data frame to generate a plot, the bars are arranged as intended:

``` r
gap_growth %>%
  ggplot(aes(y = growth, x = country)) +
  theme_bw() +
  geom_col() +
  geom_point(aes(y = later, x = country, color = "Life exp. in 2007")) +  # We need to add points manually because of the data frame structure
  geom_point(aes(y = earlier, x = country, color = "Life exp. in 1952")) +
  labs(title = "Growth in life expectancy in Asian countries", x = "Country", y = "Age (year)") +
  scale_color_manual(name = "", values = c("Life exp. in 2007" = "#00BFC4", "Life exp. in 1952" = "#F8766D")) +  # Manually add legend
  coord_flip() +
  theme(legend.position = "bottom")
```

<img src="homework_05_Factor_and_figure_management_files/figure-markdown_github/unnamed-chunk-12-1.png" style="display: block; margin: auto;" />

From the plot, we can immediately see that Oman is the country that has the most significant growth in life expectancy, while Iraq saw the smallest growth among Asian countries.