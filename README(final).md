# Final Project Submission


* Student name: Chris Lewis
* Student pace: part time - online
* Scheduled project review date/time: 6/9/2020
* Instructor name: James Irving
* Blog post URL: http://lewis34cs.github.io/the_mysteries_of_the_fabled_violin_plot


# Introduction

- **Microsoft wants to know what type of films are currently doing the best at the box office?**
- We must then translate those findings into actionable insights that the CEO can use when deciding what type of films they should be creating.

- **Questions we will be exploring with the data:**
    - **Question 1**: Is there an association between production budget and Worldwide Gross for Movies in the Past Ten Years?
    
    - **Question 2**: Which genres (from U.S. movies) have been performing the best in terms of average total net profit since 2010?
    
    - **Question 3**: Who are the top 25 U.S. directors that have the highest average net profit within the past 20 years? 

# Import


#### Importing libraries:


```python
# Your code here - remember to use markdown cells for comments as well!
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as mtick
import sqlite3
import seaborn as sns
from pandasql import sqldf
from scipy.stats import sem
```

# Reading files and placing them into separate dataframes

#### Using pandas read_csv() function to put each file from the zipped folder into a Dataframe.

#### We are taking the first file (bom.movie_gross.csv.gz) and putting it  into the variable 'movie_gross_df' as a dataframe object.


```python
movie_gross_df = pd.read_csv('zippedData/bom.movie_gross.csv.gz')
movie_gross_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>studio</th>
      <th>domestic_gross</th>
      <th>foreign_gross</th>
      <th>year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Toy Story 3</td>
      <td>BV</td>
      <td>415000000.0</td>
      <td>652000000</td>
      <td>2010</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Alice in Wonderland (2010)</td>
      <td>BV</td>
      <td>334200000.0</td>
      <td>691300000</td>
      <td>2010</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Harry Potter and the Deathly Hallows Part 1</td>
      <td>WB</td>
      <td>296000000.0</td>
      <td>664300000</td>
      <td>2010</td>
    </tr>
  </tbody>
</table>
</div>




```python
movie_gross_df = movie_gross_df.rename(columns={'title':'primary_title'})
```


```python
movie_gross_df.dtypes
```




    primary_title      object
    studio             object
    domestic_gross    float64
    foreign_gross      object
    year                int64
    dtype: object




```python
len(movie_gross_df)
```




    3387



#### Within the dtypes of the movie_gross_df, we notice that the foreign_gross column contains object values - most likely string values. So let's change those to floats. There are also commas within the values, so we will need to remove those with the .replace() method.


```python
type(movie_gross_df['foreign_gross'][0])
```




    str




```python
movie_gross_df['foreign_gross'] = movie_gross_df['foreign_gross'].str.replace(',', '')
movie_gross_df['foreign_gross'] = movie_gross_df['foreign_gross'].astype(float)
movie_gross_df.dtypes
```




    primary_title      object
    studio             object
    domestic_gross    float64
    foreign_gross     float64
    year                int64
    dtype: object



#### Our next file (imdb.name.basics.csv.gz) is placed into the 'imdb_name_df' variable. This dataframe doesn't seem like it will be very useful to begin with, so let's save cleaning it for later if we need to come back to it.


```python
imdb_name_df = pd.read_csv('zippedData/imdb.name.basics.csv.gz')
imdb_name_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>nconst</th>
      <th>primary_name</th>
      <th>birth_year</th>
      <th>death_year</th>
      <th>primary_profession</th>
      <th>known_for_titles</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>nm0061671</td>
      <td>Mary Ellen Bauder</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>miscellaneous,production_manager,producer</td>
      <td>tt0837562,tt2398241,tt0844471,tt0118553</td>
    </tr>
    <tr>
      <td>1</td>
      <td>nm0061865</td>
      <td>Joseph Bauer</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>composer,music_department,sound_department</td>
      <td>tt0896534,tt6791238,tt0287072,tt1682940</td>
    </tr>
    <tr>
      <td>2</td>
      <td>nm0062070</td>
      <td>Bruce Baum</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>miscellaneous,actor,writer</td>
      <td>tt1470654,tt0363631,tt0104030,tt0102898</td>
    </tr>
  </tbody>
</table>
</div>




```python
imdb_name_df.dtypes
```




    nconst                 object
    primary_name           object
    birth_year            float64
    death_year            float64
    primary_profession     object
    known_for_titles       object
    dtype: object



#### Our next file (imdb.title.akas.csv.gz) is put into the 'imdb_title_df' variable. Let's inspect the contents.


```python
imdb_title_df = pd.read_csv('zippedData/imdb.title.akas.csv.gz')
imdb_title_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title_id</th>
      <th>ordering</th>
      <th>title</th>
      <th>region</th>
      <th>language</th>
      <th>types</th>
      <th>attributes</th>
      <th>is_original_title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt0369610</td>
      <td>10</td>
      <td>Джурасик свят</td>
      <td>BG</td>
      <td>bg</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt0369610</td>
      <td>11</td>
      <td>Jurashikku warudo</td>
      <td>JP</td>
      <td>NaN</td>
      <td>imdbDisplay</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt0369610</td>
      <td>12</td>
      <td>Jurassic World: O Mundo dos Dinossauros</td>
      <td>BR</td>
      <td>NaN</td>
      <td>imdbDisplay</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
imdb_title_df.dtypes
```




    title_id              object
    ordering               int64
    title                 object
    region                object
    language              object
    types                 object
    attributes            object
    is_original_title    float64
    dtype: object



#### Setting the column 'title_id' to 'tconst' to match some of the other dataframes similar columns to make it easier to join later on.


```python
imdb_title_df = imdb_title_df.rename(columns={'title_id': 'tconst'})
imdb_title_df.head(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>ordering</th>
      <th>title</th>
      <th>region</th>
      <th>language</th>
      <th>types</th>
      <th>attributes</th>
      <th>is_original_title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt0369610</td>
      <td>10</td>
      <td>Джурасик свят</td>
      <td>BG</td>
      <td>bg</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(imdb_title_df)
```




    331703




```python
imdb_title_df['region'].value_counts()
```




    US      51490
    XWW     18467
    RU      13817
    DE      11634
    FR      10990
            ...  
    TO          1
    AI          1
    CSHH        1
    LS          1
    MQ          1
    Name: region, Length: 213, dtype: int64



#### Our next file (imdb.title.basics.csv.gz) is put into the 'imdb_title_basics' variable. Let's inspect the contents.


```python
imdb_title_basics = pd.read_csv('zippeddata/imdb.title.basics.csv.gz')
imdb_title_basics.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt0063540</td>
      <td>Sunghursh</td>
      <td>Sunghursh</td>
      <td>2013</td>
      <td>175.0</td>
      <td>Action,Crime,Drama</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt0066787</td>
      <td>One Day Before the Rainy Season</td>
      <td>Ashad Ka Ek Din</td>
      <td>2019</td>
      <td>114.0</td>
      <td>Biography,Drama</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt0069049</td>
      <td>The Other Side of the Wind</td>
      <td>The Other Side of the Wind</td>
      <td>2018</td>
      <td>122.0</td>
      <td>Drama</td>
    </tr>
  </tbody>
</table>
</div>




```python
imdb_title_basics.dtypes
```




    tconst              object
    primary_title       object
    original_title      object
    start_year           int64
    runtime_minutes    float64
    genres              object
    dtype: object




```python
len(imdb_title_basics)
```




    146144



#### Our next file (imdb.title.crew.csv.gz) is put into the 'imdb_crew_df' variable. Let's inspect the contents.


```python
imdb_title_crew_df = pd.read_csv('zippedData/imdb.title.crew.csv.gz')
imdb_title_crew_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>directors</th>
      <th>writers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt0285252</td>
      <td>nm0899854</td>
      <td>nm0899854</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt0438973</td>
      <td>NaN</td>
      <td>nm0175726,nm1802864</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt0462036</td>
      <td>nm1940585</td>
      <td>nm1940585</td>
    </tr>
  </tbody>
</table>
</div>




```python
imdb_title_crew_df.dtypes
```




    tconst       object
    directors    object
    writers      object
    dtype: object




```python
len(imdb_title_crew_df)
```




    146144



#### Our next file (imdb.title.principals.csv.gz) is put into the 'imdb_title_princ_df' variable. Let's inspect the contents.


```python
imdb_title_princ_df = pd.read_csv('zippedData/imdb.title.principals.csv.gz')
imdb_title_princ_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>ordering</th>
      <th>nconst</th>
      <th>category</th>
      <th>job</th>
      <th>characters</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt0111414</td>
      <td>1</td>
      <td>nm0246005</td>
      <td>actor</td>
      <td>NaN</td>
      <td>["The Man"]</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt0111414</td>
      <td>2</td>
      <td>nm0398271</td>
      <td>director</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt0111414</td>
      <td>3</td>
      <td>nm3739909</td>
      <td>producer</td>
      <td>producer</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
imdb_title_princ_df.dtypes
```




    tconst        object
    ordering       int64
    nconst        object
    category      object
    job           object
    characters    object
    dtype: object




```python
len(imdb_title_princ_df)
```




    1028186



#### Our next file (imdb.title.ratings.csv.gz) is put into the 'imdb_title_rating_df' variable. Let's inspect the contents.


```python
imdb_title_rating_df = pd.read_csv('zippedData/imdb.title.ratings.csv.gz')
imdb_title_rating_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>averagerating</th>
      <th>numvotes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt10356526</td>
      <td>8.3</td>
      <td>31</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt10384606</td>
      <td>8.9</td>
      <td>559</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt1042974</td>
      <td>6.4</td>
      <td>20</td>
    </tr>
  </tbody>
</table>
</div>




```python
imdb_title_rating_df.dtypes
```




    tconst            object
    averagerating    float64
    numvotes           int64
    dtype: object




```python
len(imdb_title_rating_df)
```




    73856



#### Our next file (rt.movie_info.tsv.gz) will be put into the 'rt_movie_info_df' variable. However, the data within this file are separated by tabs rather than commas, so we will make sure to set the delimiter equal to '\t' within the read_csv method.


```python
rt_movie_info_df = pd.read_csv('zippedData/rt.movie_info.tsv.gz', '\t')
rt_movie_info_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>synopsis</th>
      <th>rating</th>
      <th>genre</th>
      <th>director</th>
      <th>writer</th>
      <th>theater_date</th>
      <th>dvd_date</th>
      <th>currency</th>
      <th>box_office</th>
      <th>runtime</th>
      <th>studio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>This gritty, fast-paced, and innovative police...</td>
      <td>R</td>
      <td>Action and Adventure|Classics|Drama</td>
      <td>William Friedkin</td>
      <td>Ernest Tidyman</td>
      <td>Oct 9, 1971</td>
      <td>Sep 25, 2001</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>104 minutes</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>1</td>
      <td>3</td>
      <td>New York City, not-too-distant-future: Eric Pa...</td>
      <td>R</td>
      <td>Drama|Science Fiction and Fantasy</td>
      <td>David Cronenberg</td>
      <td>David Cronenberg|Don DeLillo</td>
      <td>Aug 17, 2012</td>
      <td>Jan 1, 2013</td>
      <td>$</td>
      <td>600,000</td>
      <td>108 minutes</td>
      <td>Entertainment One</td>
    </tr>
    <tr>
      <td>2</td>
      <td>5</td>
      <td>Illeana Douglas delivers a superb performance ...</td>
      <td>R</td>
      <td>Drama|Musical and Performing Arts</td>
      <td>Allison Anders</td>
      <td>Allison Anders</td>
      <td>Sep 13, 1996</td>
      <td>Apr 18, 2000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>116 minutes</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
rt_movie_info_df.dtypes
```




    id               int64
    synopsis        object
    rating          object
    genre           object
    director        object
    writer          object
    theater_date    object
    dvd_date        object
    currency        object
    box_office      object
    runtime         object
    studio          object
    dtype: object




```python
len(rt_movie_info_df)
```




    1560



#### Our next file (rt.reviews.tsv.gz) is placed into the 'rt_reviews_df' variable. This file also a tsv file, however we will need to also set it's encoding to 'latin1' since it is not utf-8.


```python
rt_reviews_df = pd.read_csv('zippedData/rt.reviews.tsv.gz', '\t', encoding='latin1')
rt_reviews_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>review</th>
      <th>rating</th>
      <th>fresh</th>
      <th>critic</th>
      <th>top_critic</th>
      <th>publisher</th>
      <th>date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>3</td>
      <td>A distinctly gallows take on contemporary fina...</td>
      <td>3/5</td>
      <td>fresh</td>
      <td>PJ Nabarro</td>
      <td>0</td>
      <td>Patrick Nabarro</td>
      <td>November 10, 2018</td>
    </tr>
    <tr>
      <td>1</td>
      <td>3</td>
      <td>It's an allegory in search of a meaning that n...</td>
      <td>NaN</td>
      <td>rotten</td>
      <td>Annalee Newitz</td>
      <td>0</td>
      <td>io9.com</td>
      <td>May 23, 2018</td>
    </tr>
    <tr>
      <td>2</td>
      <td>3</td>
      <td>... life lived in a bubble in financial dealin...</td>
      <td>NaN</td>
      <td>fresh</td>
      <td>Sean Axmaker</td>
      <td>0</td>
      <td>Stream on Demand</td>
      <td>January 4, 2018</td>
    </tr>
  </tbody>
</table>
</div>




```python
rt_reviews_df.dtypes
```




    id             int64
    review        object
    rating        object
    fresh         object
    critic        object
    top_critic     int64
    publisher     object
    date          object
    dtype: object




```python
len(rt_reviews_df)
```




    54432



#### Our next file (tmbd.movies.csv.gz) is placed into the 'tmbd_movies_df' variable. It has an extra column we don't need, so we are going to drop 'Unnamed: 0'.


```python
tmbd_movies_df = pd.read_csv('zippedData/tmdb.movies.csv.gz')

tmbd_movies_df = tmbd_movies_df.drop(columns='Unnamed: 0', axis=1)
tmbd_movies_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>genre_ids</th>
      <th>id</th>
      <th>original_language</th>
      <th>original_title</th>
      <th>popularity</th>
      <th>release_date</th>
      <th>title</th>
      <th>vote_average</th>
      <th>vote_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>[12, 14, 10751]</td>
      <td>12444</td>
      <td>en</td>
      <td>Harry Potter and the Deathly Hallows: Part 1</td>
      <td>33.533</td>
      <td>2010-11-19</td>
      <td>Harry Potter and the Deathly Hallows: Part 1</td>
      <td>7.7</td>
      <td>10788</td>
    </tr>
    <tr>
      <td>1</td>
      <td>[14, 12, 16, 10751]</td>
      <td>10191</td>
      <td>en</td>
      <td>How to Train Your Dragon</td>
      <td>28.734</td>
      <td>2010-03-26</td>
      <td>How to Train Your Dragon</td>
      <td>7.7</td>
      <td>7610</td>
    </tr>
    <tr>
      <td>2</td>
      <td>[12, 28, 878]</td>
      <td>10138</td>
      <td>en</td>
      <td>Iron Man 2</td>
      <td>28.515</td>
      <td>2010-05-07</td>
      <td>Iron Man 2</td>
      <td>6.8</td>
      <td>12368</td>
    </tr>
  </tbody>
</table>
</div>




```python
tmbd_movies_df.dtypes
```




    genre_ids             object
    id                     int64
    original_language     object
    original_title        object
    popularity           float64
    release_date          object
    title                 object
    vote_average         float64
    vote_count             int64
    dtype: object




```python
len(tmbd_movies_df)
```




    26517



#### Our final file (tn.movie_budgets.csv.gz) will be placed in the 'tn_movie_budget_df' variable.


```python
tn_movie_budget = pd.read_csv('zippedData/tn.movie_budgets.csv.gz')
tn_movie_budget.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>$425,000,000</td>
      <td>$760,507,625</td>
      <td>$2,776,345,279</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2</td>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>$410,600,000</td>
      <td>$241,063,875</td>
      <td>$1,045,663,875</td>
    </tr>
    <tr>
      <td>2</td>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>$350,000,000</td>
      <td>$42,762,350</td>
      <td>$149,762,350</td>
    </tr>
  </tbody>
</table>
</div>




```python
tn_movie_budget.dtypes
```




    id                    int64
    release_date         object
    movie                object
    production_budget    object
    domestic_gross       object
    worldwide_gross      object
    dtype: object



#### Here we see within the tn_movie_budget_df, production_budget, domestic_gross, and worldwide_gross all contain string values that have dollar signs and commas within them. We are going to replace the dollar signs and commas and turn these string values into float types.


```python
tn_movie_budget['production_budget'] = tn_movie_budget['production_budget'].str.replace('$', '')
tn_movie_budget['production_budget'] = tn_movie_budget['production_budget'].str.replace(',', '')
tn_movie_budget['domestic_gross'] = tn_movie_budget['domestic_gross'].str.replace('$', '')
tn_movie_budget['domestic_gross'] = tn_movie_budget['domestic_gross'].str.replace(',', '')
tn_movie_budget['worldwide_gross'] = tn_movie_budget['worldwide_gross'].str.replace('$', '')
tn_movie_budget['worldwide_gross'] = tn_movie_budget['worldwide_gross'].str.replace(',', '')
```


```python
tn_movie_budget['production_budget'] = tn_movie_budget['production_budget'].astype(float)
tn_movie_budget['domestic_gross'] = tn_movie_budget['domestic_gross'].astype(float)
tn_movie_budget['worldwide_gross'] = tn_movie_budget['worldwide_gross'].astype(float)
```


```python
tn_movie_budget.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>425000000.0</td>
      <td>760507625.0</td>
      <td>2.776345e+09</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2</td>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <td>2</td>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(tn_movie_budget)
```




    5782



#### We are going to rename the column 'movie' to 'primary_title', so we can use that column to more easily join to other dataframes.


```python
tn_movie_budget = tn_movie_budget.rename(columns={'movie': 'primary_title'})
```


```python
tn_movie_budget.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>425000000.0</td>
      <td>760507625.0</td>
      <td>2.776345e+09</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2</td>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <td>2</td>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
  </tbody>
</table>
</div>



## Putting Dataframe names into a list:

#### We make a list so we can go through each of the dataframes by using the functions we made below to see the info more clearly.


```python
df_list = [movie_gross_df, imdb_name_df, imdb_title_df, imdb_title_basics, imdb_title_crew_df, imdb_title_princ_df,
           imdb_title_rating_df, rt_movie_info_df, rt_reviews_df, tmbd_movies_df, tn_movie_budget]
len(df_list)
```




    11



# Defining functions: 
#### get_df_name() will return only the DataFrame's variable name


```python
def get_df_name(df):
    """ Retrieved from: https://stackoverflow.com/questions/31727333/get-the-name-of-a-pandas-dataframe """
    name =[x for x in globals() if globals()[x] is df][0]
    return name
```

#### get_df_col() will return a list containing the names of each column within that DataFrame.


```python
def get_df_col(df):
    col_list = []
    for col in df.columns:
        col_list.append(col)
    return col_list
```

#### perc_null() will return the percent of null values present in each column of each DataFrame.


```python
def perc_null(df):
    return df.isna().sum().divide(len(df))*100
```

#### pysqldf() function will return a dataframe with certain columns (dictated by what we SELECT) 


```python
def pysqldf(q):
    return sqldf(q, globals())
```

## Using a for loop to see info on all dataframes

### We'll run each dataframe contained within the df_list variable and use the above functions to return each of the DataFrames' name along with their column names, as well as the percent of null values within each column in a nice order so we can view and compare their columns more easily.


```python
for df in df_list:
    print(f"Name: {get_df_name(df)}\ncolumns: {get_df_col(df)}\ndf length:{len(df)}\nPercent Null:\n{perc_null(df)}\n\n")
```

    Name: movie_gross_df
    columns: ['primary_title', 'studio', 'domestic_gross', 'foreign_gross', 'year']
    df length:3387
    Percent Null:
    primary_title      0.000000
    studio             0.147623
    domestic_gross     0.826690
    foreign_gross     39.858282
    year               0.000000
    dtype: float64
    
    
    Name: imdb_name_df
    columns: ['nconst', 'primary_name', 'birth_year', 'death_year', 'primary_profession', 'known_for_titles']
    df length:606648
    Percent Null:
    nconst                 0.000000
    primary_name           0.000000
    birth_year            86.361778
    death_year            98.881889
    primary_profession     8.462898
    known_for_titles       4.978835
    dtype: float64
    
    
    Name: imdb_title_df
    columns: ['tconst', 'ordering', 'title', 'region', 'language', 'types', 'attributes', 'is_original_title']
    df length:331703
    Percent Null:
    tconst                0.000000
    ordering              0.000000
    title                 0.000000
    region               16.066481
    language             87.423991
    types                49.217523
    attributes           95.500493
    is_original_title     0.007537
    dtype: float64
    
    
    Name: imdb_title_basics
    columns: ['tconst', 'primary_title', 'original_title', 'start_year', 'runtime_minutes', 'genres']
    df length:146144
    Percent Null:
    tconst              0.000000
    primary_title       0.000000
    original_title      0.014369
    start_year          0.000000
    runtime_minutes    21.717621
    genres              3.700460
    dtype: float64
    
    
    Name: imdb_title_crew_df
    columns: ['tconst', 'directors', 'writers']
    df length:146144
    Percent Null:
    tconst        0.000000
    directors     3.918738
    writers      24.553180
    dtype: float64
    
    
    Name: imdb_title_princ_df
    columns: ['tconst', 'ordering', 'nconst', 'category', 'job', 'characters']
    df length:1028186
    Percent Null:
    tconst         0.000000
    ordering       0.000000
    nconst         0.000000
    category       0.000000
    job           82.718691
    characters    61.742331
    dtype: float64
    
    
    Name: imdb_title_rating_df
    columns: ['tconst', 'averagerating', 'numvotes']
    df length:73856
    Percent Null:
    tconst           0.0
    averagerating    0.0
    numvotes         0.0
    dtype: float64
    
    
    Name: rt_movie_info_df
    columns: ['id', 'synopsis', 'rating', 'genre', 'director', 'writer', 'theater_date', 'dvd_date', 'currency', 'box_office', 'runtime', 'studio']
    df length:1560
    Percent Null:
    id               0.000000
    synopsis         3.974359
    rating           0.192308
    genre            0.512821
    director        12.756410
    writer          28.782051
    theater_date    23.012821
    dvd_date        23.012821
    currency        78.205128
    box_office      78.205128
    runtime          1.923077
    studio          68.333333
    dtype: float64
    
    
    Name: rt_reviews_df
    columns: ['id', 'review', 'rating', 'fresh', 'critic', 'top_critic', 'publisher', 'date']
    df length:54432
    Percent Null:
    id             0.000000
    review        10.220091
    rating        24.832819
    fresh          0.000000
    critic         5.000735
    top_critic     0.000000
    publisher      0.567681
    date           0.000000
    dtype: float64
    
    
    Name: tmbd_movies_df
    columns: ['genre_ids', 'id', 'original_language', 'original_title', 'popularity', 'release_date', 'title', 'vote_average', 'vote_count']
    df length:26517
    Percent Null:
    genre_ids            0.0
    id                   0.0
    original_language    0.0
    original_title       0.0
    popularity           0.0
    release_date         0.0
    title                0.0
    vote_average         0.0
    vote_count           0.0
    dtype: float64
    
    
    Name: tn_movie_budget
    columns: ['id', 'release_date', 'primary_title', 'production_budget', 'domestic_gross', 'worldwide_gross']
    df length:5782
    Percent Null:
    id                   0.0
    release_date         0.0
    primary_title        0.0
    production_budget    0.0
    domestic_gross       0.0
    worldwide_gross      0.0
    dtype: float64
    
    
    

# Cleaning Data

### Right now, our target dataframes of interest are: imdb_title_basics and tn_movie_budget. Let's start by cleaning these dataframes. Above, we can view the percent of null values within each dataframe. tn_movie_budget seems to have no null values, and imdb_title_basics has very few. So let's start with imdb_title_basics. 

#### We really only care about the primary_title and genres columns within this dataframe, so let's go ahead and clean those up.


```python
imdb_title_basics_clean = imdb_title_basics.copy()
```


```python
imdb_title_basics_clean.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt0063540</td>
      <td>Sunghursh</td>
      <td>Sunghursh</td>
      <td>2013</td>
      <td>175.0</td>
      <td>Action,Crime,Drama</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt0066787</td>
      <td>One Day Before the Rainy Season</td>
      <td>Ashad Ka Ek Din</td>
      <td>2019</td>
      <td>114.0</td>
      <td>Biography,Drama</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt0069049</td>
      <td>The Other Side of the Wind</td>
      <td>The Other Side of the Wind</td>
      <td>2018</td>
      <td>122.0</td>
      <td>Drama</td>
    </tr>
    <tr>
      <td>3</td>
      <td>tt0069204</td>
      <td>Sabse Bada Sukh</td>
      <td>Sabse Bada Sukh</td>
      <td>2018</td>
      <td>NaN</td>
      <td>Comedy,Drama</td>
    </tr>
    <tr>
      <td>4</td>
      <td>tt0100275</td>
      <td>The Wandering Soap Opera</td>
      <td>La Telenovela Errante</td>
      <td>2017</td>
      <td>80.0</td>
      <td>Comedy,Drama,Fantasy</td>
    </tr>
  </tbody>
</table>
</div>




```python
perc_null(imdb_title_basics_clean)
```




    tconst              0.000000
    primary_title       0.000000
    original_title      0.014369
    start_year          0.000000
    runtime_minutes    21.717621
    genres              3.700460
    dtype: float64



#### The column with the largest  null value is runtime_minutes with 21.7% null values. We don't really need this column for the first question, but we can at least replace the NaN values with 0 for the time being.


```python
imdb_title_basics_clean['runtime_minutes'] = imdb_title_basics_clean['runtime_minutes'].fillna(0)
```


```python
perc_null(imdb_title_basics_clean)
```




    tconst             0.000000
    primary_title      0.000000
    original_title     0.014369
    start_year         0.000000
    runtime_minutes    0.000000
    genres             3.700460
    dtype: float64



#### For the genres column, we can just replace the null values with 'N/A'


```python
imdb_title_basics_clean['genres'] = imdb_title_basics_clean['genres'].fillna('N/A')
```


```python
perc_null(imdb_title_basics_clean)
```




    tconst             0.000000
    primary_title      0.000000
    original_title     0.014369
    start_year         0.000000
    runtime_minutes    0.000000
    genres             0.000000
    dtype: float64



#### Even though we aren't going to be using original_title for anything, we can still replace the null values with "unknown".


```python
imdb_title_basics_clean['original_title'] = imdb_title_basics_clean['original_title'].fillna('Unknown')
```

#### Now let's overwrite the existing dataframe by targeting the 'primary_title' column and selecting all rows that do NOT contain null values in this column


```python
len(imdb_title_basics_clean['primary_title'])
```




    146144




```python
imdb_title_basics_clean = imdb_title_basics_clean[imdb_title_basics_clean['primary_title'].notna()]
```


```python
len(imdb_title_basics_clean['primary_title'])
```




    146144




```python
perc_null(imdb_title_basics_clean)
```




    tconst             0.0
    primary_title      0.0
    original_title     0.0
    start_year         0.0
    runtime_minutes    0.0
    genres             0.0
    dtype: float64



#### Since we are going to use the column 'primary_title' as a  way to join dataframes for this question, let's find remove any duplicate values contained within the column. Since some movies may have the same title name, let's make sure to also use 'start_year' as an identifier for any true duplicate rows.


```python
imdb_title_basics_clean.duplicated(['primary_title', 'start_year']).value_counts()
```




    False    144072
    True       2072
    dtype: int64




```python
imdb_title_basics_clean = imdb_title_basics_clean.drop_duplicates(subset=['primary_title', 'start_year'])
```


```python
imdb_title_basics_clean.duplicated(['primary_title', 'start_year']).value_counts()
```




    False    144072
    dtype: int64



### Cleaning tn_movie_budget_df

#### The columns we care about in this dataframe are: primary_title, production_budget, release_date and worldwide_gross.


```python
tn_movie_budget_df = tn_movie_budget.copy()
```


```python
tn_movie_budget_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>425000000.0</td>
      <td>760507625.0</td>
      <td>2.776345e+09</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2</td>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <td>2</td>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
  </tbody>
</table>
</div>




```python
perc_null(tn_movie_budget_df)
```




    id                   0.0
    release_date         0.0
    primary_title        0.0
    production_budget    0.0
    domestic_gross       0.0
    worldwide_gross      0.0
    dtype: float64




```python
tn_movie_budget_df.dtypes
```




    id                     int64
    release_date          object
    primary_title         object
    production_budget    float64
    domestic_gross       float64
    worldwide_gross      float64
    dtype: object



## Test Area


```python
practice = tn_movie_budget.copy().reset_index(drop=True)
```


```python
practice.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>425000000.0</td>
      <td>760507625.0</td>
      <td>2.776345e+09</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2</td>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <td>2</td>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
    <tr>
      <td>3</td>
      <td>4</td>
      <td>May 1, 2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
    </tr>
    <tr>
      <td>4</td>
      <td>5</td>
      <td>Dec 15, 2017</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>317000000.0</td>
      <td>620181382.0</td>
      <td>1.316722e+09</td>
    </tr>
  </tbody>
</table>
</div>




```python
practice['release_date'] = pd.to_datetime(practice['release_date'])
```


```python
practice.sort_values('release_date', ascending=False)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>194</td>
      <td>95</td>
      <td>2020-12-31</td>
      <td>Moonfall</td>
      <td>150000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>1205</td>
      <td>6</td>
      <td>2020-12-31</td>
      <td>Hannibal the Conqueror</td>
      <td>50000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>535</td>
      <td>36</td>
      <td>2020-02-21</td>
      <td>Call of the Wild</td>
      <td>82000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>480</td>
      <td>81</td>
      <td>2019-12-31</td>
      <td>Army of the Dead</td>
      <td>90000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>3515</td>
      <td>16</td>
      <td>2019-12-31</td>
      <td>Eli</td>
      <td>11000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>5606</td>
      <td>7</td>
      <td>1925-11-19</td>
      <td>The Big Parade</td>
      <td>245000.0</td>
      <td>11000000.0</td>
      <td>22000000.0</td>
    </tr>
    <tr>
      <td>5683</td>
      <td>84</td>
      <td>1920-09-17</td>
      <td>Over the Hill to the Poorhouse</td>
      <td>100000.0</td>
      <td>3000000.0</td>
      <td>3000000.0</td>
    </tr>
    <tr>
      <td>5614</td>
      <td>15</td>
      <td>1916-12-24</td>
      <td>20,000 Leagues Under the Sea</td>
      <td>200000.0</td>
      <td>8000000.0</td>
      <td>8000000.0</td>
    </tr>
    <tr>
      <td>5523</td>
      <td>24</td>
      <td>1916-09-05</td>
      <td>Intolerance</td>
      <td>385907.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>5677</td>
      <td>78</td>
      <td>1915-02-08</td>
      <td>The Birth of a Nation</td>
      <td>110000.0</td>
      <td>10000000.0</td>
      <td>11000000.0</td>
    </tr>
  </tbody>
</table>
<p>5782 rows × 6 columns</p>
</div>



## End Test Area

#### While we could change release_date into a pd.datetime object, however we are just going to change it into a numpy.int64 instead.


```python
type(tn_movie_budget_df['release_date'][0])
```




    str




```python
len(tn_movie_budget_df['release_date'][0])
```




    12




```python
tn_movie_budget_df['release_date'][0][-4:]
```




    '2009'




```python
tn_movie_budget_df['release_date'] = tn_movie_budget_df['release_date'].map(lambda x:f'{x[-4:]}')
```


```python
tn_movie_budget_df['release_date'][0]
```




    '2009'




```python
tn_movie_budget_df['release_date'] = tn_movie_budget_df['release_date'].astype('int64')
```


```python
type(tn_movie_budget_df['release_date'][0])
```




    numpy.int64




```python
tn_movie_budget_df.sort_values('release_date', ascending=False)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>194</td>
      <td>95</td>
      <td>2020</td>
      <td>Moonfall</td>
      <td>150000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>535</td>
      <td>36</td>
      <td>2020</td>
      <td>Call of the Wild</td>
      <td>82000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>1205</td>
      <td>6</td>
      <td>2020</td>
      <td>Hannibal the Conqueror</td>
      <td>50000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>2029</td>
      <td>30</td>
      <td>2019</td>
      <td>Unhinged</td>
      <td>29000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>670</td>
      <td>71</td>
      <td>2019</td>
      <td>PLAYMOBIL</td>
      <td>75000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>5606</td>
      <td>7</td>
      <td>1925</td>
      <td>The Big Parade</td>
      <td>245000.0</td>
      <td>11000000.0</td>
      <td>22000000.0</td>
    </tr>
    <tr>
      <td>5683</td>
      <td>84</td>
      <td>1920</td>
      <td>Over the Hill to the Poorhouse</td>
      <td>100000.0</td>
      <td>3000000.0</td>
      <td>3000000.0</td>
    </tr>
    <tr>
      <td>5614</td>
      <td>15</td>
      <td>1916</td>
      <td>20,000 Leagues Under the Sea</td>
      <td>200000.0</td>
      <td>8000000.0</td>
      <td>8000000.0</td>
    </tr>
    <tr>
      <td>5523</td>
      <td>24</td>
      <td>1916</td>
      <td>Intolerance</td>
      <td>385907.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>5677</td>
      <td>78</td>
      <td>1915</td>
      <td>The Birth of a Nation</td>
      <td>110000.0</td>
      <td>10000000.0</td>
      <td>11000000.0</td>
    </tr>
  </tbody>
</table>
<p>5782 rows × 6 columns</p>
</div>




```python
tn_movie_budget_df.duplicated(['primary_title', 'release_date']).value_counts()
```




    False    5781
    True        1
    dtype: int64




```python
tn_movie_budget_df = tn_movie_budget_df.drop_duplicates(['primary_title', 'release_date'])
```


```python
tn_movie_budget_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>2009</td>
      <td>Avatar</td>
      <td>425000000.0</td>
      <td>760507625.0</td>
      <td>2.776345e+09</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2</td>
      <td>2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <td>2</td>
      <td>3</td>
      <td>2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
    <tr>
      <td>3</td>
      <td>4</td>
      <td>2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
    </tr>
    <tr>
      <td>4</td>
      <td>5</td>
      <td>2017</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>317000000.0</td>
      <td>620181382.0</td>
      <td>1.316722e+09</td>
    </tr>
  </tbody>
</table>
</div>



# Question 1: Is there an association between production budget and Worldwide Gross for Movies in the Past Ten Years?

### make a dataframe containing ALL movies from 2010-2020 where worldwide gross is not 0.


```python
bud_v_net = pysqldf("""SELECT i.tconst, i.genres, i.runtime_minutes, t.*
                    FROM tn_movie_budget_df t
                    JOIN imdb_title_basics_clean i
                    USING(primary_title)
                    WHERE (release_date > 2009 AND release_date <= 2020)
                           AND (worldwide_gross != 0);""")
```


```python
 bud_v_net['tot_profit'] = bud_v_net['worldwide_gross'] - bud_v_net['production_budget']
```


```python
bud_v_net.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>genres</th>
      <th>runtime_minutes</th>
      <th>id</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>tot_profit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt1298650</td>
      <td>Action,Adventure,Fantasy</td>
      <td>136.0</td>
      <td>2</td>
      <td>2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>6.350639e+08</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt6565702</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>113.0</td>
      <td>3</td>
      <td>2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
      <td>-2.002376e+08</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt2395427</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>141.0</td>
      <td>4</td>
      <td>2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>1.072414e+09</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(10,10))
sns.set(style='darkgrid', font_scale=2)
s = sns.regplot('production_budget', 'worldwide_gross', data=bud_v_net, 
                ci=99, robust=True, color='r', marker='+')
s.set_xlabel('Production Budget')
s.set_ylabel('Worldwide Gross')
s.set_title('Association Between Production Budget and Worldwide Gross of Movies')
s.set_xticklabels(s.get_xticklabels(), rotation=25, ha='right')

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '${x:,.0f}'
tick = mtick.StrMethodFormatter(fmt)
s.yaxis.set_major_formatter(tick)
s.xaxis.set_major_formatter(tick);
```


![png](output_131_0.png)



```python
bud_v_net.corr()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>runtime_minutes</th>
      <th>id</th>
      <th>release_date</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>tot_profit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>runtime_minutes</td>
      <td>1.000000</td>
      <td>-0.046317</td>
      <td>0.039317</td>
      <td>0.185647</td>
      <td>0.154092</td>
      <td>0.172186</td>
      <td>0.154772</td>
    </tr>
    <tr>
      <td>id</td>
      <td>-0.046317</td>
      <td>1.000000</td>
      <td>0.045306</td>
      <td>-0.038818</td>
      <td>0.011343</td>
      <td>-0.011350</td>
      <td>-0.002510</td>
    </tr>
    <tr>
      <td>release_date</td>
      <td>0.039317</td>
      <td>0.045306</td>
      <td>1.000000</td>
      <td>0.049710</td>
      <td>0.085121</td>
      <td>0.084282</td>
      <td>0.087671</td>
    </tr>
    <tr>
      <td>production_budget</td>
      <td>0.185647</td>
      <td>-0.038818</td>
      <td>0.049710</td>
      <td>1.000000</td>
      <td>0.715502</td>
      <td>0.784757</td>
      <td>0.660855</td>
    </tr>
    <tr>
      <td>domestic_gross</td>
      <td>0.154092</td>
      <td>0.011343</td>
      <td>0.085121</td>
      <td>0.715502</td>
      <td>1.000000</td>
      <td>0.946856</td>
      <td>0.939473</td>
    </tr>
    <tr>
      <td>worldwide_gross</td>
      <td>0.172186</td>
      <td>-0.011350</td>
      <td>0.084282</td>
      <td>0.784757</td>
      <td>0.946856</td>
      <td>1.000000</td>
      <td>0.983782</td>
    </tr>
    <tr>
      <td>tot_profit</td>
      <td>0.154772</td>
      <td>-0.002510</td>
      <td>0.087671</td>
      <td>0.660855</td>
      <td>0.939473</td>
      <td>0.983782</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



## Answer:

#### Since the correlation between production_budget and worldwide_gross is greater than 0.7 (it is 0.78), we can conclude that there is a strong, positive correlation between production_budget and worldwide_gross. 
- **Recommendation** The more money you put into your production budget, the more money you will potentially make in worldwide gross. So spend as much as you can!! 

## Test Area



```python
rate_v_roi = pysqldf("""SELECT c.primary_title, r.averagerating, b.production_budget,
                        b.worldwide_gross, b.release_date
                        FROM imdb_title_basics_clean c
                        JOIN imdb_title_rating_df r
                        USING(tconst)
                        JOIN tn_movie_budget_df b
                        USING(primary_title)
                        WHERE (worldwide_gross != 0);""")
```


```python
len(rate_v_roi)
```




    2459




```python
rate_v_roi['tot_profit'] = rate_v_roi['worldwide_gross'] - rate_v_roi['production_budget']
```


```python
rate_v_roi['ROI'] = (rate_v_roi['tot_profit']/rate_v_roi['production_budget']) * 100
```


```python
rate_v_roi.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>averagerating</th>
      <th>production_budget</th>
      <th>worldwide_gross</th>
      <th>release_date</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Avatar</td>
      <td>6.1</td>
      <td>425000000.0</td>
      <td>2.776345e+09</td>
      <td>2009</td>
      <td>2.351345e+09</td>
      <td>553.257713</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>6.6</td>
      <td>410600000.0</td>
      <td>1.045664e+09</td>
      <td>2011</td>
      <td>6.350639e+08</td>
      <td>154.667286</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Dark Phoenix</td>
      <td>6.0</td>
      <td>350000000.0</td>
      <td>1.497624e+08</td>
      <td>2019</td>
      <td>-2.002376e+08</td>
      <td>-57.210757</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(10,10))
sns.set(style='darkgrid', font_scale=2)
s = sns.regplot('averagerating', 'ROI', data=rate_v_roi, robust=True,
                ci=68, color='g', marker='+')
s.set_xlabel('Average Rating')
s.set_ylabel('ROI')
s.set_title('Association Between Average Rating and ROI of Movies')
s.set_xticklabels(s.get_xticklabels(), ha='right')

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt1 = '{x:,.0f}%'
tick1 = mtick.StrMethodFormatter(fmt1)
fmt2 = '{x:,.0f}.0'
tick2 = mtick.StrMethodFormatter(fmt2)
s.yaxis.set_major_formatter(tick1)
s.xaxis.set_major_formatter(tick2);
```


![png](output_141_0.png)



```python
rate_v_roi.corr()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>averagerating</th>
      <th>production_budget</th>
      <th>worldwide_gross</th>
      <th>release_date</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>averagerating</td>
      <td>1.000000</td>
      <td>0.105962</td>
      <td>0.159240</td>
      <td>0.045477</td>
      <td>0.161573</td>
      <td>-0.009020</td>
    </tr>
    <tr>
      <td>production_budget</td>
      <td>0.105962</td>
      <td>1.000000</td>
      <td>0.774251</td>
      <td>0.136020</td>
      <td>0.650410</td>
      <td>-0.060316</td>
    </tr>
    <tr>
      <td>worldwide_gross</td>
      <td>0.159240</td>
      <td>0.774251</td>
      <td>1.000000</td>
      <td>0.107443</td>
      <td>0.984305</td>
      <td>0.081843</td>
    </tr>
    <tr>
      <td>release_date</td>
      <td>0.045477</td>
      <td>0.136020</td>
      <td>0.107443</td>
      <td>1.000000</td>
      <td>0.091025</td>
      <td>-0.233376</td>
    </tr>
    <tr>
      <td>tot_profit</td>
      <td>0.161573</td>
      <td>0.650410</td>
      <td>0.984305</td>
      <td>0.091025</td>
      <td>1.000000</td>
      <td>0.115048</td>
    </tr>
    <tr>
      <td>ROI</td>
      <td>-0.009020</td>
      <td>-0.060316</td>
      <td>0.081843</td>
      <td>-0.233376</td>
      <td>0.115048</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



#### According to the graph above, there is no correlation (-0.009) between the average rating of voters and the Return On Investment of movies.


```python
rate_v_bud = pysqldf("""SELECT *
                        FROM rate_v_roi
                        WHERE (release_date > 2009 AND release_date <= 2020);""")
```


```python
rate_v_bud.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>averagerating</th>
      <th>production_budget</th>
      <th>worldwide_gross</th>
      <th>release_date</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>6.6</td>
      <td>410600000.0</td>
      <td>1.045664e+09</td>
      <td>2011</td>
      <td>6.350639e+08</td>
      <td>154.667286</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Dark Phoenix</td>
      <td>6.0</td>
      <td>350000000.0</td>
      <td>1.497624e+08</td>
      <td>2019</td>
      <td>-2.002376e+08</td>
      <td>-57.210757</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Avengers: Age of Ultron</td>
      <td>7.3</td>
      <td>330600000.0</td>
      <td>1.403014e+09</td>
      <td>2015</td>
      <td>1.072414e+09</td>
      <td>324.384139</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(10,10))
sns.set(style='darkgrid', font_scale=2)
s = sns.regplot('averagerating', 'production_budget', data=rate_v_bud, robust=True,
                ci=68, color='blue', marker='+')
s.set_xlabel('Average Rating')
s.set_ylabel('Production Budget')
s.set_title('Association Between Average Rating and Production Budget of Movies')
s.set_xticklabels(s.get_xticklabels(), ha='right')

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt1 = '${x:,.0f}'
tick1 = mtick.StrMethodFormatter(fmt1)
fmt2 = '{x:,.0f}.0'
tick2 = mtick.StrMethodFormatter(fmt2)
s.yaxis.set_major_formatter(tick1)
s.xaxis.set_major_formatter(tick2);
```


![png](output_146_0.png)



```python
rate_v_bud.corr()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>averagerating</th>
      <th>production_budget</th>
      <th>worldwide_gross</th>
      <th>release_date</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>averagerating</td>
      <td>1.000000</td>
      <td>0.140121</td>
      <td>0.211653</td>
      <td>0.014295</td>
      <td>0.215779</td>
      <td>-0.004713</td>
    </tr>
    <tr>
      <td>production_budget</td>
      <td>0.140121</td>
      <td>1.000000</td>
      <td>0.785277</td>
      <td>0.042997</td>
      <td>0.661278</td>
      <td>-0.043076</td>
    </tr>
    <tr>
      <td>worldwide_gross</td>
      <td>0.211653</td>
      <td>0.785277</td>
      <td>1.000000</td>
      <td>0.083660</td>
      <td>0.983732</td>
      <td>0.097787</td>
    </tr>
    <tr>
      <td>release_date</td>
      <td>0.014295</td>
      <td>0.042997</td>
      <td>0.083660</td>
      <td>1.000000</td>
      <td>0.088886</td>
      <td>0.039526</td>
    </tr>
    <tr>
      <td>tot_profit</td>
      <td>0.215779</td>
      <td>0.661278</td>
      <td>0.983732</td>
      <td>0.088886</td>
      <td>1.000000</td>
      <td>0.130974</td>
    </tr>
    <tr>
      <td>ROI</td>
      <td>-0.004713</td>
      <td>-0.043076</td>
      <td>0.097787</td>
      <td>0.039526</td>
      <td>0.130974</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



#### According to the graph, there is little to no correlation (0.12) between Production Budget and Average Rating


```python
rate_v_tot = rate_v_bud.copy()
```


```python
rate_v_tot.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>averagerating</th>
      <th>production_budget</th>
      <th>worldwide_gross</th>
      <th>release_date</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>6.6</td>
      <td>410600000.0</td>
      <td>1.045664e+09</td>
      <td>2011</td>
      <td>6.350639e+08</td>
      <td>154.667286</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Dark Phoenix</td>
      <td>6.0</td>
      <td>350000000.0</td>
      <td>1.497624e+08</td>
      <td>2019</td>
      <td>-2.002376e+08</td>
      <td>-57.210757</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Avengers: Age of Ultron</td>
      <td>7.3</td>
      <td>330600000.0</td>
      <td>1.403014e+09</td>
      <td>2015</td>
      <td>1.072414e+09</td>
      <td>324.384139</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(10,10))
sns.set(style='darkgrid', font_scale=2)
s = sns.regplot('averagerating', 'tot_profit', data=rate_v_bud, robust=True,
                ci=68, color='black', marker='+')
s.set_xlabel('Average Rating')
s.set_ylabel('Total Profit')
s.set_title('Association Between Average Rating and Total Profit of Movies')
s.set_xticklabels(s.get_xticklabels(), ha='right')

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt1 = '${x:,.0f}'
tick1 = mtick.StrMethodFormatter(fmt1)
fmt2 = '{x:,.0f}.0'
tick2 = mtick.StrMethodFormatter(fmt2)
s.yaxis.set_major_formatter(tick1)
s.xaxis.set_major_formatter(tick2);
```


![png](output_151_0.png)



```python
rate_v_tot.corr()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>averagerating</th>
      <th>production_budget</th>
      <th>worldwide_gross</th>
      <th>release_date</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>averagerating</td>
      <td>1.000000</td>
      <td>0.140121</td>
      <td>0.211653</td>
      <td>0.014295</td>
      <td>0.215779</td>
      <td>-0.004713</td>
    </tr>
    <tr>
      <td>production_budget</td>
      <td>0.140121</td>
      <td>1.000000</td>
      <td>0.785277</td>
      <td>0.042997</td>
      <td>0.661278</td>
      <td>-0.043076</td>
    </tr>
    <tr>
      <td>worldwide_gross</td>
      <td>0.211653</td>
      <td>0.785277</td>
      <td>1.000000</td>
      <td>0.083660</td>
      <td>0.983732</td>
      <td>0.097787</td>
    </tr>
    <tr>
      <td>release_date</td>
      <td>0.014295</td>
      <td>0.042997</td>
      <td>0.083660</td>
      <td>1.000000</td>
      <td>0.088886</td>
      <td>0.039526</td>
    </tr>
    <tr>
      <td>tot_profit</td>
      <td>0.215779</td>
      <td>0.661278</td>
      <td>0.983732</td>
      <td>0.088886</td>
      <td>1.000000</td>
      <td>0.130974</td>
    </tr>
    <tr>
      <td>ROI</td>
      <td>-0.004713</td>
      <td>-0.043076</td>
      <td>0.097787</td>
      <td>0.039526</td>
      <td>0.130974</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



#### according to the graph, there is little correlation (0.216) between average ratings and total profit.


```python
rate_v_wwg = pysqldf("""SELECT c.primary_title, r.averagerating, b.production_budget,
                        b.worldwide_gross, b.release_date
                        FROM imdb_title_basics_clean c
                        JOIN imdb_title_rating_df r
                        USING(tconst)
                        JOIN tn_movie_budget_df b
                        USING(primary_title)
                        WHERE (worldwide_gross != 0)
                               AND (b.release_date > 2009 AND b.release_date <= 2020);""")
```


```python
rate_v_wwg.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>averagerating</th>
      <th>production_budget</th>
      <th>worldwide_gross</th>
      <th>release_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>6.6</td>
      <td>410600000.0</td>
      <td>1.045664e+09</td>
      <td>2011</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Dark Phoenix</td>
      <td>6.0</td>
      <td>350000000.0</td>
      <td>1.497624e+08</td>
      <td>2019</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Avengers: Age of Ultron</td>
      <td>7.3</td>
      <td>330600000.0</td>
      <td>1.403014e+09</td>
      <td>2015</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(10,10))
sns.set(style='darkgrid', font_scale=2)
s = sns.regplot('averagerating', 'worldwide_gross', data=rate_v_wwg, robust=True,
                ci=68, color='orange', marker='+')
s.set_xlabel('Average Rating')
s.set_ylabel('Worldwide Gross')
s.set_title('Association Between Average Rating and Worldwide Gross of Movies')
s.set_xticklabels(s.get_xticklabels(), ha='right')

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt1 = '${x:,.0f}'
tick1 = mtick.StrMethodFormatter(fmt1)
fmt2 = '{x:,.0f}.0'
tick2 = mtick.StrMethodFormatter(fmt2)
s.yaxis.set_major_formatter(tick1)
s.xaxis.set_major_formatter(tick2);
```


![png](output_156_0.png)



```python
rate_v_wwg.corr()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>averagerating</th>
      <th>production_budget</th>
      <th>worldwide_gross</th>
      <th>release_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>averagerating</td>
      <td>1.000000</td>
      <td>0.140121</td>
      <td>0.211653</td>
      <td>0.014295</td>
    </tr>
    <tr>
      <td>production_budget</td>
      <td>0.140121</td>
      <td>1.000000</td>
      <td>0.785277</td>
      <td>0.042997</td>
    </tr>
    <tr>
      <td>worldwide_gross</td>
      <td>0.211653</td>
      <td>0.785277</td>
      <td>1.000000</td>
      <td>0.083660</td>
    </tr>
    <tr>
      <td>release_date</td>
      <td>0.014295</td>
      <td>0.042997</td>
      <td>0.083660</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



#### According to the graph above, there is a small positive correlation (0.21) between Average Rating and Worldwide gross of movies.


```python
min_v_bud = pysqldf("""SELECT b.*, c.runtime_minutes
                        FROM imdb_title_basics_clean c
                        JOIN imdb_title_rating_df r
                        USING(tconst)
                        JOIN tn_movie_budget_df b
                        USING(primary_title)
                        WHERE (runtime_minutes > 0)
                               AND (production_budget > 0);""")
```


```python
min_v_bud.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>runtime_minutes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>26</td>
      <td>2012</td>
      <td>Foodfight!</td>
      <td>45000000.0</td>
      <td>0.0</td>
      <td>73706.0</td>
      <td>91.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>21</td>
      <td>2015</td>
      <td>The Overnight</td>
      <td>200000.0</td>
      <td>1109808.0</td>
      <td>1165996.0</td>
      <td>88.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>17</td>
      <td>2013</td>
      <td>On the Road</td>
      <td>25000000.0</td>
      <td>720828.0</td>
      <td>9313302.0</td>
      <td>124.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(10,10))
sns.set(style='darkgrid', font_scale=2)
s = sns.regplot('runtime_minutes', 'production_budget', data=min_v_bud, robust=True,
                ci=68, color='purple', marker='+')
s.set_xlabel('Minutes')
s.set_ylabel('Production Budget')
s.set_title('Association Between Runtime Minutes and Budget of Movies')
s.set_xticklabels(s.get_xticklabels(), ha='right', rotation=10)

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt1 = '{x:,.0f} min'
tick1 = mtick.StrMethodFormatter(fmt1)
fmt2 = '{x:,.0f}.0'
tick2 = mtick.StrMethodFormatter(fmt2)
s.yaxis.set_major_formatter(tick2)
s.xaxis.set_major_formatter(tick1);
```


![png](output_161_0.png)



```python
min_v_bud.corr()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>runtime_minutes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>id</td>
      <td>1.000000</td>
      <td>0.003202</td>
      <td>-0.045578</td>
      <td>-0.005488</td>
      <td>-0.021132</td>
      <td>-0.036211</td>
    </tr>
    <tr>
      <td>release_date</td>
      <td>0.003202</td>
      <td>1.000000</td>
      <td>0.114039</td>
      <td>0.055354</td>
      <td>0.095251</td>
      <td>0.098647</td>
    </tr>
    <tr>
      <td>production_budget</td>
      <td>-0.045578</td>
      <td>0.114039</td>
      <td>1.000000</td>
      <td>0.717247</td>
      <td>0.785041</td>
      <td>0.265457</td>
    </tr>
    <tr>
      <td>domestic_gross</td>
      <td>-0.005488</td>
      <td>0.055354</td>
      <td>0.717247</td>
      <td>1.000000</td>
      <td>0.945452</td>
      <td>0.215024</td>
    </tr>
    <tr>
      <td>worldwide_gross</td>
      <td>-0.021132</td>
      <td>0.095251</td>
      <td>0.785041</td>
      <td>0.945452</td>
      <td>1.000000</td>
      <td>0.229400</td>
    </tr>
    <tr>
      <td>runtime_minutes</td>
      <td>-0.036211</td>
      <td>0.098647</td>
      <td>0.265457</td>
      <td>0.215024</td>
      <td>0.229400</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



#### According to the graph above, there is a small positive correlation (0.265) between runtime minutes of a movie and its production budget.

# Question 2: Which genres (from U.S. movies) have been performing the best in terms of average total net profit since 2010?

#### Finding Unique Genres

#### First, let's grab only the movies that were created in the United States.


```python
imdb_title_basics_df = pysqldf('''SELECT b.*
                                  FROM imdb_title_basics_clean b
                                  JOIN imdb_title_df t
                                  USING (tconst)
                                  WHERE (t.region = 'US')
                                  GROUP BY (primary_title);''')
```

#### We are going to use the imdb_title_basics_df['genres'] column to retrieve all unique genres within that column.


```python
imdb_title_basics_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt1699720</td>
      <td>!Women Art Revolution</td>
      <td>Women Art Revolution</td>
      <td>2010</td>
      <td>83.0</td>
      <td>Documentary</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt2346170</td>
      <td>#1 Serial Killer</td>
      <td>#1 Serial Killer</td>
      <td>2013</td>
      <td>87.0</td>
      <td>Horror</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt3120962</td>
      <td>#5</td>
      <td>#5</td>
      <td>2013</td>
      <td>68.0</td>
      <td>Biography,Comedy,Fantasy</td>
    </tr>
  </tbody>
</table>
</div>



#### In order to get what we want, we are going to run a test on the first row of the genre column


```python
genre_list = imdb_title_basics_df['genres']
test = imdb_title_basics_df['genres'].iloc[0]
test
```




    'Documentary'




```python
type(test)
```




    str



#### Above we see that it is a string, so let's try to use the .str.contains() method to see what happens.


```python
is_action = genre_list.str.contains('Documentary')
is_action[:5]
```




    0     True
    1    False
    2    False
    3    False
    4     True
    Name: genres, dtype: bool



#### It gave us a list of booleans based off of what we were searching for (in that example, 'Action'). Let's make a copy of the imdb_title_basics_df (only columns 'primary_title" and 'genres') so we don't accidentally overwrite that df and use the copy to experiment with.


```python
df = imdb_title_basics_df[['primary_title','genres']].copy()
```


```python
df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>!Women Art Revolution</td>
      <td>Documentary</td>
    </tr>
    <tr>
      <td>1</td>
      <td>#1 Serial Killer</td>
      <td>Horror</td>
    </tr>
    <tr>
      <td>2</td>
      <td>#5</td>
      <td>Biography,Comedy,Fantasy</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(df)
```




    45218



#### Let's split each string in each row of the column 'genres' in our new df and see what happens.


```python
df['genres'].str.split(',')
```




    0                       [Documentary]
    1                            [Horror]
    2        [Biography, Comedy, Fantasy]
    3                            [Comedy]
    4                       [Documentary]
                         ...             
    45213                         [Drama]
    45214                   [Documentary]
    45215                         [Drama]
    45216                         [Drama]
    45217                   [Documentary]
    Name: genres, Length: 45218, dtype: object



#### Now, let's make a  variable named 'genres' and use a comma as a separator and join all the data within the df['genres'] column into the 'genres' variable and drop any null values.


```python
genres = ','.join(df['genres'].dropna())
```


```python
len(genres)
```




    668280



#### Let's make a new variable called 'unique_genres' and make a list of a set of the values within the variable 'genres' and have all values lowercased and split with ',' 


```python
unique_genres = list(set(genres.lower().split(',')))
unique_genres
```




    ['war',
     'talk-show',
     'sport',
     'news',
     'history',
     'western',
     'thriller',
     'music',
     'mystery',
     'game-show',
     'action',
     'musical',
     'drama',
     'n/a',
     'horror',
     'biography',
     'adventure',
     'adult',
     'sci-fi',
     'fantasy',
     'animation',
     'crime',
     'short',
     'documentary',
     'reality-tv',
     'family',
     'comedy',
     'romance']



### Defining another Function

#### Let's make another function that will add a new column for each possible genre, where the value will be a boolean value for each movie within the dataframe.


```python
def genre_cols(df, genres):
    for i in genres:
        if i == 'musical':
            df[f'is_music'] = df['genres'].str.contains(f'{i}'.title())
        else:
            df[f'is_{i}'] = df['genres'].str.contains(f'{i}'.title())
    return df
        
```


```python
genre_df = genre_cols(df, unique_genres)
genre_df = genre_df.dropna()
genre_df = genre_df.drop(columns='genres')
```


```python
genre_df.columns
```




    Index(['primary_title', 'is_war', 'is_talk-show', 'is_sport', 'is_news',
           'is_history', 'is_western', 'is_thriller', 'is_music', 'is_mystery',
           'is_game-show', 'is_action', 'is_drama', 'is_n/a', 'is_horror',
           'is_biography', 'is_adventure', 'is_adult', 'is_sci-fi', 'is_fantasy',
           'is_animation', 'is_crime', 'is_short', 'is_documentary',
           'is_reality-tv', 'is_family', 'is_comedy', 'is_romance'],
          dtype='object')




```python
genre_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>is_war</th>
      <th>is_talk-show</th>
      <th>is_sport</th>
      <th>is_news</th>
      <th>is_history</th>
      <th>is_western</th>
      <th>is_thriller</th>
      <th>is_music</th>
      <th>is_mystery</th>
      <th>...</th>
      <th>is_sci-fi</th>
      <th>is_fantasy</th>
      <th>is_animation</th>
      <th>is_crime</th>
      <th>is_short</th>
      <th>is_documentary</th>
      <th>is_reality-tv</th>
      <th>is_family</th>
      <th>is_comedy</th>
      <th>is_romance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>!Women Art Revolution</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>...</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <td>1</td>
      <td>#1 Serial Killer</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>...</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <td>2</td>
      <td>#5</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>...</td>
      <td>False</td>
      <td>True</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 28 columns</p>
</div>



## Separating grouped genres into individual genre columns with boolean values

#### We are going to join genre_df with imdb_title_basics_df using 'primary_title' as the column to join on and place that new dataframe in the variable indiv_genre.


```python
imdb_title_basics_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt1699720</td>
      <td>!Women Art Revolution</td>
      <td>Women Art Revolution</td>
      <td>2010</td>
      <td>83.0</td>
      <td>Documentary</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt2346170</td>
      <td>#1 Serial Killer</td>
      <td>#1 Serial Killer</td>
      <td>2013</td>
      <td>87.0</td>
      <td>Horror</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt3120962</td>
      <td>#5</td>
      <td>#5</td>
      <td>2013</td>
      <td>68.0</td>
      <td>Biography,Comedy,Fantasy</td>
    </tr>
  </tbody>
</table>
</div>




```python
indiv_genre = pysqldf("""Select g.*, i.tconst, i.original_title, i.genres
               FROM genre_df g
               JOIN imdb_title_basics_df i
               USING(primary_title)
               ;""")
```


```python
len(imdb_title_basics_df)
```




    45218




```python
len(indiv_genre)
```




    45218




```python
indiv_genre.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>is_war</th>
      <th>is_talk-show</th>
      <th>is_sport</th>
      <th>is_news</th>
      <th>is_history</th>
      <th>is_western</th>
      <th>is_thriller</th>
      <th>is_music</th>
      <th>is_mystery</th>
      <th>...</th>
      <th>is_crime</th>
      <th>is_short</th>
      <th>is_documentary</th>
      <th>is_reality-tv</th>
      <th>is_family</th>
      <th>is_comedy</th>
      <th>is_romance</th>
      <th>tconst</th>
      <th>original_title</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>!Women Art Revolution</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>tt1699720</td>
      <td>Women Art Revolution</td>
      <td>Documentary</td>
    </tr>
    <tr>
      <td>1</td>
      <td>#1 Serial Killer</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>tt2346170</td>
      <td>#1 Serial Killer</td>
      <td>Horror</td>
    </tr>
    <tr>
      <td>2</td>
      <td>#5</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>tt3120962</td>
      <td>#5</td>
      <td>Biography,Comedy,Fantasy</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 31 columns</p>
</div>



#### Now let's join indiv_genre with tn_movie_budget_df using primary_title, where we filter out any movies that did not release between 2010 and 2020, and their worldwide gross is not equal to 0. We will place that joined dataframe in a variable called indiv_genre_df. 


```python
indiv_genre_df = pysqldf("""SELECT i.*, t.production_budget, t.domestic_gross, t.worldwide_gross
                   FROM indiv_genre i
                   JOIN tn_movie_budget_df t
                   USING(primary_title)
                   WHERE (t.release_date > 2009 AND t.release_date <= 2020)
                         AND (t.worldwide_gross != 0);""")
```


```python
indiv_genre_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>is_war</th>
      <th>is_talk-show</th>
      <th>is_sport</th>
      <th>is_news</th>
      <th>is_history</th>
      <th>is_western</th>
      <th>is_thriller</th>
      <th>is_music</th>
      <th>is_mystery</th>
      <th>...</th>
      <th>is_reality-tv</th>
      <th>is_family</th>
      <th>is_comedy</th>
      <th>is_romance</th>
      <th>tconst</th>
      <th>original_title</th>
      <th>genres</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>tt1298650</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>Action,Adventure,Fantasy</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Dark Phoenix</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>tt6565702</td>
      <td>Dark Phoenix</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Avengers: Age of Ultron</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>tt2395427</td>
      <td>Avengers: Age of Ultron</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 34 columns</p>
</div>



#### We are going to drop any duplicates by targeting primary_title, release_date, and production_budget


```python
len(indiv_genre_df)
```




    1398




```python
indiv_genre_df.duplicated(['primary_title', 'production_budget']).value_counts()
```




    False    1398
    dtype: int64



#### Now we are going to add a column called ['tot_profit'] which will contain the net profit each movie made by subtracting production_budget from worldwide_gross.


```python
indiv_genre_df['tot_profit'] = indiv_genre_df['worldwide_gross'] - indiv_genre_df['production_budget']
```


```python
indiv_genre_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>is_war</th>
      <th>is_talk-show</th>
      <th>is_sport</th>
      <th>is_news</th>
      <th>is_history</th>
      <th>is_western</th>
      <th>is_thriller</th>
      <th>is_music</th>
      <th>is_mystery</th>
      <th>...</th>
      <th>is_family</th>
      <th>is_comedy</th>
      <th>is_romance</th>
      <th>tconst</th>
      <th>original_title</th>
      <th>genres</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>tot_profit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>tt1298650</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>Action,Adventure,Fantasy</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>6.350639e+08</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Dark Phoenix</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>tt6565702</td>
      <td>Dark Phoenix</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
      <td>-2.002376e+08</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Avengers: Age of Ultron</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>tt2395427</td>
      <td>Avengers: Age of Ultron</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>1.072414e+09</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 35 columns</p>
</div>



#### Let's also make another column called ['ROI'] that gives us the return on investment for each movie.


```python
indiv_genre_df['ROI'] = (indiv_genre_df['tot_profit']/indiv_genre_df['production_budget']) * 100
```


```python
indiv_genre_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>is_war</th>
      <th>is_talk-show</th>
      <th>is_sport</th>
      <th>is_news</th>
      <th>is_history</th>
      <th>is_western</th>
      <th>is_thriller</th>
      <th>is_music</th>
      <th>is_mystery</th>
      <th>...</th>
      <th>is_comedy</th>
      <th>is_romance</th>
      <th>tconst</th>
      <th>original_title</th>
      <th>genres</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt1298650</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>Action,Adventure,Fantasy</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>6.350639e+08</td>
      <td>154.667286</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Dark Phoenix</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt6565702</td>
      <td>Dark Phoenix</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
      <td>-2.002376e+08</td>
      <td>-57.210757</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Avengers: Age of Ultron</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt2395427</td>
      <td>Avengers: Age of Ultron</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>1.072414e+09</td>
      <td>324.384139</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 36 columns</p>
</div>



#### Let's loop through each column within the indiv_genre_df and see if any columns have nothing in them. If so, we can remove them from the df.


```python
for col in indiv_genre_df:
    if 'is_' in col:
        print(f"{col}: {indiv_genre_df[col].sum()}")
    else:
        continue
    
```

    is_war: 18
    is_talk-show: 0
    is_sport: 31
    is_news: 0
    is_history: 41
    is_western: 11
    is_thriller: 240
    is_music: 7
    is_mystery: 121
    is_game-show: 0
    is_action: 394
    is_drama: 670
    is_n/a: 3
    is_horror: 159
    is_biography: 112
    is_adventure: 338
    is_adult: 0
    is_sci-fi: 121
    is_fantasy: 118
    is_animation: 98
    is_crime: 205
    is_short: 0
    is_documentary: 45
    is_reality-tv: 0
    is_family: 84
    is_comedy: 486
    is_romance: 178
    

#### It looks like there are a few - so let's remove these columns from the dataframe. We are also going to drop the 'n/a' column as well.


```python
indiv_genre_df.drop(columns=["is_news", "is_adult","is_talk-show","is_game-show", 
                   "is_reality-tv","is_short", "is_n/a"], inplace=True)
```


```python
for col in indiv_genre_df:
    if 'is_' in col:
        print(f"{col}: {indiv_genre_df[col].sum()}")
    else:
        continue
    
```

    is_war: 18
    is_sport: 31
    is_history: 41
    is_western: 11
    is_thriller: 240
    is_music: 7
    is_mystery: 121
    is_action: 394
    is_drama: 670
    is_horror: 159
    is_biography: 112
    is_adventure: 338
    is_sci-fi: 121
    is_fantasy: 118
    is_animation: 98
    is_crime: 205
    is_documentary: 45
    is_family: 84
    is_comedy: 486
    is_romance: 178
    

#### Now we are going to make a list that contains all the column names within the indiv_genre_df


```python
indiv_genre_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>is_war</th>
      <th>is_sport</th>
      <th>is_history</th>
      <th>is_western</th>
      <th>is_thriller</th>
      <th>is_music</th>
      <th>is_mystery</th>
      <th>is_action</th>
      <th>is_drama</th>
      <th>...</th>
      <th>is_comedy</th>
      <th>is_romance</th>
      <th>tconst</th>
      <th>original_title</th>
      <th>genres</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt1298650</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>Action,Adventure,Fantasy</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>6.350639e+08</td>
      <td>154.667286</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Dark Phoenix</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt6565702</td>
      <td>Dark Phoenix</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
      <td>-2.002376e+08</td>
      <td>-57.210757</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Avengers: Age of Ultron</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt2395427</td>
      <td>Avengers: Age of Ultron</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>1.072414e+09</td>
      <td>324.384139</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Avengers: Infinity War</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt4154756</td>
      <td>Avengers: Infinity War</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>300000000.0</td>
      <td>678815482.0</td>
      <td>2.048134e+09</td>
      <td>1.748134e+09</td>
      <td>582.711400</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Justice League</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt0974015</td>
      <td>Justice League</td>
      <td>Action,Adventure,Fantasy</td>
      <td>300000000.0</td>
      <td>229024295.0</td>
      <td>6.559452e+08</td>
      <td>3.559452e+08</td>
      <td>118.648403</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 29 columns</p>
</div>




```python
col_list = [col for col in indiv_genre_df.columns if "is_" in col]
col_list
```




    ['is_war',
     'is_sport',
     'is_history',
     'is_western',
     'is_thriller',
     'is_music',
     'is_mystery',
     'is_action',
     'is_drama',
     'is_horror',
     'is_biography',
     'is_adventure',
     'is_sci-fi',
     'is_fantasy',
     'is_animation',
     'is_crime',
     'is_documentary',
     'is_family',
     'is_comedy',
     'is_romance']




```python
test_col = col_list[0]
print(test_col)
indiv_genre_df.groupby(test_col).get_group(1)['tot_profit'].agg(['mean', 'median'])
indiv_genre_df[indiv_genre_df[test_col] == 1].sort_values('tot_profit',ascending=False)
```

    is_war
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>is_war</th>
      <th>is_sport</th>
      <th>is_history</th>
      <th>is_western</th>
      <th>is_thriller</th>
      <th>is_music</th>
      <th>is_mystery</th>
      <th>is_action</th>
      <th>is_drama</th>
      <th>...</th>
      <th>is_comedy</th>
      <th>is_romance</th>
      <th>tconst</th>
      <th>original_title</th>
      <th>genres</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>186</td>
      <td>300: Rise of an Empire</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt1253863</td>
      <td>300: Rise of an Empire</td>
      <td>Action,Fantasy,War</td>
      <td>110000000.0</td>
      <td>106580051.0</td>
      <td>330780051.0</td>
      <td>220780051.0</td>
      <td>200.709137</td>
    </tr>
    <tr>
      <td>266</td>
      <td>Fury</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt2713180</td>
      <td>Fury</td>
      <td>Action,Drama,War</td>
      <td>80000000.0</td>
      <td>85817906.0</td>
      <td>210315681.0</td>
      <td>130315681.0</td>
      <td>162.894601</td>
    </tr>
    <tr>
      <td>685</td>
      <td>Dear John</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>tt0989757</td>
      <td>Dear John</td>
      <td>Drama,Romance,War</td>
      <td>25000000.0</td>
      <td>80014842.0</td>
      <td>142033509.0</td>
      <td>117033509.0</td>
      <td>468.134036</td>
    </tr>
    <tr>
      <td>291</td>
      <td>War Horse</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt1568911</td>
      <td>War Horse</td>
      <td>Drama,History,War</td>
      <td>70000000.0</td>
      <td>79883359.0</td>
      <td>156815529.0</td>
      <td>86815529.0</td>
      <td>124.022184</td>
    </tr>
    <tr>
      <td>822</td>
      <td>The Book Thief</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt0816442</td>
      <td>The Book Thief</td>
      <td>Drama,War</td>
      <td>19000000.0</td>
      <td>21488481.0</td>
      <td>76086711.0</td>
      <td>57086711.0</td>
      <td>300.456374</td>
    </tr>
    <tr>
      <td>1099</td>
      <td>Incendies</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt1255953</td>
      <td>Incendies</td>
      <td>Drama,Mystery,War</td>
      <td>6800000.0</td>
      <td>6857096.0</td>
      <td>16038343.0</td>
      <td>9238343.0</td>
      <td>135.857985</td>
    </tr>
    <tr>
      <td>1242</td>
      <td>Indivisible</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt6512428</td>
      <td>Indivisible</td>
      <td>Drama,War</td>
      <td>2700000.0</td>
      <td>3511417.0</td>
      <td>3588305.0</td>
      <td>888305.0</td>
      <td>32.900185</td>
    </tr>
    <tr>
      <td>1327</td>
      <td>Camp X-Ray</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt2994190</td>
      <td>Camp X-Ray</td>
      <td>Drama,War</td>
      <td>1000000.0</td>
      <td>9837.0</td>
      <td>101053.0</td>
      <td>-898947.0</td>
      <td>-89.894700</td>
    </tr>
    <tr>
      <td>1295</td>
      <td>Amigo</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt1562847</td>
      <td>Amigo</td>
      <td>Drama,War</td>
      <td>1500000.0</td>
      <td>184705.0</td>
      <td>184705.0</td>
      <td>-1315295.0</td>
      <td>-87.686333</td>
    </tr>
    <tr>
      <td>965</td>
      <td>For Greater Glory: The True Story of Cristiada</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt1566501</td>
      <td>For Greater Glory: The True Story of Cristiada</td>
      <td>Drama,History,War</td>
      <td>12000000.0</td>
      <td>5669081.0</td>
      <td>10026255.0</td>
      <td>-1973745.0</td>
      <td>-16.447875</td>
    </tr>
    <tr>
      <td>1236</td>
      <td>Fort McCoy</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt1282046</td>
      <td>Fort McCoy</td>
      <td>Drama,History,War</td>
      <td>3000000.0</td>
      <td>78948.0</td>
      <td>78948.0</td>
      <td>-2921052.0</td>
      <td>-97.368400</td>
    </tr>
    <tr>
      <td>1052</td>
      <td>Beneath Hill 60</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt1418646</td>
      <td>Beneath Hill 60</td>
      <td>Drama,History,War</td>
      <td>9000000.0</td>
      <td>0.0</td>
      <td>3440939.0</td>
      <td>-5559061.0</td>
      <td>-61.767344</td>
    </tr>
    <tr>
      <td>1027</td>
      <td>Coriolanus</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt3313066</td>
      <td>National Theatre Live: Coriolanus</td>
      <td>Drama,History,War</td>
      <td>10000000.0</td>
      <td>749641.0</td>
      <td>2179623.0</td>
      <td>-7820377.0</td>
      <td>-78.203770</td>
    </tr>
    <tr>
      <td>1029</td>
      <td>Cinco de Mayo, La Batalla</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt2553908</td>
      <td>Cinco de Mayo: La batalla</td>
      <td>Drama,History,War</td>
      <td>10000000.0</td>
      <td>173472.0</td>
      <td>173472.0</td>
      <td>-9826528.0</td>
      <td>-98.265280</td>
    </tr>
    <tr>
      <td>900</td>
      <td>Rock the Kasbah</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>tt3164256</td>
      <td>Rock the Kasbah</td>
      <td>Comedy,Music,War</td>
      <td>15000000.0</td>
      <td>3020665.0</td>
      <td>3386153.0</td>
      <td>-11613847.0</td>
      <td>-77.425647</td>
    </tr>
    <tr>
      <td>968</td>
      <td>5 Days of War</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt1486193</td>
      <td>5 Days of War</td>
      <td>Action,Drama,War</td>
      <td>12000000.0</td>
      <td>17479.0</td>
      <td>87793.0</td>
      <td>-11912207.0</td>
      <td>-99.268392</td>
    </tr>
    <tr>
      <td>759</td>
      <td>Bitter Harvest</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>tt3182620</td>
      <td>Bitter Harvest</td>
      <td>Drama,Romance,War</td>
      <td>21000000.0</td>
      <td>557241.0</td>
      <td>606162.0</td>
      <td>-20393838.0</td>
      <td>-97.113514</td>
    </tr>
    <tr>
      <td>532</td>
      <td>There Be Dragons</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>tt1316616</td>
      <td>There Be Dragons</td>
      <td>Biography,Drama,War</td>
      <td>36000000.0</td>
      <td>1069334.0</td>
      <td>4020990.0</td>
      <td>-31979010.0</td>
      <td>-88.830583</td>
    </tr>
  </tbody>
</table>
<p>18 rows × 29 columns</p>
</div>



#### Now let's make an empty dictionary called total_profits. For each column name in col_list, we will make a new key in total_profits, with the column name being the key, and the value will append all tot_profits of rows where row = 1 in that specific column.




```python
total_profits = {}
for col in col_list:
    total_profits[col] = indiv_genre_df.groupby(col).get_group(1).reset_index()['tot_profit']

```

#### We then turn that dictionary into a dataframe called profit_df


```python
profit_df = pd.DataFrame(total_profits)
profit_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>is_war</th>
      <th>is_sport</th>
      <th>is_history</th>
      <th>is_western</th>
      <th>is_thriller</th>
      <th>is_music</th>
      <th>is_mystery</th>
      <th>is_action</th>
      <th>is_drama</th>
      <th>is_horror</th>
      <th>is_biography</th>
      <th>is_adventure</th>
      <th>is_sci-fi</th>
      <th>is_fantasy</th>
      <th>is_animation</th>
      <th>is_crime</th>
      <th>is_documentary</th>
      <th>is_family</th>
      <th>is_comedy</th>
      <th>is_romance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>220780051.0</td>
      <td>1.122470e+09</td>
      <td>-33396176.0</td>
      <td>-14997885.0</td>
      <td>5.796209e+08</td>
      <td>1.099200e+09</td>
      <td>74357408.0</td>
      <td>6.350639e+08</td>
      <td>112459006.0</td>
      <td>341514650.0</td>
      <td>397938302.0</td>
      <td>6.350639e+08</td>
      <td>-2.002376e+08</td>
      <td>635063875.0</td>
      <td>3.264772e+08</td>
      <td>9.848463e+08</td>
      <td>70720921.0</td>
      <td>767003568.0</td>
      <td>326477240.0</td>
      <td>161040419.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>130315681.0</td>
      <td>9.852782e+07</td>
      <td>349837368.0</td>
      <td>349948323.0</td>
      <td>8.094391e+08</td>
      <td>3.026656e+08</td>
      <td>160650494.0</td>
      <td>-2.002376e+08</td>
      <td>161040419.0</td>
      <td>351530715.0</td>
      <td>289870414.0</td>
      <td>-2.002376e+08</td>
      <td>1.072414e+09</td>
      <td>355945209.0</td>
      <td>1.042521e+09</td>
      <td>1.328723e+09</td>
      <td>60230839.0</td>
      <td>825491110.0</td>
      <td>439213485.0</td>
      <td>192239672.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>86815529.0</td>
      <td>-1.218109e+07</td>
      <td>70720921.0</td>
      <td>72525156.0</td>
      <td>9.848463e+08</td>
      <td>-8.968068e+06</td>
      <td>277448265.0</td>
      <td>1.072414e+09</td>
      <td>74357408.0</td>
      <td>88202668.0</td>
      <td>-10306691.0</td>
      <td>1.072414e+09</td>
      <td>1.748134e+09</td>
      <td>118151347.0</td>
      <td>8.212152e+08</td>
      <td>6.792360e+08</td>
      <td>74008792.0</td>
      <td>290359051.0</td>
      <td>821215193.0</td>
      <td>210650574.0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>-31979010.0</td>
      <td>1.635915e+08</td>
      <td>-79448583.0</td>
      <td>-33485675.0</td>
      <td>9.105270e+08</td>
      <td>3.555268e+07</td>
      <td>86856088.0</td>
      <td>1.748134e+09</td>
      <td>168902025.0</td>
      <td>-7365642.0</td>
      <td>70720921.0</td>
      <td>1.748134e+09</td>
      <td>7.778100e+06</td>
      <td>617500281.0</td>
      <td>8.688795e+08</td>
      <td>5.051635e+08</td>
      <td>-13632971.0</td>
      <td>452220086.0</td>
      <td>868879522.0</td>
      <td>199680778.0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>117033509.0</td>
      <td>6.130084e+07</td>
      <td>86815529.0</td>
      <td>217276928.0</td>
      <td>1.328723e+09</td>
      <td>-5.068194e+06</td>
      <td>28673154.0</td>
      <td>3.559452e+08</td>
      <td>47784.0</td>
      <td>141521247.0</td>
      <td>302665550.0</td>
      <td>3.559452e+08</td>
      <td>8.900694e+08</td>
      <td>767003568.0</td>
      <td>5.435883e+08</td>
      <td>4.106634e+08</td>
      <td>-9406527.0</td>
      <td>437234314.0</td>
      <td>543588329.0</td>
      <td>90805525.0</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>665</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>-61445.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>666</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>374149.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>667</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>114822.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>668</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>-23453.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>669</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>-4416.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>670 rows × 20 columns</p>
</div>



#### Here we are pulling out the median of each column by chaining describe() and .loc['50%']


```python
profit_df.describe().loc['mean'].sort_values(ascending=False)
```




    is_animation      2.682845e+08
    is_sci-fi         2.619044e+08
    is_adventure      2.502832e+08
    is_music          2.027985e+08
    is_action         1.762202e+08
    is_fantasy        1.581110e+08
    is_family         1.503116e+08
    is_comedy         9.643817e+07
    is_thriller       8.612681e+07
    is_mystery        6.751390e+07
    is_sport          6.285834e+07
    is_biography      6.213725e+07
    is_horror         5.781108e+07
    is_crime          5.613160e+07
    is_romance        5.105321e+07
    is_western        5.105298e+07
    is_drama          5.041892e+07
    is_history        4.830813e+07
    is_war            2.866357e+07
    is_documentary    1.175151e+07
    Name: mean, dtype: float64




```python
profit_df.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>is_war</th>
      <th>is_sport</th>
      <th>is_history</th>
      <th>is_western</th>
      <th>is_thriller</th>
      <th>is_music</th>
      <th>is_mystery</th>
      <th>is_action</th>
      <th>is_drama</th>
      <th>is_horror</th>
      <th>is_biography</th>
      <th>is_adventure</th>
      <th>is_sci-fi</th>
      <th>is_fantasy</th>
      <th>is_animation</th>
      <th>is_crime</th>
      <th>is_documentary</th>
      <th>is_family</th>
      <th>is_comedy</th>
      <th>is_romance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>count</td>
      <td>1.800000e+01</td>
      <td>3.100000e+01</td>
      <td>4.100000e+01</td>
      <td>1.100000e+01</td>
      <td>2.400000e+02</td>
      <td>7.000000e+00</td>
      <td>1.210000e+02</td>
      <td>3.940000e+02</td>
      <td>6.700000e+02</td>
      <td>1.590000e+02</td>
      <td>1.120000e+02</td>
      <td>3.380000e+02</td>
      <td>1.210000e+02</td>
      <td>1.180000e+02</td>
      <td>9.800000e+01</td>
      <td>2.050000e+02</td>
      <td>4.500000e+01</td>
      <td>8.400000e+01</td>
      <td>4.860000e+02</td>
      <td>1.780000e+02</td>
    </tr>
    <tr>
      <td>mean</td>
      <td>2.866357e+07</td>
      <td>6.285834e+07</td>
      <td>4.830813e+07</td>
      <td>5.105298e+07</td>
      <td>8.612681e+07</td>
      <td>2.027985e+08</td>
      <td>6.751390e+07</td>
      <td>1.762202e+08</td>
      <td>5.041892e+07</td>
      <td>5.781108e+07</td>
      <td>6.213725e+07</td>
      <td>2.502832e+08</td>
      <td>2.619044e+08</td>
      <td>1.581110e+08</td>
      <td>2.682845e+08</td>
      <td>5.613160e+07</td>
      <td>1.175151e+07</td>
      <td>1.503116e+08</td>
      <td>9.643817e+07</td>
      <td>5.105321e+07</td>
    </tr>
    <tr>
      <td>std</td>
      <td>6.749072e+07</td>
      <td>2.063058e+08</td>
      <td>8.576800e+07</td>
      <td>1.217093e+08</td>
      <td>1.691911e+08</td>
      <td>4.109609e+08</td>
      <td>9.245466e+07</td>
      <td>2.702076e+08</td>
      <td>1.117940e+08</td>
      <td>9.052261e+07</td>
      <td>1.191460e+08</td>
      <td>3.018875e+08</td>
      <td>3.512185e+08</td>
      <td>2.416574e+08</td>
      <td>2.681278e+08</td>
      <td>1.413167e+08</td>
      <td>3.053817e+07</td>
      <td>2.273314e+08</td>
      <td>1.681997e+08</td>
      <td>7.736713e+07</td>
    </tr>
    <tr>
      <td>min</td>
      <td>-3.197901e+07</td>
      <td>-2.121325e+07</td>
      <td>-7.944858e+07</td>
      <td>-3.348568e+07</td>
      <td>-5.033500e+07</td>
      <td>-8.968068e+06</td>
      <td>-1.977277e+07</td>
      <td>-2.002376e+08</td>
      <td>-7.944858e+07</td>
      <td>-3.292904e+07</td>
      <td>-3.442146e+07</td>
      <td>-2.002376e+08</td>
      <td>-2.002376e+08</td>
      <td>-6.953398e+07</td>
      <td>-1.104502e+08</td>
      <td>-5.033500e+07</td>
      <td>-1.363297e+07</td>
      <td>-1.104502e+08</td>
      <td>-1.069000e+08</td>
      <td>-3.494610e+07</td>
    </tr>
    <tr>
      <td>25%</td>
      <td>-9.324990e+06</td>
      <td>-6.295874e+06</td>
      <td>-8.383647e+06</td>
      <td>-1.139016e+07</td>
      <td>-4.895682e+05</td>
      <td>-3.532879e+06</td>
      <td>1.360665e+06</td>
      <td>7.660605e+06</td>
      <td>-2.075244e+06</td>
      <td>2.417755e+05</td>
      <td>-4.985789e+06</td>
      <td>2.854137e+07</td>
      <td>1.355328e+07</td>
      <td>8.252677e+06</td>
      <td>5.214399e+07</td>
      <td>-3.924991e+06</td>
      <td>-5.562910e+05</td>
      <td>1.451086e+07</td>
      <td>2.302288e+06</td>
      <td>6.155980e+05</td>
    </tr>
    <tr>
      <td>50%</td>
      <td>-1.644520e+06</td>
      <td>1.659955e+06</td>
      <td>1.182779e+07</td>
      <td>-7.495500e+05</td>
      <td>3.102218e+07</td>
      <td>-1.794702e+06</td>
      <td>3.678539e+07</td>
      <td>7.024338e+07</td>
      <td>1.292354e+07</td>
      <td>2.800587e+07</td>
      <td>2.113212e+07</td>
      <td>1.367544e+08</td>
      <td>1.236173e+08</td>
      <td>4.823342e+07</td>
      <td>1.941399e+08</td>
      <td>1.731787e+07</td>
      <td>1.495262e+06</td>
      <td>5.896908e+07</td>
      <td>3.334312e+07</td>
      <td>2.080049e+07</td>
    </tr>
    <tr>
      <td>75%</td>
      <td>4.512462e+07</td>
      <td>3.052326e+07</td>
      <td>8.870275e+07</td>
      <td>3.748605e+07</td>
      <td>1.006796e+08</td>
      <td>1.691091e+08</td>
      <td>8.685609e+07</td>
      <td>2.309313e+08</td>
      <td>5.791563e+07</td>
      <td>7.680511e+07</td>
      <td>7.932608e+07</td>
      <td>3.858897e+08</td>
      <td>3.690761e+08</td>
      <td>1.842345e+08</td>
      <td>3.957044e+08</td>
      <td>6.594536e+07</td>
      <td>6.962436e+06</td>
      <td>1.822006e+08</td>
      <td>1.120915e+08</td>
      <td>7.394852e+07</td>
    </tr>
    <tr>
      <td>max</td>
      <td>2.207801e+08</td>
      <td>1.122470e+09</td>
      <td>3.498374e+08</td>
      <td>3.499483e+08</td>
      <td>1.328723e+09</td>
      <td>1.099200e+09</td>
      <td>5.064643e+08</td>
      <td>1.748134e+09</td>
      <td>1.122470e+09</td>
      <td>6.624580e+08</td>
      <td>8.399853e+08</td>
      <td>1.748134e+09</td>
      <td>1.748134e+09</td>
      <td>1.099200e+09</td>
      <td>1.086336e+09</td>
      <td>1.328723e+09</td>
      <td>1.516858e+08</td>
      <td>1.099200e+09</td>
      <td>1.086336e+09</td>
      <td>5.309981e+08</td>
    </tr>
  </tbody>
</table>
</div>




```python
profit_df = profit_df.reindex(profit_df.mean().sort_values(ascending=False).index, axis=1)
```


```python
profit_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>is_animation</th>
      <th>is_sci-fi</th>
      <th>is_adventure</th>
      <th>is_music</th>
      <th>is_action</th>
      <th>is_fantasy</th>
      <th>is_family</th>
      <th>is_comedy</th>
      <th>is_thriller</th>
      <th>is_mystery</th>
      <th>is_sport</th>
      <th>is_biography</th>
      <th>is_horror</th>
      <th>is_crime</th>
      <th>is_romance</th>
      <th>is_western</th>
      <th>is_drama</th>
      <th>is_history</th>
      <th>is_war</th>
      <th>is_documentary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>3.264772e+08</td>
      <td>-2.002376e+08</td>
      <td>6.350639e+08</td>
      <td>1.099200e+09</td>
      <td>6.350639e+08</td>
      <td>635063875.0</td>
      <td>767003568.0</td>
      <td>326477240.0</td>
      <td>5.796209e+08</td>
      <td>74357408.0</td>
      <td>1.122470e+09</td>
      <td>397938302.0</td>
      <td>341514650.0</td>
      <td>9.848463e+08</td>
      <td>161040419.0</td>
      <td>-14997885.0</td>
      <td>112459006.0</td>
      <td>-33396176.0</td>
      <td>220780051.0</td>
      <td>70720921.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1.042521e+09</td>
      <td>1.072414e+09</td>
      <td>-2.002376e+08</td>
      <td>3.026656e+08</td>
      <td>-2.002376e+08</td>
      <td>355945209.0</td>
      <td>825491110.0</td>
      <td>439213485.0</td>
      <td>8.094391e+08</td>
      <td>160650494.0</td>
      <td>9.852782e+07</td>
      <td>289870414.0</td>
      <td>351530715.0</td>
      <td>1.328723e+09</td>
      <td>192239672.0</td>
      <td>349948323.0</td>
      <td>161040419.0</td>
      <td>349837368.0</td>
      <td>130315681.0</td>
      <td>60230839.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>8.212152e+08</td>
      <td>1.748134e+09</td>
      <td>1.072414e+09</td>
      <td>-8.968068e+06</td>
      <td>1.072414e+09</td>
      <td>118151347.0</td>
      <td>290359051.0</td>
      <td>821215193.0</td>
      <td>9.848463e+08</td>
      <td>277448265.0</td>
      <td>-1.218109e+07</td>
      <td>-10306691.0</td>
      <td>88202668.0</td>
      <td>6.792360e+08</td>
      <td>210650574.0</td>
      <td>72525156.0</td>
      <td>74357408.0</td>
      <td>70720921.0</td>
      <td>86815529.0</td>
      <td>74008792.0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>8.688795e+08</td>
      <td>7.778100e+06</td>
      <td>1.748134e+09</td>
      <td>3.555268e+07</td>
      <td>1.748134e+09</td>
      <td>617500281.0</td>
      <td>452220086.0</td>
      <td>868879522.0</td>
      <td>9.105270e+08</td>
      <td>86856088.0</td>
      <td>1.635915e+08</td>
      <td>70720921.0</td>
      <td>-7365642.0</td>
      <td>5.051635e+08</td>
      <td>199680778.0</td>
      <td>-33485675.0</td>
      <td>168902025.0</td>
      <td>-79448583.0</td>
      <td>-31979010.0</td>
      <td>-13632971.0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>5.435883e+08</td>
      <td>8.900694e+08</td>
      <td>3.559452e+08</td>
      <td>-5.068194e+06</td>
      <td>3.559452e+08</td>
      <td>767003568.0</td>
      <td>437234314.0</td>
      <td>543588329.0</td>
      <td>1.328723e+09</td>
      <td>28673154.0</td>
      <td>6.130084e+07</td>
      <td>302665550.0</td>
      <td>141521247.0</td>
      <td>4.106634e+08</td>
      <td>90805525.0</td>
      <td>217276928.0</td>
      <td>47784.0</td>
      <td>86815529.0</td>
      <td>117033509.0</td>
      <td>-9406527.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(20,20))
sns.set(style='darkgrid', font_scale=1.75)
ax = sns.barplot(data=profit_df, orient='h', ci=68)
ax.set_xlabel('Net Profit', fontsize=40)
ax.set_ylabel('Individual Genres', fontsize=40)
ax.set_title('Average Net Profit for Individual Genres', fontsize=50, weight='bold')
ax.set_yticklabels(ax.get_yticklabels(), fontsize=30)
ax.set_xticklabels(ax.get_xticklabels(),rotation=15, fontsize=20, ha='right')

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '${x:,.0f}'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
```


![png](output_229_0.png)


## Answer:

#### Here we see that out of all movies originating from the United States between the years 2010 and 2020 (that were within our datasets), in terms of genres that have the best Average Net profit - Sci-fi, Adventure, and Animation seem to be leading the pack with Action, Fantasy, and Family somewhat close behind.
**Recommendation** - Make a movie using one or combining multiple of the top genres in order to generate a higher net profit. Don't forget, the more money you put into production budget, the more you will receive in net profit.


```python
plt.figure(figsize=(10,10))
sns.set(style='darkgrid', font_scale=2)
sns.swarmplot(orient='h', size=2, color='white',data=profit_df)
ax = sns.violinplot(data=profit_df, orient='h', inner='quartiles', scale='width')
ax.set_title('Average Net Profit for Individual Genres', fontsize=30, weight='bold')
ax.set_ylabel('Individual Genres', fontsize=20)
ax.set_xlabel('Net Profit', fontsize=20)
ax.set_xticklabels(ax.get_xticklabels(), rotation=15, ha='right')
ax.set_yticklabels(ax.get_yticklabels())

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '${x:,.0f}'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
```

    C:\Users\Lewis\anaconda3\envs\learn-env\lib\site-packages\seaborn\categorical.py:1324: RuntimeWarning: invalid value encountered in less
      off_low = points < low_gutter
    C:\Users\Lewis\anaconda3\envs\learn-env\lib\site-packages\seaborn\categorical.py:1328: RuntimeWarning: invalid value encountered in greater
      off_high = points > high_gutter
    


![png](output_232_1.png)


#### Above is a violin plot with boxplot markers. Because the KDE on almost all these genres seems to be skewed to the right (all have long right tails) their mean values will be greater than their median values.


```python
plt.figure(figsize=(20,20))
sns.set(style='darkgrid', font_scale=2.5)
#ax = sns.swarmplot(orient='h',data=profit_df)
ax = sns.boxplot(data=profit_df, orient='h')
ax.set_title('Average Net Profit for Individual Genres', fontsize=50, weight='bold')
ax.set_ylabel('Individual Genres', fontsize=40)
ax.set_xlabel('Net Profit', fontsize=40)
ax.set_xticklabels(ax.get_xticklabels(), fontsize=20, rotation=15, ha='right')

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '${x:,.0f}'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
```


![png](output_234_0.png)


#### The boxplot allows us to see values considered as outliers outside the plots themselves. This visualization allows us to view the median lines much easier.

### Average Production Budget for Individual Genres Within the past 10 years.


```python
prod_buds = {}
for col in col_list:
    prod_buds[col] = indiv_genre_df.groupby(col).get_group(1).reset_index()['production_budget']

```


```python
prod_budget_df = pd.DataFrame(prod_buds)
```


```python
prod_budget_df = prod_budget_df.reindex(prod_budget_df.mean().sort_values(ascending=False).index, axis=1)
```


```python
plt.figure(figsize=(20,20))
sns.set(style='darkgrid', font_scale=1.75)
ax = sns.barplot(data=prod_budget_df, orient='h', ci=68)
ax.set_xlabel('Production Budget', fontsize=30)
ax.set_ylabel('Individual Genres', fontsize=30)
ax.set_title('Average Production Budget for Individual Genres', fontsize=40, weight='bold')
ax.set_yticklabels(ax.get_yticklabels(), fontsize=20)
ax.set_xticklabels(ax.get_xticklabels(),rotation=15, fontsize=20, ha='right')

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '${x:,.0f}'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
```


![png](output_240_0.png)



```python
plt.figure(figsize=(20,20))
sns.set(style='darkgrid', font_scale=1.75)
ax = sns.boxplot(data=prod_budget_df, orient='h')
ax.set_xlabel('Production Budget', fontsize=30)
ax.set_ylabel('Individual Genres', fontsize=30)
ax.set_title('Average Production Budget for Individual Genres', fontsize=40, weight='bold')
ax.set_yticklabels(ax.get_yticklabels(), fontsize=20)
ax.set_xticklabels(ax.get_xticklabels(),rotation=15, fontsize=20, ha='right')

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '${x:,.0f}'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
```


![png](output_241_0.png)


### Now that we've tried visualizing the tot_profit and production budget, let's visualize the percentage of worldwide profit based off of production budget for each genre.


```python
perc_profits = {}
for col in col_list:
    perc_profits[col] = indiv_genre_df.groupby(col).get_group(1).reset_index()['ROI']
```


```python
perc_profits_df = pd.DataFrame(perc_profits)
perc_profits_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>is_war</th>
      <th>is_sport</th>
      <th>is_history</th>
      <th>is_western</th>
      <th>is_thriller</th>
      <th>is_music</th>
      <th>is_mystery</th>
      <th>is_action</th>
      <th>is_drama</th>
      <th>is_horror</th>
      <th>is_biography</th>
      <th>is_adventure</th>
      <th>is_sci-fi</th>
      <th>is_fantasy</th>
      <th>is_animation</th>
      <th>is_crime</th>
      <th>is_documentary</th>
      <th>is_family</th>
      <th>is_comedy</th>
      <th>is_romance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>200.709137</td>
      <td>748.313273</td>
      <td>-21.407805</td>
      <td>-5.453776</td>
      <td>193.206974</td>
      <td>686.999816</td>
      <td>40.193194</td>
      <td>154.667286</td>
      <td>53.551908</td>
      <td>179.744553</td>
      <td>294.769113</td>
      <td>154.667286</td>
      <td>-57.210757</td>
      <td>154.667286</td>
      <td>125.568169</td>
      <td>393.938507</td>
      <td>74.443075</td>
      <td>306.801427</td>
      <td>125.568169</td>
      <td>84.758115</td>
    </tr>
    <tr>
      <td>1</td>
      <td>162.894601</td>
      <td>151.581268</td>
      <td>233.224912</td>
      <td>349.948323</td>
      <td>294.341491</td>
      <td>360.316131</td>
      <td>123.577303</td>
      <td>-57.210757</td>
      <td>84.758115</td>
      <td>197.489166</td>
      <td>289.870414</td>
      <td>-57.210757</td>
      <td>324.384139</td>
      <td>118.648403</td>
      <td>521.260356</td>
      <td>699.327786</td>
      <td>158.502208</td>
      <td>412.745555</td>
      <td>204.285342</td>
      <td>174.763338</td>
    </tr>
    <tr>
      <td>2</td>
      <td>124.022184</td>
      <td>-20.301812</td>
      <td>74.443075</td>
      <td>80.583507</td>
      <td>393.938507</td>
      <td>-12.811526</td>
      <td>221.958612</td>
      <td>324.384139</td>
      <td>40.193194</td>
      <td>58.801779</td>
      <td>-10.306691</td>
      <td>324.384139</td>
      <td>582.711400</td>
      <td>42.964126</td>
      <td>410.607596</td>
      <td>388.134853</td>
      <td>255.202731</td>
      <td>145.179525</td>
      <td>410.607596</td>
      <td>210.650574</td>
    </tr>
    <tr>
      <td>3</td>
      <td>-88.830583</td>
      <td>327.183044</td>
      <td>-88.276203</td>
      <td>-79.727798</td>
      <td>455.263490</td>
      <td>64.641227</td>
      <td>69.484870</td>
      <td>582.711400</td>
      <td>93.834458</td>
      <td>-4.910428</td>
      <td>74.443075</td>
      <td>582.711400</td>
      <td>2.828400</td>
      <td>247.000112</td>
      <td>434.439761</td>
      <td>404.130763</td>
      <td>-54.531884</td>
      <td>226.110043</td>
      <td>434.439761</td>
      <td>210.190293</td>
    </tr>
    <tr>
      <td>4</td>
      <td>468.134036</td>
      <td>122.601670</td>
      <td>124.022184</td>
      <td>620.791223</td>
      <td>699.327786</td>
      <td>-90.503464</td>
      <td>28.110935</td>
      <td>118.648403</td>
      <td>0.026547</td>
      <td>145.898193</td>
      <td>360.316131</td>
      <td>118.648403</td>
      <td>356.027765</td>
      <td>306.801427</td>
      <td>271.794164</td>
      <td>328.530754</td>
      <td>-40.897943</td>
      <td>240.238634</td>
      <td>271.794164</td>
      <td>113.506906</td>
    </tr>
  </tbody>
</table>
</div>




```python
perc_profits_df = perc_profits_df.reindex(perc_profits_df.mean().sort_values(ascending=False).index, axis=1)
```


```python
plt.figure(figsize=(10,10))
sns.set(style='darkgrid', font_scale=1.75)
ax = sns.barplot(data=perc_profits_df, orient='h', ci=68)
ax.set_xlabel('Return On Investment', fontsize=25)
ax.set_ylabel('Individual Genres', fontsize=25)
ax.set_title('Average Return On Investment for Individual Genres', weight='bold', fontsize=30)
ax.set_xticklabels(ax.get_xticklabels())
ax.set_yticklabels(ax.get_yticklabels())

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '{x:,.0f}%'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
```


![png](output_246_0.png)


#### According to this barplot, the genres with the highest ROI are mystery, thriller, and horror.


```python
plt.figure(figsize=(20,10))
sns.set(style='darkgrid', font_scale=2)
ax = sns.swarmplot(data=perc_profits_df)
#ax = sns.violinplot(data=perc_profits_df, orient='h', inner='quartiles', scale='width')
ax.set_title('Average Return On Investment for Individual Genres', fontsize=30, weight='bold')
ax.set_ylabel('Individual Genres', fontsize=25)
ax.set_xlabel('Return On Investment', fontsize=25)
ax.set_xticklabels(ax.get_xticklabels(), rotation=70, ha='right')
ax.set_yticklabels(ax.get_yticklabels())

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '{x:,.0f}%'
tick = mtick.StrMethodFormatter(fmt)
ax.yaxis.set_major_formatter(tick);
```

    C:\Users\Lewis\anaconda3\envs\learn-env\lib\site-packages\seaborn\categorical.py:1324: RuntimeWarning: invalid value encountered in less
      off_low = points < low_gutter
    C:\Users\Lewis\anaconda3\envs\learn-env\lib\site-packages\seaborn\categorical.py:1328: RuntimeWarning: invalid value encountered in greater
      off_high = points > high_gutter
    


![png](output_248_1.png)


#### According to the graph above, we can see that those three genres (horror, mystery, and thriller) have some major outliers compared to the rest of the genres.

## Grouped Genres

#### Let's join these two DataFrames to get movie budgets and genres together


```python
imdb_title_basics_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt1699720</td>
      <td>!Women Art Revolution</td>
      <td>Women Art Revolution</td>
      <td>2010</td>
      <td>83.0</td>
      <td>Documentary</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt2346170</td>
      <td>#1 Serial Killer</td>
      <td>#1 Serial Killer</td>
      <td>2013</td>
      <td>87.0</td>
      <td>Horror</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt3120962</td>
      <td>#5</td>
      <td>#5</td>
      <td>2013</td>
      <td>68.0</td>
      <td>Biography,Comedy,Fantasy</td>
    </tr>
  </tbody>
</table>
</div>




```python
tn_movie_budget_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>2009</td>
      <td>Avatar</td>
      <td>425000000.0</td>
      <td>760507625.0</td>
      <td>2.776345e+09</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2</td>
      <td>2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <td>2</td>
      <td>3</td>
      <td>2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
  </tbody>
</table>
</div>



#### These are not filtered  by region = 'US'. Originally had that, but there were too few grouped genre categories.


```python
new_df = pysqldf("""SELECT i.tconst, i.genres, i.runtime_minutes, t.*
                    FROM tn_movie_budget_df t
                    JOIN imdb_title_basics_clean i
                    USING(primary_title)
                    WHERE (release_date > 2009 AND release_date <= 2020)
                           AND (worldwide_gross != 0)
                           AND (production_budget > 0);""")
```


```python
new_df = new_df.drop(columns='id')
```


```python
new_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>genres</th>
      <th>runtime_minutes</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt1298650</td>
      <td>Action,Adventure,Fantasy</td>
      <td>136.0</td>
      <td>2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt6565702</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>113.0</td>
      <td>2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt2395427</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>141.0</td>
      <td>2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
    </tr>
  </tbody>
</table>
</div>



#### According to the line below, there are 83 grouped genres that pertain to only 1 movie each.


```python
(new_df['genres'].value_counts() == 1).sum()
```




    83



#### Let's make a new column in our new_df called 'tot_profit' that calculates the total profit for each movie by subtracting the production budget from the worldwide gross and 'ROI' that calculates the total percentage between production_budget and worldwide_gross


```python
new_df['tot_profit'] = new_df['worldwide_gross'] - new_df['production_budget']
```


```python
new_df['ROI'] = (new_df['worldwide_gross']/new_df['production_budget']) * 100
```


```python
new_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>genres</th>
      <th>runtime_minutes</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt1298650</td>
      <td>Action,Adventure,Fantasy</td>
      <td>136.0</td>
      <td>2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>6.350639e+08</td>
      <td>254.667286</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt6565702</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>113.0</td>
      <td>2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
      <td>-2.002376e+08</td>
      <td>42.789243</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt2395427</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>141.0</td>
      <td>2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>1.072414e+09</td>
      <td>424.384139</td>
    </tr>
    <tr>
      <td>3</td>
      <td>tt4154756</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>149.0</td>
      <td>2018</td>
      <td>Avengers: Infinity War</td>
      <td>300000000.0</td>
      <td>678815482.0</td>
      <td>2.048134e+09</td>
      <td>1.748134e+09</td>
      <td>682.711400</td>
    </tr>
    <tr>
      <td>4</td>
      <td>tt0974015</td>
      <td>Action,Adventure,Fantasy</td>
      <td>120.0</td>
      <td>2017</td>
      <td>Justice League</td>
      <td>300000000.0</td>
      <td>229024295.0</td>
      <td>6.559452e+08</td>
      <td>3.559452e+08</td>
      <td>218.648403</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(new_df)
```




    2126



#### Make a list called group_cols that appends every unique value within the genres column 


```python
group_cols = [col for col in new_df['genres'].unique()]
```


```python
new_dict = {}
gb = new_df.groupby('genres')
# gb.get_group('Action,Animation,Comedy')['tot_profit']
for genre in group_cols:
    new_dict[genre] = gb.get_group(genre).reset_index()['tot_profit']
```

#### Turn dictionary into a dataframe


```python
grouped_profit_df = pd.DataFrame(new_dict)
```


```python
grouped_profit_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Action,Adventure,Fantasy</th>
      <th>Action,Adventure,Sci-Fi</th>
      <th>Action,Adventure,Thriller</th>
      <th>Action,Thriller</th>
      <th>Action,Adventure,Western</th>
      <th>Adventure,Animation,Comedy</th>
      <th>Adventure,Family,Fantasy</th>
      <th>Adventure,Fantasy</th>
      <th>Action,Crime,Thriller</th>
      <th>Action,Adventure,Comedy</th>
      <th>...</th>
      <th>Comedy,Fantasy,Musical</th>
      <th>Documentary,Drama,History</th>
      <th>Horror,Sci-Fi</th>
      <th>Documentary,History,War</th>
      <th>Comedy,Horror,Mystery</th>
      <th>Adventure,Horror</th>
      <th>Adventure,Biography,Documentary</th>
      <th>Comedy,Fantasy,Thriller</th>
      <th>Action,Biography,Documentary</th>
      <th>Biography,Documentary,Music</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>635063875.0</td>
      <td>-2.002376e+08</td>
      <td>579620923.0</td>
      <td>809439099.0</td>
      <td>-14997885.0</td>
      <td>326477240.0</td>
      <td>767003568.0</td>
      <td>710366855.0</td>
      <td>9.848463e+08</td>
      <td>439213485.0</td>
      <td>...</td>
      <td>-1997564.0</td>
      <td>-1159068.0</td>
      <td>-1614407.0</td>
      <td>1092308.0</td>
      <td>-1263137.0</td>
      <td>11931420.0</td>
      <td>-711243.0</td>
      <td>-486267.0</td>
      <td>-498178.0</td>
      <td>-79944.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>355945209.0</td>
      <td>1.072414e+09</td>
      <td>112459006.0</td>
      <td>296168316.0</td>
      <td>72525156.0</td>
      <td>821215193.0</td>
      <td>825491110.0</td>
      <td>695577621.0</td>
      <td>1.328723e+09</td>
      <td>666980024.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>-1323738.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2</td>
      <td>118151347.0</td>
      <td>1.748134e+09</td>
      <td>910526981.0</td>
      <td>70720921.0</td>
      <td>-3282693.0</td>
      <td>868879522.0</td>
      <td>290359051.0</td>
      <td>2687603.0</td>
      <td>5.051635e+08</td>
      <td>600867516.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>5139730.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>3</td>
      <td>617500281.0</td>
      <td>7.778100e+06</td>
      <td>155355920.0</td>
      <td>212249198.0</td>
      <td>NaN</td>
      <td>543588329.0</td>
      <td>452220086.0</td>
      <td>NaN</td>
      <td>1.134531e+08</td>
      <td>110328374.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>4</td>
      <td>558241137.0</td>
      <td>8.900694e+08</td>
      <td>179115534.0</td>
      <td>102878928.0</td>
      <td>NaN</td>
      <td>360155383.0</td>
      <td>622402853.0</td>
      <td>NaN</td>
      <td>-4.590954e+06</td>
      <td>493144660.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 274 columns</p>
</div>




```python
len(grouped_profit_df.columns)
```




    274




```python
grouped_profit_df = grouped_profit_df.reindex(grouped_profit_df.mean().sort_values(ascending=False).index, axis=1)
```

#### Filtering out Grouped Genres that have a movie count of less than 15.


```python
for col in grouped_profit_df:
    #if the count within the column is less than 15, drop the column from the dataframe.
    if grouped_profit_df[col].describe()['count'] < 15:
        grouped_profit_df = grouped_profit_df.drop(columns=col)
    else:
        continue
grouped_profit_df.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Action,Adventure,Sci-Fi</th>
      <th>Action,Adventure,Animation</th>
      <th>Adventure,Animation,Comedy</th>
      <th>Action,Adventure,Comedy</th>
      <th>Action,Adventure,Fantasy</th>
      <th>Action,Crime,Thriller</th>
      <th>Action,Thriller</th>
      <th>N/A</th>
      <th>Horror,Mystery,Thriller</th>
      <th>Action,Adventure,Drama</th>
      <th>...</th>
      <th>Documentary</th>
      <th>Biography,Comedy,Drama</th>
      <th>Drama,Thriller</th>
      <th>Comedy,Drama,Romance</th>
      <th>Comedy,Drama</th>
      <th>Drama</th>
      <th>Action,Drama,Thriller</th>
      <th>Comedy,Crime,Drama</th>
      <th>Action,Crime,Drama</th>
      <th>Crime,Drama,Thriller</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>count</td>
      <td>5.300000e+01</td>
      <td>1.700000e+01</td>
      <td>6.900000e+01</td>
      <td>2.500000e+01</td>
      <td>3.500000e+01</td>
      <td>2.400000e+01</td>
      <td>2.000000e+01</td>
      <td>1.900000e+01</td>
      <td>3.700000e+01</td>
      <td>2.900000e+01</td>
      <td>...</td>
      <td>1.020000e+02</td>
      <td>2.000000e+01</td>
      <td>2.900000e+01</td>
      <td>6.700000e+01</td>
      <td>6.500000e+01</td>
      <td>1.740000e+02</td>
      <td>1.800000e+01</td>
      <td>1.500000e+01</td>
      <td>4.600000e+01</td>
      <td>3.100000e+01</td>
    </tr>
    <tr>
      <td>mean</td>
      <td>4.760434e+08</td>
      <td>3.222576e+08</td>
      <td>2.947854e+08</td>
      <td>2.705272e+08</td>
      <td>2.304667e+08</td>
      <td>1.408030e+08</td>
      <td>1.246050e+08</td>
      <td>1.132652e+08</td>
      <td>8.779402e+07</td>
      <td>8.767324e+07</td>
      <td>...</td>
      <td>3.901065e+07</td>
      <td>3.524653e+07</td>
      <td>2.987420e+07</td>
      <td>2.814452e+07</td>
      <td>2.813802e+07</td>
      <td>2.628883e+07</td>
      <td>2.574403e+07</td>
      <td>2.474509e+07</td>
      <td>2.312713e+07</td>
      <td>1.682720e+07</td>
    </tr>
    <tr>
      <td>std</td>
      <td>4.197517e+08</td>
      <td>2.601914e+08</td>
      <td>2.973197e+08</td>
      <td>2.824711e+08</td>
      <td>2.578842e+08</td>
      <td>3.361813e+08</td>
      <td>1.953436e+08</td>
      <td>2.048105e+08</td>
      <td>8.649343e+07</td>
      <td>1.361975e+08</td>
      <td>...</td>
      <td>1.045770e+08</td>
      <td>7.135562e+07</td>
      <td>7.484742e+07</td>
      <td>5.135097e+07</td>
      <td>4.351037e+07</td>
      <td>6.450728e+07</td>
      <td>3.112574e+07</td>
      <td>4.186392e+07</td>
      <td>4.720956e+07</td>
      <td>3.639174e+07</td>
    </tr>
    <tr>
      <td>min</td>
      <td>-2.002376e+08</td>
      <td>-2.935140e+07</td>
      <td>-3.585151e+07</td>
      <td>-1.069000e+08</td>
      <td>-2.664387e+07</td>
      <td>-5.033500e+07</td>
      <td>-2.490888e+07</td>
      <td>-7.944858e+07</td>
      <td>-2.365923e+07</td>
      <td>-6.448372e+07</td>
      <td>...</td>
      <td>-3.165737e+07</td>
      <td>-3.212508e+07</td>
      <td>-2.365923e+07</td>
      <td>-7.944858e+07</td>
      <td>-2.345527e+07</td>
      <td>-7.944858e+07</td>
      <td>-1.680049e+07</td>
      <td>-6.617054e+06</td>
      <td>-4.322557e+07</td>
      <td>-3.421347e+07</td>
    </tr>
    <tr>
      <td>25%</td>
      <td>1.679166e+08</td>
      <td>1.142478e+08</td>
      <td>6.471712e+07</td>
      <td>1.585218e+07</td>
      <td>2.106865e+07</td>
      <td>-6.889198e+06</td>
      <td>4.180598e+06</td>
      <td>4.631540e+05</td>
      <td>3.074923e+07</td>
      <td>-6.496851e+06</td>
      <td>...</td>
      <td>-1.286535e+06</td>
      <td>5.349608e+06</td>
      <td>-4.925361e+06</td>
      <td>-6.664030e+05</td>
      <td>-9.848440e+05</td>
      <td>-3.866558e+06</td>
      <td>-1.306412e+06</td>
      <td>-1.653651e+06</td>
      <td>-7.881548e+06</td>
      <td>-7.823122e+06</td>
    </tr>
    <tr>
      <td>50%</td>
      <td>3.690761e+08</td>
      <td>3.775991e+08</td>
      <td>1.788479e+08</td>
      <td>2.391462e+08</td>
      <td>1.433886e+08</td>
      <td>1.668186e+07</td>
      <td>5.428272e+07</td>
      <td>2.200463e+07</td>
      <td>6.336420e+07</td>
      <td>4.784919e+07</td>
      <td>...</td>
      <td>4.231260e+06</td>
      <td>1.161142e+07</td>
      <td>4.221211e+06</td>
      <td>5.733666e+06</td>
      <td>1.214162e+07</td>
      <td>1.862998e+06</td>
      <td>1.516817e+07</td>
      <td>2.434356e+06</td>
      <td>7.271262e+06</td>
      <td>3.622980e+05</td>
    </tr>
    <tr>
      <td>75%</td>
      <td>7.051664e+08</td>
      <td>4.380684e+08</td>
      <td>3.975198e+08</td>
      <td>4.392135e+08</td>
      <td>3.721435e+08</td>
      <td>1.171323e+08</td>
      <td>1.544579e+08</td>
      <td>1.433371e+08</td>
      <td>1.200103e+08</td>
      <td>1.214997e+08</td>
      <td>...</td>
      <td>3.730142e+07</td>
      <td>3.793973e+07</td>
      <td>2.383071e+07</td>
      <td>4.066859e+07</td>
      <td>4.517894e+07</td>
      <td>3.582118e+07</td>
      <td>5.121079e+07</td>
      <td>3.119188e+07</td>
      <td>4.222076e+07</td>
      <td>3.568979e+07</td>
    </tr>
    <tr>
      <td>max</td>
      <td>1.748134e+09</td>
      <td>1.042521e+09</td>
      <td>1.122470e+09</td>
      <td>8.744962e+08</td>
      <td>9.868946e+08</td>
      <td>1.328723e+09</td>
      <td>8.094391e+08</td>
      <td>6.792360e+08</td>
      <td>2.980001e+08</td>
      <td>5.406446e+08</td>
      <td>...</td>
      <td>8.254911e+08</td>
      <td>2.990344e+08</td>
      <td>3.182667e+08</td>
      <td>2.154125e+08</td>
      <td>1.635498e+08</td>
      <td>3.975198e+08</td>
      <td>7.755159e+07</td>
      <td>1.208577e+08</td>
      <td>1.941042e+08</td>
      <td>1.208577e+08</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 34 columns</p>
</div>




```python
grouped_profit_df = grouped_profit_df.drop(columns='N/A')
```


```python
plt.figure(figsize=(20,20))
sns.set(style='darkgrid', font_scale=2)
ax = sns.barplot(data=grouped_profit_df, orient='h', ci=68)
ax.set_xlabel('Grouped Net Profit', fontsize=40)
ax.set_ylabel('Grouped Genres', fontsize=40)
ax.set_title('Average Net Profit of Grouped Genres', fontsize=50, weight='bold')
ax.set_xticklabels(ax.get_xticklabels(), rotation=15, fontsize=30, ha='right')
ax.set_yticklabels(ax.get_yticklabels(), fontsize=30)
#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '${x:,.0f}'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
# ax.set(xlim=(0, 600000000))
# ax.set_xticks(range(0, 600000000, 50000000));
```


![png](output_276_0.png)


#### According to the graph, the grouped genre [Action, Adventure, Scifi] is leading in terms of average net profit. 
**Recommendation** - If you decide to use multiple genres, try and aim for one of the top 5 grouped genres on this list in order to have the greatest chance of getting the highest net profit.


```python
plt.figure(figsize=(20,20))
sns.set(style='darkgrid', font_scale=1.75)
sns.swarmplot(data=grouped_profit_df, orient='h', size=3, color='white')
ax = sns.violinplot(data=grouped_profit_df, inner='quartiles', orient='h', scale='width')
ax.set_title('Average Net Profit for Grouped Genres', fontsize=50, weight='bold')
ax.set_xlabel('Average Net Profit', fontsize=40)
ax.set_ylabel('Grouped Genres', fontsize=40)
ax.set_xticklabels(ax.get_xticklabels(), rotation=15, fontsize=30, ha='right')
ax.set_yticklabels(ax.get_yticklabels(), fontsize=30)
#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '${x:,.0f}'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
#ax.set(xlim=(0, 600000000))
#ax.set_xticks(range(0, 600000000, 50000000));
```

    C:\Users\Lewis\anaconda3\envs\learn-env\lib\site-packages\seaborn\categorical.py:1324: RuntimeWarning: invalid value encountered in less
      off_low = points < low_gutter
    C:\Users\Lewis\anaconda3\envs\learn-env\lib\site-packages\seaborn\categorical.py:1328: RuntimeWarning: invalid value encountered in greater
      off_high = points > high_gutter
    


![png](output_278_1.png)



```python
prod_dict = {}
gb = new_df.groupby('genres')
# gb.get_group('Action,Animation,Comedy')['tot_profit']
for genre in group_cols:
    prod_dict[genre] = gb.get_group(genre).reset_index()['production_budget']
```


```python
grp_prod_bud_df = pd.DataFrame(prod_dict)
```


```python
grp_prod_bud_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Action,Adventure,Fantasy</th>
      <th>Action,Adventure,Sci-Fi</th>
      <th>Action,Adventure,Thriller</th>
      <th>Action,Thriller</th>
      <th>Action,Adventure,Western</th>
      <th>Adventure,Animation,Comedy</th>
      <th>Adventure,Family,Fantasy</th>
      <th>Adventure,Fantasy</th>
      <th>Action,Crime,Thriller</th>
      <th>Action,Adventure,Comedy</th>
      <th>...</th>
      <th>Comedy,Fantasy,Musical</th>
      <th>Documentary,Drama,History</th>
      <th>Horror,Sci-Fi</th>
      <th>Documentary,History,War</th>
      <th>Comedy,Horror,Mystery</th>
      <th>Adventure,Horror</th>
      <th>Adventure,Biography,Documentary</th>
      <th>Comedy,Fantasy,Thriller</th>
      <th>Action,Biography,Documentary</th>
      <th>Biography,Documentary,Music</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>410600000.0</td>
      <td>350000000.0</td>
      <td>300000000.0</td>
      <td>275000000.0</td>
      <td>275000000.0</td>
      <td>260000000.0</td>
      <td>250000000.0</td>
      <td>250000000.0</td>
      <td>250000000.0</td>
      <td>215000000.0</td>
      <td>...</td>
      <td>2000000.0</td>
      <td>1900000.0</td>
      <td>1900000.0</td>
      <td>1500000.0</td>
      <td>1500000.0</td>
      <td>1000000.0</td>
      <td>1000000.0</td>
      <td>900000.0</td>
      <td>500000.0</td>
      <td>100000.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>300000000.0</td>
      <td>330600000.0</td>
      <td>210000000.0</td>
      <td>120000000.0</td>
      <td>90000000.0</td>
      <td>200000000.0</td>
      <td>200000000.0</td>
      <td>250000000.0</td>
      <td>190000000.0</td>
      <td>180000000.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>1500000.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2</td>
      <td>275000000.0</td>
      <td>300000000.0</td>
      <td>200000000.0</td>
      <td>95000000.0</td>
      <td>4500000.0</td>
      <td>200000000.0</td>
      <td>200000000.0</td>
      <td>195000000.0</td>
      <td>125000000.0</td>
      <td>170000000.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>500000.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>3</td>
      <td>250000000.0</td>
      <td>275000000.0</td>
      <td>125000000.0</td>
      <td>92000000.0</td>
      <td>NaN</td>
      <td>200000000.0</td>
      <td>200000000.0</td>
      <td>NaN</td>
      <td>77000000.0</td>
      <td>135000000.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>4</td>
      <td>230000000.0</td>
      <td>250000000.0</td>
      <td>125000000.0</td>
      <td>70000000.0</td>
      <td>NaN</td>
      <td>200000000.0</td>
      <td>180000000.0</td>
      <td>NaN</td>
      <td>70000000.0</td>
      <td>130000000.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 274 columns</p>
</div>




```python
for col in grp_prod_bud_df:
    if grp_prod_bud_df[col].describe()['count'] < 15:
        grp_prod_bud_df = grp_prod_bud_df.drop(columns=col)
    else:
        continue
```


```python
#grp_prod_bud_df = grp_prod_bud_df.drop(columns='N/A')
```


```python
grp_prod_bud_df = grp_prod_bud_df.reindex(grp_prod_bud_df.mean().sort_values(ascending=False).index, axis=1)
```


```python
plt.figure(figsize=(10,10))
sns.set(style='darkgrid', font_scale=1.50)
ax = sns.barplot(data=grp_prod_bud_df, orient='h', ci=68)
ax.set_xlabel('Grouped Production Budget', fontsize=20)
ax.set_ylabel('Grouped Genres', fontsize=20)
ax.set_title('Average Production Budget of Grouped Genres', fontsize=30, weight='bold')
ax.set_xticklabels(ax.get_xticklabels(), rotation=15, ha='right')
ax.set_yticklabels(ax.get_yticklabels())
#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '${x:,.0f}'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
```


![png](output_285_0.png)


#### According to the graph, the grouped genre ['Action, Adventure, Scifi] is also leading in terms of highest average production budget.


```python
plt.figure(figsize=(30,15))
sns.set(style='darkgrid', font_scale=1.70)
sns.swarmplot(data=grp_prod_bud_df, size=4, color='white')
ax = sns.violinplot(data=grp_prod_bud_df, inner='quartiles', ci=68, scale='width')
ax.set_xlabel('Grouped Production Budget', fontsize=30)
ax.set_ylabel('Grouped Genres', fontsize=30)
ax.set_title('Average Production Budget of Grouped Genres', fontsize=40, weight='bold')
ax.set_xticklabels(ax.get_xticklabels(), rotation=70, ha='right')
ax.set_yticklabels(ax.get_yticklabels())
#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '${x:,.0f}'
tick = mtick.StrMethodFormatter(fmt)
ax.yaxis.set_major_formatter(tick);
```


![png](output_287_0.png)



```python
perc_dict = {}
gb = new_df.groupby('genres')
for genre in group_cols:
    perc_dict[genre] = gb.get_group(genre)['ROI']
```


```python
perc_group_prof_df = pd.DataFrame(perc_dict)
```


```python
for col in perc_group_prof_df:
    #If the count in the column is less than 15, drop the column from the dataframe.
    if perc_group_prof_df[col].describe()['count'] < 15:
        perc_group_prof_df = perc_group_prof_df.drop(columns=col)
    else:
        continue
perc_group_prof_df.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Action,Adventure,Fantasy</th>
      <th>Action,Adventure,Sci-Fi</th>
      <th>Action,Thriller</th>
      <th>Adventure,Animation,Comedy</th>
      <th>Action,Crime,Thriller</th>
      <th>Action,Adventure,Comedy</th>
      <th>Action,Adventure,Drama</th>
      <th>Action,Adventure,Animation</th>
      <th>Documentary</th>
      <th>Drama,Romance</th>
      <th>...</th>
      <th>Comedy,Drama</th>
      <th>Biography,Drama,History</th>
      <th>Comedy,Crime,Drama</th>
      <th>Drama,Mystery,Thriller</th>
      <th>Biography,Drama</th>
      <th>Biography,Comedy,Drama</th>
      <th>Drama,Horror,Mystery</th>
      <th>Crime,Drama,Thriller</th>
      <th>Horror,Mystery,Thriller</th>
      <th>Horror,Thriller</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>count</td>
      <td>35.000000</td>
      <td>53.000000</td>
      <td>20.000000</td>
      <td>69.000000</td>
      <td>24.000000</td>
      <td>25.000000</td>
      <td>29.000000</td>
      <td>17.000000</td>
      <td>102.000000</td>
      <td>55.000000</td>
      <td>...</td>
      <td>65.000000</td>
      <td>26.000000</td>
      <td>15.000000</td>
      <td>16.000000</td>
      <td>20.000000</td>
      <td>20.000000</td>
      <td>17.000000</td>
      <td>31.000000</td>
      <td>37.000000</td>
      <td>29.000000</td>
    </tr>
    <tr>
      <td>mean</td>
      <td>237.471678</td>
      <td>384.854737</td>
      <td>287.844517</td>
      <td>390.069162</td>
      <td>208.655245</td>
      <td>365.788420</td>
      <td>193.585689</td>
      <td>410.098117</td>
      <td>252.180665</td>
      <td>403.856237</td>
      <td>...</td>
      <td>342.967132</td>
      <td>317.016431</td>
      <td>185.874299</td>
      <td>392.859794</td>
      <td>345.256328</td>
      <td>364.832221</td>
      <td>654.146860</td>
      <td>165.412317</td>
      <td>2760.670329</td>
      <td>561.868623</td>
    </tr>
    <tr>
      <td>std</td>
      <td>148.547171</td>
      <td>209.151061</td>
      <td>250.971763</td>
      <td>306.082512</td>
      <td>197.178385</td>
      <td>321.252705</td>
      <td>147.448595</td>
      <td>323.595224</td>
      <td>352.343559</td>
      <td>574.038039</td>
      <td>...</td>
      <td>474.171264</td>
      <td>258.488996</td>
      <td>183.091699</td>
      <td>419.915350</td>
      <td>368.769049</td>
      <td>380.520222</td>
      <td>798.796814</td>
      <td>161.399081</td>
      <td>6809.418525</td>
      <td>877.306989</td>
    </tr>
    <tr>
      <td>min</td>
      <td>8.516405</td>
      <td>42.789243</td>
      <td>0.364464</td>
      <td>40.247480</td>
      <td>0.013800</td>
      <td>0.355600</td>
      <td>0.048446</td>
      <td>2.161997</td>
      <td>0.013800</td>
      <td>0.364464</td>
      <td>...</td>
      <td>0.453745</td>
      <td>0.412313</td>
      <td>0.359733</td>
      <td>0.687000</td>
      <td>29.289173</td>
      <td>0.584596</td>
      <td>12.221429</td>
      <td>2.087755</td>
      <td>4.104056</td>
      <td>0.501200</td>
    </tr>
    <tr>
      <td>25%</td>
      <td>119.588469</td>
      <td>237.044727</td>
      <td>122.211721</td>
      <td>186.577085</td>
      <td>48.774117</td>
      <td>137.743279</td>
      <td>86.837066</td>
      <td>211.655794</td>
      <td>28.661027</td>
      <td>56.420905</td>
      <td>...</td>
      <td>51.976215</td>
      <td>125.874132</td>
      <td>61.585020</td>
      <td>103.851912</td>
      <td>97.278369</td>
      <td>109.768576</td>
      <td>137.743279</td>
      <td>36.064146</td>
      <td>407.492300</td>
      <td>102.742067</td>
    </tr>
    <tr>
      <td>50%</td>
      <td>218.648403</td>
      <td>354.498168</td>
      <td>240.974408</td>
      <td>296.921458</td>
      <td>159.168159</td>
      <td>365.867558</td>
      <td>155.974192</td>
      <td>395.228987</td>
      <td>145.071336</td>
      <td>173.590865</td>
      <td>...</td>
      <td>212.177811</td>
      <td>215.058427</td>
      <td>152.635417</td>
      <td>193.405014</td>
      <td>210.822850</td>
      <td>221.947609</td>
      <td>331.669132</td>
      <td>104.048095</td>
      <td>779.593740</td>
      <td>170.261803</td>
    </tr>
    <tr>
      <td>75%</td>
      <td>303.573042</td>
      <td>524.551428</td>
      <td>358.690570</td>
      <td>456.004629</td>
      <td>308.330760</td>
      <td>430.384813</td>
      <td>271.408512</td>
      <td>426.913444</td>
      <td>282.036908</td>
      <td>479.393251</td>
      <td>...</td>
      <td>428.445353</td>
      <td>421.829464</td>
      <td>259.215815</td>
      <td>558.970452</td>
      <td>438.732029</td>
      <td>478.952781</td>
      <td>830.538000</td>
      <td>262.248301</td>
      <td>2500.205200</td>
      <td>329.828533</td>
    </tr>
    <tr>
      <td>max</td>
      <td>716.809150</td>
      <td>859.371700</td>
      <td>856.890320</td>
      <td>1568.021855</td>
      <td>799.327786</td>
      <td>1381.078609</td>
      <td>687.475292</td>
      <td>1444.091235</td>
      <td>1967.778200</td>
      <td>2717.924114</td>
      <td>...</td>
      <td>2575.494167</td>
      <td>927.086864</td>
      <td>622.985020</td>
      <td>1226.517525</td>
      <td>1246.422667</td>
      <td>1400.149735</td>
      <td>2976.130200</td>
      <td>592.169624</td>
      <td>41656.474000</td>
      <td>3042.219367</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 34 columns</p>
</div>




```python
perc_group_prof_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Action,Adventure,Fantasy</th>
      <th>Action,Adventure,Sci-Fi</th>
      <th>Action,Thriller</th>
      <th>Adventure,Animation,Comedy</th>
      <th>Action,Crime,Thriller</th>
      <th>Action,Adventure,Comedy</th>
      <th>Action,Adventure,Drama</th>
      <th>Action,Adventure,Animation</th>
      <th>Documentary</th>
      <th>Drama,Romance</th>
      <th>...</th>
      <th>Comedy,Drama</th>
      <th>Biography,Drama,History</th>
      <th>Comedy,Crime,Drama</th>
      <th>Drama,Mystery,Thriller</th>
      <th>Biography,Drama</th>
      <th>Biography,Comedy,Drama</th>
      <th>Drama,Horror,Mystery</th>
      <th>Crime,Drama,Thriller</th>
      <th>Horror,Mystery,Thriller</th>
      <th>Horror,Thriller</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>254.667286</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>1</td>
      <td>NaN</td>
      <td>42.789243</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2</td>
      <td>NaN</td>
      <td>424.384139</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>3</td>
      <td>NaN</td>
      <td>682.711400</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>4</td>
      <td>218.648403</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 34 columns</p>
</div>




```python
perc_group_prof_df = perc_group_prof_df.reindex(perc_group_prof_df.mean().sort_values(ascending=False).index, axis=1)
```


```python
plt.figure(figsize=(10,10))
sns.set(style='darkgrid', font_scale=1.5)
ax = sns.barplot(data=perc_group_prof_df, orient='h', ci=68)
ax.set_xlabel('Grouped Return On Investment', fontsize=25)
ax.set_ylabel('Grouped Genres', fontsize=25)
ax.set_title('Average Return On Investment of Grouped Genres', fontsize=30, weight='bold')

ax.set_xticklabels(ax.get_xticklabels(), rotation=15, ha='right')
ax.set_yticklabels(ax.get_yticklabels())
#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '{x:,.0f}%'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
# ax.set(xlim=(0, 600000000))
# ax.set_xticks(range(0, 600000000, 50000000));
```


![png](output_293_0.png)


# Question 3: Who are the top 25 U.S. directors that have the highest average net profit within the past 20 years?

#### Our files of interest for this question would be: imdb_title_df, imdb_name_df, tn_movie_budget_df, imdb_title_crew, and imdb_title_basics_df


```python
imdb_title_rating_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>averagerating</th>
      <th>numvotes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt10356526</td>
      <td>8.3</td>
      <td>31</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt10384606</td>
      <td>8.9</td>
      <td>559</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt1042974</td>
      <td>6.4</td>
      <td>20</td>
    </tr>
  </tbody>
</table>
</div>




```python
imdb_title_crew_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>directors</th>
      <th>writers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt0285252</td>
      <td>nm0899854</td>
      <td>nm0899854</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt0438973</td>
      <td>NaN</td>
      <td>nm0175726,nm1802864</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt0462036</td>
      <td>nm1940585</td>
      <td>nm1940585</td>
    </tr>
  </tbody>
</table>
</div>




```python
imdb_title_basics_clean.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt0063540</td>
      <td>Sunghursh</td>
      <td>Sunghursh</td>
      <td>2013</td>
      <td>175.0</td>
      <td>Action,Crime,Drama</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt0066787</td>
      <td>One Day Before the Rainy Season</td>
      <td>Ashad Ka Ek Din</td>
      <td>2019</td>
      <td>114.0</td>
      <td>Biography,Drama</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt0069049</td>
      <td>The Other Side of the Wind</td>
      <td>The Other Side of the Wind</td>
      <td>2018</td>
      <td>122.0</td>
      <td>Drama</td>
    </tr>
    <tr>
      <td>3</td>
      <td>tt0069204</td>
      <td>Sabse Bada Sukh</td>
      <td>Sabse Bada Sukh</td>
      <td>2018</td>
      <td>0.0</td>
      <td>Comedy,Drama</td>
    </tr>
    <tr>
      <td>4</td>
      <td>tt0100275</td>
      <td>The Wandering Soap Opera</td>
      <td>La Telenovela Errante</td>
      <td>2017</td>
      <td>80.0</td>
      <td>Comedy,Drama,Fantasy</td>
    </tr>
  </tbody>
</table>
</div>




```python
tn_movie_budget_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>2009</td>
      <td>Avatar</td>
      <td>425000000.0</td>
      <td>760507625.0</td>
      <td>2.776345e+09</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2</td>
      <td>2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
    </tr>
    <tr>
      <td>2</td>
      <td>3</td>
      <td>2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
    </tr>
    <tr>
      <td>3</td>
      <td>4</td>
      <td>2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
    </tr>
    <tr>
      <td>4</td>
      <td>5</td>
      <td>2017</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>317000000.0</td>
      <td>620181382.0</td>
      <td>1.316722e+09</td>
    </tr>
  </tbody>
</table>
</div>




```python
imdb_name_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>nconst</th>
      <th>primary_name</th>
      <th>birth_year</th>
      <th>death_year</th>
      <th>primary_profession</th>
      <th>known_for_titles</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>nm0061671</td>
      <td>Mary Ellen Bauder</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>miscellaneous,production_manager,producer</td>
      <td>tt0837562,tt2398241,tt0844471,tt0118553</td>
    </tr>
    <tr>
      <td>1</td>
      <td>nm0061865</td>
      <td>Joseph Bauer</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>composer,music_department,sound_department</td>
      <td>tt0896534,tt6791238,tt0287072,tt1682940</td>
    </tr>
    <tr>
      <td>2</td>
      <td>nm0062070</td>
      <td>Bruce Baum</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>miscellaneous,actor,writer</td>
      <td>tt1470654,tt0363631,tt0104030,tt0102898</td>
    </tr>
    <tr>
      <td>3</td>
      <td>nm0062195</td>
      <td>Axel Baumann</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>camera_department,cinematographer,art_department</td>
      <td>tt0114371,tt2004304,tt1618448,tt1224387</td>
    </tr>
    <tr>
      <td>4</td>
      <td>nm0062798</td>
      <td>Pete Baxter</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>production_designer,art_department,set_decorator</td>
      <td>tt0452644,tt0452692,tt3458030,tt2178256</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Replacing NaN values in death_year with 0 
imdb_name_df['death_year'].fillna(0, inplace=True)
```


```python
imdb_title_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>ordering</th>
      <th>title</th>
      <th>region</th>
      <th>language</th>
      <th>types</th>
      <th>attributes</th>
      <th>is_original_title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt0369610</td>
      <td>10</td>
      <td>Джурасик свят</td>
      <td>BG</td>
      <td>bg</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt0369610</td>
      <td>11</td>
      <td>Jurashikku warudo</td>
      <td>JP</td>
      <td>NaN</td>
      <td>imdbDisplay</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt0369610</td>
      <td>12</td>
      <td>Jurassic World: O Mundo dos Dinossauros</td>
      <td>BR</td>
      <td>NaN</td>
      <td>imdbDisplay</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>3</td>
      <td>tt0369610</td>
      <td>13</td>
      <td>O Mundo dos Dinossauros</td>
      <td>BR</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>short title</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>4</td>
      <td>tt0369610</td>
      <td>14</td>
      <td>Jurassic World</td>
      <td>FR</td>
      <td>NaN</td>
      <td>imdbDisplay</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



#### Going to make a new dataframe by joining imdb_title_basics, imdb_title_crew_df, tn_movie_budget_df, imdb_title_df, imdb_name_df where we target only US movies, removing null valued rows where worldwide gross <= 0, making sure the director isn't already dead, and setting the release date between 2000 and 2020. Then we group it all by the tconst primary key.


```python
best_dir_df = pysqldf("""SELECT b.primary_title, b.tconst, b.genres,
                         m.production_budget, m.worldwide_gross, n.primary_name
                         FROM imdb_title_basics_clean b
                         JOIN imdb_title_crew_df c
                         USING (tconst)
                         JOIN tn_movie_budget_df m
                         USING (primary_title)
                         JOIN imdb_title_df t
                         USING(tconst)
                         JOIN imdb_name_df n
                         ON (c.directors = n.nconst)
                         WHERE(t.region = 'US')
                               AND (m.worldwide_gross > 0)
                               AND (n.death_year < 1)
                               AND (m.release_date > 1999 AND m.release_date <= 2020)
                               AND (m.release_date = b.start_year)
                         GROUP BY (b.tconst);""")
```


```python
best_dir_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>primary_title</th>
      <th>tconst</th>
      <th>genres</th>
      <th>production_budget</th>
      <th>worldwide_gross</th>
      <th>primary_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Foodfight!</td>
      <td>tt0249516</td>
      <td>Action,Animation,Comedy</td>
      <td>45000000.0</td>
      <td>7.370600e+04</td>
      <td>Lawrence Kasanoff</td>
    </tr>
    <tr>
      <td>1</td>
      <td>The Secret Life of Walter Mitty</td>
      <td>tt0359950</td>
      <td>Adventure,Comedy,Drama</td>
      <td>91000000.0</td>
      <td>1.878612e+08</td>
      <td>Ben Stiller</td>
    </tr>
    <tr>
      <td>2</td>
      <td>A Walk Among the Tombstones</td>
      <td>tt0365907</td>
      <td>Action,Crime,Drama</td>
      <td>28000000.0</td>
      <td>6.210859e+07</td>
      <td>Scott Frank</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Jurassic World</td>
      <td>tt0369610</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>215000000.0</td>
      <td>1.648855e+09</td>
      <td>Colin Trevorrow</td>
    </tr>
    <tr>
      <td>4</td>
      <td>The Rum Diary</td>
      <td>tt0376136</td>
      <td>Comedy,Drama</td>
      <td>45000000.0</td>
      <td>2.154473e+07</td>
      <td>Bruce Robinson</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(best_dir_df)
```




    1097




```python
perc_null(best_dir_df)
```




    primary_title        0.0
    tconst               0.0
    genres               0.0
    production_budget    0.0
    worldwide_gross      0.0
    primary_name         0.0
    dtype: float64




```python
best_dir_df['tot_profit'] = best_dir_df['worldwide_gross'] - best_dir_df['production_budget']
```


```python

```


```python
#Making a list of all the unique names within best_dir_df's primary_name column
unique_dirs = [name for name in best_dir_df['primary_name'].unique()]
```


```python
#Making a dictionary where the key equals the name and the value appends every total_profit made under that name
dir_dict = {}
gb = best_dir_df.groupby('primary_name')
for name in unique_dirs:
    dir_dict[name] = gb.get_group(name).reset_index()['tot_profit']
```


```python
#Changing the dictionary into a dataframe
dir_revenue_df = pd.DataFrame(dir_dict)
```


```python
dir_revenue_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Lawrence Kasanoff</th>
      <th>Ben Stiller</th>
      <th>Scott Frank</th>
      <th>Colin Trevorrow</th>
      <th>Bruce Robinson</th>
      <th>Andrew Stanton</th>
      <th>Jay Roach</th>
      <th>Joe Carnahan</th>
      <th>Ole Bornedal</th>
      <th>Shawn Levy</th>
      <th>...</th>
      <th>Peter Farrelly</th>
      <th>Panos Cosmatos</th>
      <th>Bo Burnham</th>
      <th>Karyn Kusama</th>
      <th>Brett Haley</th>
      <th>Charles Stone III</th>
      <th>Spike Lee</th>
      <th>Andrew Hyatt</th>
      <th>Ari Aster</th>
      <th>Michael Moore</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>-44926294.0</td>
      <td>96861183.0</td>
      <td>34108587.0</td>
      <td>1.433855e+09</td>
      <td>-23455268.0</td>
      <td>7778100.0</td>
      <td>17796502.0</td>
      <td>67241171.0</td>
      <td>68925064.0</td>
      <td>153880341.0</td>
      <td>...</td>
      <td>299034439.0</td>
      <td>-4572344.0</td>
      <td>12341016.0</td>
      <td>-5318904.0</td>
      <td>420962.0</td>
      <td>28527161.0</td>
      <td>78017335.0</td>
      <td>20529498.0</td>
      <td>60133905.0</td>
      <td>1653715.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>NaN</td>
      <td>5348693.0</td>
      <td>NaN</td>
      <td>3.672318e+06</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9907746.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>97269033.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>35672764.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>226756621.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>4</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 709 columns</p>
</div>




```python
dir_revenue_df.mean().sort_values(ascending=False)
```




    Joss Whedon             1.182675e+09
    James Wan               8.712059e+08
    Lee Unkrich             8.688795e+08
    Sam Mendes              7.450740e+08
    Tim Miller              7.430256e+08
                                ...     
    Robert Schwentke       -5.092332e+07
    Xiao Feng              -6.448372e+07
    Andrey Konchalovskiy   -6.953398e+07
    Simon Wells            -1.104502e+08
    Simon Kinberg          -2.002376e+08
    Length: 709, dtype: float64




```python
#Sorting the values according to the mean of every column
dir_revenue_df = dir_revenue_df.reindex(dir_revenue_df.mean().sort_values(ascending=False).index, axis=1)
```


```python
dir_revenue_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Joss Whedon</th>
      <th>James Wan</th>
      <th>Lee Unkrich</th>
      <th>Sam Mendes</th>
      <th>Tim Miller</th>
      <th>Peter Jackson</th>
      <th>Colin Trevorrow</th>
      <th>Patty Jenkins</th>
      <th>Taika Waititi</th>
      <th>Andy Muschietti</th>
      <th>...</th>
      <th>Sngmoo Lee</th>
      <th>Joel Schumacher</th>
      <th>Mario Van Peebles</th>
      <th>Lawrence Kasanoff</th>
      <th>Michael Mann</th>
      <th>Robert Schwentke</th>
      <th>Xiao Feng</th>
      <th>Andrey Konchalovskiy</th>
      <th>Simon Wells</th>
      <th>Simon Kinberg</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1.292936e+09</td>
      <td>2.980001e+08</td>
      <td>868879522.0</td>
      <td>910526981.0</td>
      <td>743025593.0</td>
      <td>767003568.0</td>
      <td>1.433855e+09</td>
      <td>671133378.0</td>
      <td>666980024.0</td>
      <td>662457969.0</td>
      <td>...</td>
      <td>-33485675.0</td>
      <td>-34213468.0</td>
      <td>-38336215.0</td>
      <td>-44926294.0</td>
      <td>-50334996.0</td>
      <td>-50923322.0</td>
      <td>-64483721.0</td>
      <td>-69533984.0</td>
      <td>-110450242.0</td>
      <td>-200237650.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1.072414e+09</td>
      <td>9.868946e+08</td>
      <td>NaN</td>
      <td>579620923.0</td>
      <td>NaN</td>
      <td>710366855.0</td>
      <td>3.672318e+06</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2</td>
      <td>NaN</td>
      <td>1.328723e+09</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>695577621.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>4</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 709 columns</p>
</div>




```python
#Making a new dataframe consisting of the top 25 directors based on average net profit
top_25 = dir_revenue_df.iloc[:,0:24]
top_25.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Joss Whedon</th>
      <th>James Wan</th>
      <th>Lee Unkrich</th>
      <th>Sam Mendes</th>
      <th>Tim Miller</th>
      <th>Peter Jackson</th>
      <th>Colin Trevorrow</th>
      <th>Patty Jenkins</th>
      <th>Taika Waititi</th>
      <th>Andy Muschietti</th>
      <th>...</th>
      <th>James Gunn</th>
      <th>Christopher Nolan</th>
      <th>Alfonso Cuarón</th>
      <th>Robert Stromberg</th>
      <th>Michael Bay</th>
      <th>Dan Scanlon</th>
      <th>Bill Condon</th>
      <th>Brad Bird</th>
      <th>Sam Taylor-Johnson</th>
      <th>Ang Lee</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1.292936e+09</td>
      <td>2.980001e+08</td>
      <td>868879522.0</td>
      <td>910526981.0</td>
      <td>743025593.0</td>
      <td>767003568.0</td>
      <td>1.433855e+09</td>
      <td>671133378.0</td>
      <td>666980024.0</td>
      <td>662457969.0</td>
      <td>...</td>
      <td>600867516.0</td>
      <td>501379375.0</td>
      <td>583698673.0</td>
      <td>578536735.0</td>
      <td>928790543.0</td>
      <td>543588329.0</td>
      <td>-1.984583e+07</td>
      <td>3.662752e+07</td>
      <td>530998101.0</td>
      <td>500912003.0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1.072414e+09</td>
      <td>9.868946e+08</td>
      <td>NaN</td>
      <td>579620923.0</td>
      <td>NaN</td>
      <td>710366855.0</td>
      <td>3.672318e+06</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>809439099.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>55275291.0</td>
      <td>NaN</td>
      <td>1.099200e+09</td>
      <td>1.042521e+09</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2</td>
      <td>NaN</td>
      <td>1.328723e+09</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>695577621.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>675524642.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>894039076.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>349837368.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>385893340.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>4</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 24 columns</p>
</div>




```python
plt.figure(figsize=(10,10))
sns.set(style='darkgrid', font_scale=1.5)
ax = sns.barplot(data=top_25, orient='h', ci=None, palette="GnBu_d")
ax.set_xlabel('Average Net Profit', fontsize=25)
ax.set_ylabel('Directors', fontsize=25)
ax.set_title('Average Net Profit of Top 25 Directors', fontsize=30, weight='bold')

ax.set_xticklabels(ax.get_xticklabels(), rotation=15, ha='right')
ax.set_yticklabels(ax.get_yticklabels())

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '${x:,.0f}'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
```


![png](output_318_0.png)


#### These are the top 25 directors who, on average, have the accrued the highest net profit from directing U.S.  movies within the past 20 years.
**Recommendation** - Hiring one of these directors would be a wise choice for your movie (depending on your genre/grouped genre)

# Conclusion

#### My recommendations for creating Microsoft's first movie would be:
 - invest a lot of money into the production budget!!
 - stick with one of the top genres/grouped genres (action, adventure, sci-fi, fantasy, family, comedy) when it comes to deciding what type of movie you want to make; those movie genres will give you the highest chance of reaping in the greatest rewards.
 - Hire one of the directors from the top 25 based off of what genre/grouped genre the movie will be.

### Appendix


```python
crew_rating = pysqldf("""SELECT *
                         FROM imdb_title_rating_df 
                         JOIN imdb_title_crew_df c
                         USING(tconst)
                         JOIN imdb_name_df i
                         ON(c.directors = i.nconst);""")
```


```python
crew_rating.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>directors</th>
      <th>writers</th>
      <th>nconst</th>
      <th>primary_name</th>
      <th>birth_year</th>
      <th>death_year</th>
      <th>primary_profession</th>
      <th>known_for_titles</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt10356526</td>
      <td>8.3</td>
      <td>31</td>
      <td>nm8353804</td>
      <td>nm3057599,nm4179342</td>
      <td>nm8353804</td>
      <td>Sukh Sanghera</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>director,cinematographer,location_management</td>
      <td>tt10356526,tt8749962,tt9579874,tt5231224</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt1042974</td>
      <td>6.4</td>
      <td>20</td>
      <td>nm1915232</td>
      <td>nm1915232</td>
      <td>nm1915232</td>
      <td>Marcel Grant</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>director,writer,producer</td>
      <td>tt1042974,tt1538868,tt0949872,tt0458519</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt1043726</td>
      <td>4.2</td>
      <td>50352</td>
      <td>nm0001317</td>
      <td>nm0393517,nm0316417,nm0001317,nm1048866</td>
      <td>nm0001317</td>
      <td>Renny Harlin</td>
      <td>1959.0</td>
      <td>0.0</td>
      <td>producer,director,writer</td>
      <td>tt2238032,tt0149261,tt0099423,tt0106582</td>
    </tr>
    <tr>
      <td>3</td>
      <td>tt1060240</td>
      <td>6.5</td>
      <td>21</td>
      <td>nm1926349</td>
      <td>nm1926349</td>
      <td>nm1926349</td>
      <td>Carlos M. Barros</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>director,writer,casting_director</td>
      <td>tt1612376,tt1823187,tt0460544,tt1060240</td>
    </tr>
    <tr>
      <td>4</td>
      <td>tt1069246</td>
      <td>6.2</td>
      <td>326</td>
      <td>nm0868643</td>
      <td>nm2726004,nm0140502,nm1523568,nm0868643</td>
      <td>nm0868643</td>
      <td>Fina Torres</td>
      <td>1951.0</td>
      <td>0.0</td>
      <td>writer,director,producer</td>
      <td>tt0089739,tt0107537,tt2325833,tt1069246</td>
    </tr>
  </tbody>
</table>
</div>




```python
crew_rating['death_year'].isna().value_counts()
```




    False    64996
    Name: death_year, dtype: int64




```python
#Replacing the NaN values with 0
crew_rating['death_year'].fillna(0, inplace=True)
```


```python
#crew_rating now equals all directors who have not yet died
crew_rating = crew_rating[crew_rating['death_year'] < 1]
```


```python
crew_rating.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>directors</th>
      <th>writers</th>
      <th>nconst</th>
      <th>primary_name</th>
      <th>birth_year</th>
      <th>death_year</th>
      <th>primary_profession</th>
      <th>known_for_titles</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt10356526</td>
      <td>8.3</td>
      <td>31</td>
      <td>nm8353804</td>
      <td>nm3057599,nm4179342</td>
      <td>nm8353804</td>
      <td>Sukh Sanghera</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>director,cinematographer,location_management</td>
      <td>tt10356526,tt8749962,tt9579874,tt5231224</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt1042974</td>
      <td>6.4</td>
      <td>20</td>
      <td>nm1915232</td>
      <td>nm1915232</td>
      <td>nm1915232</td>
      <td>Marcel Grant</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>director,writer,producer</td>
      <td>tt1042974,tt1538868,tt0949872,tt0458519</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt1043726</td>
      <td>4.2</td>
      <td>50352</td>
      <td>nm0001317</td>
      <td>nm0393517,nm0316417,nm0001317,nm1048866</td>
      <td>nm0001317</td>
      <td>Renny Harlin</td>
      <td>1959.0</td>
      <td>0.0</td>
      <td>producer,director,writer</td>
      <td>tt2238032,tt0149261,tt0099423,tt0106582</td>
    </tr>
    <tr>
      <td>3</td>
      <td>tt1060240</td>
      <td>6.5</td>
      <td>21</td>
      <td>nm1926349</td>
      <td>nm1926349</td>
      <td>nm1926349</td>
      <td>Carlos M. Barros</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>director,writer,casting_director</td>
      <td>tt1612376,tt1823187,tt0460544,tt1060240</td>
    </tr>
    <tr>
      <td>4</td>
      <td>tt1069246</td>
      <td>6.2</td>
      <td>326</td>
      <td>nm0868643</td>
      <td>nm2726004,nm0140502,nm1523568,nm0868643</td>
      <td>nm0868643</td>
      <td>Fina Torres</td>
      <td>1951.0</td>
      <td>0.0</td>
      <td>writer,director,producer</td>
      <td>tt0089739,tt0107537,tt2325833,tt1069246</td>
    </tr>
  </tbody>
</table>
</div>




```python
crew_rating['death_year'].value_counts()
```




    0.0    64439
    Name: death_year, dtype: int64




```python
dir_rating_df = pysqldf("""SELECT tconst, averagerating, numvotes, primary_name as 'director'
                           FROM crew_rating;""")
```


```python
dir_rating_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>director</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt10356526</td>
      <td>8.3</td>
      <td>31</td>
      <td>Sukh Sanghera</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt1042974</td>
      <td>6.4</td>
      <td>20</td>
      <td>Marcel Grant</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt1043726</td>
      <td>4.2</td>
      <td>50352</td>
      <td>Renny Harlin</td>
    </tr>
    <tr>
      <td>3</td>
      <td>tt1060240</td>
      <td>6.5</td>
      <td>21</td>
      <td>Carlos M. Barros</td>
    </tr>
    <tr>
      <td>4</td>
      <td>tt1069246</td>
      <td>6.2</td>
      <td>326</td>
      <td>Fina Torres</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(imdb_title_basics_df)
```




    45218




```python
imdb_title_basics_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt1699720</td>
      <td>!Women Art Revolution</td>
      <td>Women Art Revolution</td>
      <td>2010</td>
      <td>83.0</td>
      <td>Documentary</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt2346170</td>
      <td>#1 Serial Killer</td>
      <td>#1 Serial Killer</td>
      <td>2013</td>
      <td>87.0</td>
      <td>Horror</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt3120962</td>
      <td>#5</td>
      <td>#5</td>
      <td>2013</td>
      <td>68.0</td>
      <td>Biography,Comedy,Fantasy</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(dir_rating_df)
```




    64439




```python
gen_dir_rating_df = pysqldf('''SELECT genres, d.*
                               FROM imdb_title_basics_df 
                               JOIN dir_rating_df d
                               USING(tconst);''')
```


```python
len(gen_dir_rating_df)
```




    22126




```python
gen_dir_rating_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>genres</th>
      <th>tconst</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>director</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Documentary</td>
      <td>tt1699720</td>
      <td>6.9</td>
      <td>196</td>
      <td>Lynn Hershman-Leeson</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Horror</td>
      <td>tt2346170</td>
      <td>5.6</td>
      <td>40</td>
      <td>Stanley Yung</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Biography,Comedy,Fantasy</td>
      <td>tt3120962</td>
      <td>6.8</td>
      <td>6</td>
      <td>Ricky Bardy</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Making a new dataframe that focuses on the top 5 grouped genres
test_rating_df = pysqldf('''SELECT tconst, averagerating, numvotes, director, genres
                            FROM gen_dir_rating_df
                            WHERE genres = 'Action,Adventure,Comedy' 
                            OR genres = 'Action,Adventure,Sci-Fi' 
                            OR genres = 'Action,Adventure,Fantasy' 
                            OR genres = 'Adventure,Animation,Comedy'
                            OR genres = 'Action,Adventure,Animation';''')
```


```python
test_rating_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>director</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt2106284</td>
      <td>6.8</td>
      <td>1818</td>
      <td>Seth Grossman</td>
      <td>Action,Adventure,Comedy</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt6727598</td>
      <td>5.5</td>
      <td>1173</td>
      <td>Nick Lyon</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt3004572</td>
      <td>2.3</td>
      <td>181</td>
      <td>Kyle Misak</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>3</td>
      <td>tt2439946</td>
      <td>2.5</td>
      <td>1855</td>
      <td>Peter Geiger</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>4</td>
      <td>tt6050658</td>
      <td>5.5</td>
      <td>6</td>
      <td>Lou Qi</td>
      <td>Action,Adventure,Comedy</td>
    </tr>
  </tbody>
</table>
</div>




```python
for col in test_rating_df:
    print(f'{col}: {test_rating_df[col].duplicated().sum()}')
```

    tconst: 0
    averagerating: 348
    numvotes: 30
    director: 65
    genres: 411
    


```python
new_test_rating = pysqldf("""SELECT *
                             FROM test_rating_df
                             WHERE numvotes > 20000
                             GROUP BY (director);""")
```


```python
new_test_rating.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>director</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt1340138</td>
      <td>6.4</td>
      <td>234615</td>
      <td>Alan Taylor</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt2345759</td>
      <td>5.5</td>
      <td>154345</td>
      <td>Alex Kurtzman</td>
      <td>Action,Adventure,Fantasy</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt2404233</td>
      <td>5.4</td>
      <td>95111</td>
      <td>Alex Proyas</td>
      <td>Action,Adventure,Fantasy</td>
    </tr>
    <tr>
      <td>3</td>
      <td>tt0401729</td>
      <td>6.6</td>
      <td>241792</td>
      <td>Andrew Stanton</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>4</td>
      <td>tt3717252</td>
      <td>5.8</td>
      <td>62942</td>
      <td>Anna Foerster</td>
      <td>Action,Adventure,Fantasy</td>
    </tr>
  </tbody>
</table>
</div>




```python
new_test_rating = new_test_rating.sort_values('averagerating',ascending=False).reset_index(drop=True)
```


```python
new_test_rating['genres'].unique()
```




    array(['Action,Adventure,Sci-Fi', 'Adventure,Animation,Comedy',
           'Action,Adventure,Comedy', 'Action,Adventure,Animation',
           'Action,Adventure,Fantasy'], dtype=object)




```python
new_test_rating.loc[new_test_rating['genres'] == 'Action,Adventure,Sci-Fi']
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>director</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt1375666</td>
      <td>8.8</td>
      <td>1841066</td>
      <td>Christopher Nolan</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>3</td>
      <td>tt1392190</td>
      <td>8.1</td>
      <td>780910</td>
      <td>George Miller</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>9</td>
      <td>tt1408101</td>
      <td>7.7</td>
      <td>445535</td>
      <td>J.J. Abrams</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>12</td>
      <td>tt1677720</td>
      <td>7.5</td>
      <td>301660</td>
      <td>Steven Spielberg</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>14</td>
      <td>tt0437086</td>
      <td>7.5</td>
      <td>88207</td>
      <td>Robert Rodriguez</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>15</td>
      <td>tt1951264</td>
      <td>7.5</td>
      <td>575455</td>
      <td>Francis Lawrence</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>16</td>
      <td>tt2250912</td>
      <td>7.5</td>
      <td>426302</td>
      <td>Jon Watts</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>19</td>
      <td>tt1825683</td>
      <td>7.3</td>
      <td>516148</td>
      <td>Ryan Coogler</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>24</td>
      <td>tt2395427</td>
      <td>7.3</td>
      <td>665594</td>
      <td>Joss Whedon</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>27</td>
      <td>tt1300854</td>
      <td>7.2</td>
      <td>692794</td>
      <td>Shane Black</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>30</td>
      <td>tt1392170</td>
      <td>7.2</td>
      <td>795227</td>
      <td>Gary Ross</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>33</td>
      <td>tt2660888</td>
      <td>7.1</td>
      <td>209844</td>
      <td>Justin Lin</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>35</td>
      <td>tt3385516</td>
      <td>7.0</td>
      <td>356556</td>
      <td>Bryan Singer</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>36</td>
      <td>tt0948470</td>
      <td>7.0</td>
      <td>525632</td>
      <td>Marc Webb</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>37</td>
      <td>tt1228705</td>
      <td>7.0</td>
      <td>657690</td>
      <td>Jon Favreau</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>38</td>
      <td>tt0369610</td>
      <td>7.0</td>
      <td>539338</td>
      <td>Colin Trevorrow</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>39</td>
      <td>tt1483013</td>
      <td>7.0</td>
      <td>453966</td>
      <td>Joseph Kosinski</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>43</td>
      <td>tt0458339</td>
      <td>6.9</td>
      <td>668137</td>
      <td>Joe Johnston</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>45</td>
      <td>tt1663662</td>
      <td>6.9</td>
      <td>443667</td>
      <td>Guillermo del Toro</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>47</td>
      <td>tt4701182</td>
      <td>6.9</td>
      <td>94155</td>
      <td>Travis Knight</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>57</td>
      <td>tt0401729</td>
      <td>6.6</td>
      <td>241792</td>
      <td>Andrew Stanton</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>66</td>
      <td>tt1424381</td>
      <td>6.4</td>
      <td>199764</td>
      <td>Nimród Antal</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>67</td>
      <td>tt1340138</td>
      <td>6.4</td>
      <td>234615</td>
      <td>Alan Taylor</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>69</td>
      <td>tt1411250</td>
      <td>6.4</td>
      <td>144821</td>
      <td>David Twohy</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>70</td>
      <td>tt0831387</td>
      <td>6.4</td>
      <td>350687</td>
      <td>Gareth Edwards</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>76</td>
      <td>tt2908446</td>
      <td>6.2</td>
      <td>199710</td>
      <td>Robert Schwentke</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>77</td>
      <td>tt4881806</td>
      <td>6.2</td>
      <td>219125</td>
      <td>J.A. Bayona</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>80</td>
      <td>tt1464540</td>
      <td>6.1</td>
      <td>217762</td>
      <td>D.J. Caruso</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>83</td>
      <td>tt6565702</td>
      <td>6.0</td>
      <td>24451</td>
      <td>Simon Kinberg</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>87</td>
      <td>tt3717490</td>
      <td>6.0</td>
      <td>92013</td>
      <td>Dean Israelite</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>94</td>
      <td>tt2094766</td>
      <td>5.8</td>
      <td>167110</td>
      <td>Justin Kurzel</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>96</td>
      <td>tt1440129</td>
      <td>5.8</td>
      <td>225342</td>
      <td>Peter Berg</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>97</td>
      <td>tt1583421</td>
      <td>5.8</td>
      <td>165536</td>
      <td>Jon M. Chu</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>102</td>
      <td>tt2109248</td>
      <td>5.7</td>
      <td>283486</td>
      <td>Michael Bay</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>104</td>
      <td>tt2557478</td>
      <td>5.6</td>
      <td>89462</td>
      <td>Steven S. DeKnight</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>107</td>
      <td>tt1133985</td>
      <td>5.5</td>
      <td>252281</td>
      <td>Martin Campbell</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
    <tr>
      <td>112</td>
      <td>tt1628841</td>
      <td>5.2</td>
      <td>155344</td>
      <td>Roland Emmerich</td>
      <td>Action,Adventure,Sci-Fi</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Seeing if the cut on the count of numvotes = 20,000 would slice off too much of the data
plt.figure(figsize=(10,5))
ax = new_test_rating['numvotes'].hist(bins='auto')
ax.axvline(20000);
```


![png](output_346_0.png)



```python
new_test_rating['genres'].value_counts()
```




    Action,Adventure,Sci-Fi       37
    Action,Adventure,Fantasy      30
    Adventure,Animation,Comedy    25
    Action,Adventure,Comedy       20
    Action,Adventure,Animation     5
    Name: genres, dtype: int64




```python
plt.figure(figsize=(20, 20))
sns.set(style='darkgrid')
ax = sns.barplot(x='averagerating', y='director',data=new_test_rating.head(25), orient='h', ci=None)
ax.set_xlabel('Average Rating', fontsize=40)
ax.set_ylabel('Directors', fontsize=40)
ax.set_title('Average Ratings of Top 25 Directors Within Targeted Group Genres', fontsize=50, weight='bold')
ax.set_yticklabels(ax.get_yticklabels(), fontsize=30)
ax.set_xticklabels(ax.get_xticklabels(), fontsize=30)

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '{x:,.0f}.0'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
```


![png](output_348_0.png)



```python
#date_df is retrieving all movies made between the years 2010 and 2020
date_df = new_df[(new_df['release_date'] >= 2010) & (new_df['release_date'] < 2020)]
```


```python
date_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>genres</th>
      <th>runtime_minutes</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt1298650</td>
      <td>Action,Adventure,Fantasy</td>
      <td>136.0</td>
      <td>2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>6.350639e+08</td>
      <td>254.667286</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt6565702</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>113.0</td>
      <td>2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
      <td>-2.002376e+08</td>
      <td>42.789243</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt2395427</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>141.0</td>
      <td>2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>1.072414e+09</td>
      <td>424.384139</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Placing all unique column genres from date_df into a variable called date_cols
date_cols = [col for col in date_df['genres'].unique()]
```


```python
#Making a new dictionary where the key is the unqiue genre and the key appends every movie total_profit that is the genre
date_dict = {}
gbdate = date_df.groupby('genres')
for genre in date_cols:
    date_dict[genre] = gbdate.get_group(genre)['tot_profit']

```


```python
#Making a dataframe from the dictionary
group_date_df = pd.DataFrame(date_dict)
```


```python
group_date_df.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Action,Adventure,Fantasy</th>
      <th>Action,Adventure,Sci-Fi</th>
      <th>Action,Adventure,Thriller</th>
      <th>Action,Thriller</th>
      <th>Action,Adventure,Western</th>
      <th>Adventure,Animation,Comedy</th>
      <th>Adventure,Family,Fantasy</th>
      <th>Adventure,Fantasy</th>
      <th>Action,Crime,Thriller</th>
      <th>Action,Adventure,Comedy</th>
      <th>...</th>
      <th>Comedy,Fantasy,Musical</th>
      <th>Documentary,Drama,History</th>
      <th>Horror,Sci-Fi</th>
      <th>Documentary,History,War</th>
      <th>Comedy,Horror,Mystery</th>
      <th>Adventure,Horror</th>
      <th>Adventure,Biography,Documentary</th>
      <th>Comedy,Fantasy,Thriller</th>
      <th>Action,Biography,Documentary</th>
      <th>Biography,Documentary,Music</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>635063875.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>1</td>
      <td>NaN</td>
      <td>-2.002376e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2</td>
      <td>NaN</td>
      <td>1.072414e+09</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 274 columns</p>
</div>




```python
for col in group_date_df:
    #If the count in the column is less than 25, drop the column
    if group_date_df[col].describe()['count'] < 25:
        group_date_df = group_date_df.drop(columns=col)
    else:
        continue

```


```python
plt.figure(figsize=(20,20))
sns.set(style='darkgrid')
ax = sns.barplot(data=group_date_df, orient='h', ci=68)
ax.set_xlabel('Grouped Average Net Profit', fontsize=40)
ax.set_ylabel('Grouped Genres', fontsize=40)
ax.set_title('Average Net Profit of Grouped Genres From 2010 to 2020',fontsize=50, weight='bold')
ax.set_xticklabels(ax.get_xticklabels(), rotation=15, fontsize=30)
ax.set_yticklabels(ax.get_yticklabels(), fontsize=30)

#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
fmt = '${x:,.0f}'
tick = mtick.StrMethodFormatter(fmt)
ax.xaxis.set_major_formatter(tick);
```


![png](output_356_0.png)


# Test Area


```python
#getting selected grouped genres from date_df where domestic and worldwide gross combined are not 0
next_gen_df = pysqldf("""SELECT *
                         FROM date_df
                         WHERE genres = 'Action,Adventure,Comedy' 
                            OR genres = 'Action,Adventure,Sci-Fi' 
                            OR genres = 'Action,Adventure,Fantasy' 
                            OR genres = 'Adventure,Animation,Comedy'
                            AND (domestic_gross + worldwide_gross) != 0;""")
```


```python
next_gen_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>genres</th>
      <th>runtime_minutes</th>
      <th>release_date</th>
      <th>primary_title</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>tt1298650</td>
      <td>Action,Adventure,Fantasy</td>
      <td>136.0</td>
      <td>2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000.0</td>
      <td>241063875.0</td>
      <td>1.045664e+09</td>
      <td>6.350639e+08</td>
      <td>254.667286</td>
    </tr>
    <tr>
      <td>1</td>
      <td>tt6565702</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>113.0</td>
      <td>2019</td>
      <td>Dark Phoenix</td>
      <td>350000000.0</td>
      <td>42762350.0</td>
      <td>1.497624e+08</td>
      <td>-2.002376e+08</td>
      <td>42.789243</td>
    </tr>
    <tr>
      <td>2</td>
      <td>tt2395427</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>141.0</td>
      <td>2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000.0</td>
      <td>459005868.0</td>
      <td>1.403014e+09</td>
      <td>1.072414e+09</td>
      <td>424.384139</td>
    </tr>
    <tr>
      <td>3</td>
      <td>tt4154756</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>149.0</td>
      <td>2018</td>
      <td>Avengers: Infinity War</td>
      <td>300000000.0</td>
      <td>678815482.0</td>
      <td>2.048134e+09</td>
      <td>1.748134e+09</td>
      <td>682.711400</td>
    </tr>
    <tr>
      <td>4</td>
      <td>tt0974015</td>
      <td>Action,Adventure,Fantasy</td>
      <td>120.0</td>
      <td>2017</td>
      <td>Justice League</td>
      <td>300000000.0</td>
      <td>229024295.0</td>
      <td>6.559452e+08</td>
      <td>3.559452e+08</td>
      <td>218.648403</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(20, 20))
sns.set(style='darkgrid', font_scale=1)
ax = sns.lmplot(x='production_budget', y='tot_profit', col_wrap=2,col='genres',data=next_gen_df, ci=68, robust=True)
fig = plt.gcf()
for ax in fig.get_axes():
    ax.set_xlabel('Production Budget')
    ax.set_ylabel('Average Net Profit')
    ax.set_xticklabels(ax.get_xticklabels(), rotation=25, ha='right')
#Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
    fmt = '${x:,.0f}'
    tick = mtick.StrMethodFormatter(fmt)
    ax.yaxis.set_major_formatter(tick)
    ax.xaxis.set_major_formatter(tick);
```


    <Figure size 1440x1440 with 0 Axes>



![png](output_360_1.png)


#### The above graph shows us that some of the selected grouped genres have a slightly stronger positive correlation when comparing production budget to average net profit


```python
sns.set(font_scale=2)
g = sns.lmplot('production_budget','tot_profit',next_gen_df,hue='genres',aspect=2,col='genres',col_wrap=2, ci=99)
fig = g.fig
axes = fig.get_axes()
for ax in axes:
    ax.set_xlabel('Production Budget')
    ax.set_ylabel('Average Net Profit')
    ax.set_xticklabels(ax.get_xticklabels(), rotation=25, ha='right')
    
    #Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
    fmt = '${x:,.0f}'
    tick = mtick.StrMethodFormatter(fmt)
    ax.yaxis.set_major_formatter(tick)
    ax.xaxis.set_major_formatter(tick);
```


![png](output_362_0.png)


#### Same as the above graph, just with slightly different coding.


```python
next_gen_df.corr()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>runtime_minutes</th>
      <th>release_date</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>tot_profit</th>
      <th>ROI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>runtime_minutes</td>
      <td>1.000000</td>
      <td>0.086043</td>
      <td>0.628423</td>
      <td>0.478251</td>
      <td>0.516214</td>
      <td>0.443951</td>
      <td>0.114609</td>
    </tr>
    <tr>
      <td>release_date</td>
      <td>0.086043</td>
      <td>1.000000</td>
      <td>-0.046861</td>
      <td>-0.014802</td>
      <td>-0.009782</td>
      <td>-0.001162</td>
      <td>0.031703</td>
    </tr>
    <tr>
      <td>production_budget</td>
      <td>0.628423</td>
      <td>-0.046861</td>
      <td>1.000000</td>
      <td>0.533766</td>
      <td>0.608343</td>
      <td>0.469368</td>
      <td>0.013333</td>
    </tr>
    <tr>
      <td>domestic_gross</td>
      <td>0.478251</td>
      <td>-0.014802</td>
      <td>0.533766</td>
      <td>1.000000</td>
      <td>0.938902</td>
      <td>0.933849</td>
      <td>0.657150</td>
    </tr>
    <tr>
      <td>worldwide_gross</td>
      <td>0.516214</td>
      <td>-0.009782</td>
      <td>0.608343</td>
      <td>0.938902</td>
      <td>1.000000</td>
      <td>0.986353</td>
      <td>0.684802</td>
    </tr>
    <tr>
      <td>tot_profit</td>
      <td>0.443951</td>
      <td>-0.001162</td>
      <td>0.469368</td>
      <td>0.933849</td>
      <td>0.986353</td>
      <td>1.000000</td>
      <td>0.759111</td>
    </tr>
    <tr>
      <td>ROI</td>
      <td>0.114609</td>
      <td>0.031703</td>
      <td>0.013333</td>
      <td>0.657150</td>
      <td>0.684802</td>
      <td>0.759111</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
sns.set(font_scale=2)
h = sns.lmplot('tot_profit', 'runtime_minutes',next_gen_df,hue='genres',aspect=2,col='genres',col_wrap=2)
fig = h.fig
axes = fig.get_axes()
for ax in axes:
    ax.set_xlabel('Total Profit')
    ax.set_ylabel('Runtime Minutes')
    ax.set_xticklabels(ax.get_xticklabels(), rotation=25, ha='right')
    #Retrieved from https://stackoverflow.com/questions/38152356/matplotlib-dollar-sign-with-thousands-comma-tick-labels
    fmt1 = '${x:,.0f}'
    fmt2 = '{x:,.0f} min'
    tick1 = mtick.StrMethodFormatter(fmt1)
    tick2 = mtick.StrMethodFormatter(fmt2)
    ax.yaxis.set_major_formatter(tick2)
    ax.xaxis.set_major_formatter(tick1);
```


![png](output_365_0.png)


#### Comparing each of the selected grouped genres total profit to their runtime minutes. Again, some have a slightly higher positive correlation than other selected grouped genres.
