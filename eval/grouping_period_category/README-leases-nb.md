---
jupytext:
  custom_cell_magics: kql
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.17.3
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

(label-tp-leases)=
# grouping by period and category

```{admonition} download the zip
:class: warning

to work on this assignment locally on your laptop, {download}`start with downloading the zip<./ARTEFACTS-leases.zip>`
```

in this TP we work on

- data that represents *periods* and not just one timestamp
- checking for overlaps
- grouping by period (week, month, year..)
- then later on, grouping by period *and* category
- and some simple visualization tools

here's an example of the outputs we will obtain

(label-leases-output)=
````{grid} 3 3 3 3
```{image} media/result-color-w.png
```
```{image} media/result-color-m.png
```
```{image} media/result-color-y.png
```
````

+++

## imports

```{code-cell} ipython3
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
```

1. make sure to use matplotlib in interactive mode - aka `ipympl`

```{code-cell} ipython3
:tags: [level_basic]

# your code
%matplotlib ipympl
```

2. optional: setup itables, so that we can have scrollable tables

```{code-cell} ipython3
:tags: [level_basic]

# your code
import itables
itables.init_notebook_mode()
```

## the data

we have a table of events, each with a begin (`beg`) and `end` time; in addition each is attached to a `country`

```{code-cell} ipython3
leases = pd.read_csv("data/leases.csv")
leases.head(10)
```

### adapt the type of each columns

surely the columns dtypes need some care

```{code-cell} ipython3
:tags: [level_basic]

# your code
leases['beg'] = pd.to_datetime(leases['beg'])
leases['end'] = pd.to_datetime(leases['end'])
```

```{code-cell} ipython3
:tags: [level_intermediate]

# check it

leases.dtypes
```

### raincheck

check that the data is well-formed, i.e. **the `end`** timestamp **happens after `beg`**

```{code-cell} ipython3
:tags: [level_basic]

# your code

# ceci nous renvoie un tableau avec que des true
(leases['end'] - leases['beg']) >= pd.to_timedelta(0)
# mais pour voir directement, on peut aussi faire 
np.sum((leases['end'] - leases['beg']) < pd.to_timedelta(0))
```

### are there any overlapping events ?

+++

it turns out there are **no event overlap**, but write a code that checks that this is true

```{admonition} note
:class: tip

nothing in the rest depends on this question, so if you find this too hard, you can skip to the next question
```

```{code-cell} ipython3
:tags: [level_basic]

leases.sort_values(by = 'beg',inplace = True, ignore_index= True)

# après avoir trié le dataframe on va en créer une copie mais décalée de 1 pour pouvoir comparer la fin d'une période avec le début de la période suivante
leases1 = leases.drop(leases.index[-1], axis = 0)
leases2 = leases.drop(leases.index[0], axis = 0)

# le .values permet de comparer les valeurs malgré le fait que les index sont différents
leases1['end'] <= leases2['beg'].values

# On obtient une Series avec que des true donc il n'y pas d'overlap
```

### timespan

What is the timespan covered by the dataset (**earliest** and **latest** events, and **duration** in-between) ?

```{code-cell} ipython3
:tags: [level_basic]

# your code
a = leases.loc[leases.index[0],'beg']
b = leases.loc[leases.index[-1],'end']
print("début :",a,"fin :",b,"timespan :",b-a)
```

### aggregated duration

so, given that there is no overlap, we can assume this corresponds to "reservations" attached to a unique resource (hence the term  *lease*)  
write a code that computes the **overall reservation time**, as well as the **average usage ratio** over the overall timespan

```{code-cell} ipython3
:tags: [level_basic]

# your code
tot = np.sum(leases['end'] - leases['beg'])
print("temps total de réservation :",tot)
print("durée moyenne d'une résevation:", tot/len(leases))
print("fraction du temps ayant servi à une réservation :", tot/(b-a))
```

## visualization - grouping by period

### usage by period

grouping by periods: by week, by month or by year, display the **total usage in that period**  
(when ambiguous, use the `beg` column to determine if a lease is in a period or the other)

```{admonition} *hint*
:class: dropdown tip

There are at least 2 options to do this grouping, based on `resample()` and `to_period()`  
advanced users may wish to write them both and to comment on their respective pros and cons

```

`````{admonition} for now, **just get the grouping right**
:class: dropdown

you should produce something like e.g.

````{grid} 3 3 3 3
```{image} media/result-bw-w.png
```
```{image} media/result-bw-m.png
```
```{image} media/result-bw-y.png
```
````
we'll make cosmetic improvements below, and [the final results look like this](#label-leases-output), but let's not get ahead of ourselves
`````

```{code-cell} ipython3
:tags: [level_basic]

# your code

leases = pd.read_csv("data/leases.csv")
leases['beg'] = pd.to_datetime(leases['beg'])
leases['end'] = pd.to_datetime(leases['end'])

leases['time_delta'] = leases['end']-leases['beg']
leases.set_index('beg', inplace = True)

leases_year = leases.to_period('Y')
leases_month = leases.to_period('M')
leases_week = leases.to_period('W')

by_year = leases_year.groupby(by = 'beg')
by_month = leases_month.groupby(by = 'beg')
by_week = leases_week.groupby(by = 'beg')



plt.figure(1)
plt.subplot(1,3,1)
by_year['time_delta'].sum().plot.bar()

plt.subplot(1,3,2)
by_month['time_delta'].sum().plot.bar()

plt.subplot(1,3,3)
by_week['time_delta'].sum().plot.bar()

plt.show()
```

### improve the title and bottom ticks

add a title to your visualisations

also, and particularly relevant in the case of the per-week visu, we don't get to read **the labels on the horizontal axis**, because there are **too many of them**  
to improve this, you can use matplotlib's `set_xticks()` function; you can either figure out by yourself, or read the few tips below

````{admonition} a few tips
:class: dropdown tip

- the object that receives the `set_xticks()` method is an instance of `Axes` (one X&Y axes system),  
  which is not the figure itself (a figure may contain several Axes)  
  ask google or chatgpt to find the way you can spot the `Axes` instance in your figure
- it is not that clear in the docs, but all you need to do is to pass `set_xticks` a list of *indices* (integers)  
  i.e. if you have, say, a hundred bars, you could pass `[0, 10, 20, ..., 100]` and you will end up with one tick every 10 bars.
- there are also means to use smaller fonts, which may help see more relevant info
````

```{code-cell} ipython3
# let's say as arule of thumb
LEGEND = {
    'W': "week",
    'M': "month",
    'Y': "year",
}

SPACES = {
    'W': 12,   # in the per-week visu, show one tick every 12 - so about one every 3 months
    'M': 3,    # one every 3 months
    'Y': 1,    # on all years
}
```

```{code-cell} ipython3
:tags: [level_basic]

# your code
fig, ax = plt.subplots(nrows = 1, ncols = 3)

plt.subplot(1,3,1)
by_year['time_delta'].sum().plot.bar()
plt.title("Par an")

plt.subplot(1,3,2)
plt.title("Par mois")
by_month['time_delta'].sum().plot.bar()
ax[1].set_xticks([3*i for i in range(int(953/30/3)+1)])

plt.subplot(1,3,3)
by_week['time_delta'].sum().plot.bar()
plt.title("Par semaine")
ax[2].set_xticks([12*i for i in range(int(953/7/12)+1)])

plt.show()
```

### a function to convert to hours

you are to write a function that converts a `pd.Timedelta` into a number of hours  
1. read and understand the test code for the details of what is expected
2. use it to test your own implementation

```{code-cell} ipython3
:tags: [level_basic]

# your code
from math import ceil

def convert_timedelta_to_hours(timedelta: pd.Timedelta) -> int:
    sec = timedelta.seconds
    return ceil(sec/3600) + 24 * timedelta.days
```

```{code-cell} ipython3
:tags: [level_intermediate]

# test it

# if an hour has started even by one second, it is counted
test_cases = ( 
    # input in seconds, expected result in hours
    (0, 0), 
    (1, 1),     (3599, 1),     (3600, 1), 
    (3601, 2),  (7199, 2),     (7200, 2), 
    # 2 hours + 1s -> 3 hours
    (7201, 3),  
    # 3 hours + 2 minutes -> 4 hours
    (pd.Timedelta(3, 'h') + pd.Timedelta(2, 'm'), 4),
    # 2 days -> 48 hours
    (pd.Timedelta(2, 'D'), 48),
)

def test_convert_timedelta_to_hours():
    for seconds, exp in test_cases:
        # convert into pd.Timedelta if not already one
        if not isinstance(seconds, pd.Timedelta):
            timedelta = pd.Timedelta(seconds=seconds)
        else:
            timedelta = seconds
        # compute and compare
        got = convert_timedelta_to_hours(timedelta)
        print(f"with {timedelta=} we get {got} and expected {exp} -> {got == exp}")

test_convert_timedelta_to_hours()
```

```{code-cell} ipython3
:tags: [level_intermediate]

# for debugging; this should return 48

convert_timedelta_to_hours(pd.Timedelta(2, 'D'))
```

### use it to display totals in hours

keep the same visu, but display **the Y axis in hours**  
btw, what was the unit in the graphs above ?

```{code-cell} ipython3
:tags: [level_basic]

# the graph above was in nanoseconds



# On repart du début pour éviter les malentendus
leases = pd.read_csv("data/leases.csv")
leases['beg'] = pd.to_datetime(leases['beg'])
leases['end'] = pd.to_datetime(leases['end'])

leases['time_delta'] = leases['end']-leases['beg']

# ici ça change : on crée une nouvelle colonne avec les temps en heures 
leases['time_delta_hours'] = leases['time_delta'].apply(convert_timedelta_to_hours)
leases.set_index('beg', inplace = True)

leases_year = leases.to_period('Y')
leases_month = leases.to_period('M')
leases_week = leases.to_period('W')

by_year = leases_year.groupby(by = 'beg')
by_month = leases_month.groupby(by = 'beg')
by_week = leases_week.groupby(by = 'beg')


# puis on affiche
fig, ax = plt.subplots(nrows = 1, ncols = 3)

plt.subplot(1,3,1)
by_year['time_delta_hours'].sum().plot.bar()
plt.title("Par an")

plt.subplot(1,3,2)
plt.title("Par mois")
by_month['time_delta_hours'].sum().plot.bar()
ax[1].set_xticks([3*i for i in range(int(953/30/3)+1)])

plt.subplot(1,3,3)
by_week['time_delta_hours'].sum().plot.bar()
plt.title("Par semaine")
ax[2].set_xticks([12*i for i in range(int(953/7/12)+1)])

plt.show()
```

## grouping by period and region

the following table allows you to map each country into a region

```{code-cell} ipython3
# load it
leases = pd.read_csv("data/leases.csv")
leases['beg'] = pd.to_datetime(leases['beg'])
leases['end'] = pd.to_datetime(leases['end'])

leases['time_delta'] = (leases['end']-leases['beg']).apply(convert_timedelta_to_hours)

countries = pd.read_csv("data/countries.csv")
countries.head(3)
```

### a glimpse on regions

what's the most effective way to see how many regions and how many countries per region ?

```{code-cell} ipython3
:tags: [level_basic]

# your code
by_region = countries.groupby(by = 'region')
by_region.count()
```

### attach a region to each lease

your mission is to now show the same graphs, but we want to reflect the relative usage of each region, so we want to [split each bar into several colors, one per region see expected result below](#label-leases-output)

+++

most likely your first move is to tag all leases with a `region` column

```{code-cell} ipython3
:tags: [level_basic]

# your code
leases_regions = pd.merge(leases, countries, left_on = 'country', right_on = 'name')
leases_regions.drop(['name'],axis = 1, inplace = True)
leases_regions.head()
```

### visu by period by region

you can now produce [the target figures, again they look like this](#label-leases-output)

```{code-cell} ipython3
:tags: [level_basic]

# your code

leases_regions.set_index('beg', inplace = True)




fig, ax = plt.subplots(nrows = 1, ncols = 3)


leases_regions_year = leases_regions.to_period('Y')
by_year = leases_regions_year.groupby(by = ['beg','region'])
res_y = by_year['time_delta'].sum().unstack()
res_y.plot(kind = 'bar', stacked = True, ax = ax[0], title = "Par an")


leases_regions_month = leases_regions.to_period('M')
by_month = leases_regions_month.groupby(by = ['beg','region'])
res_m = by_month['time_delta'].sum().unstack()
res_m.plot(kind = 'bar', stacked = True, ax = ax[1], title = "Par mois")
ax[1].set_xticks([3*i for i in range(int(953/30/3)+1)])


leases_regions_week = leases_regions.to_period('W')
by_week = leases_regions_week.groupby(by = ['beg','region'])
res_w = by_week['time_delta'].sum().unstack()
res_w.plot(kind = 'bar', stacked = True, ax = ax[2], title = "Par semaine")
ax[2].set_xticks([12*i for i in range(int(953/7/12)+1)])


plt.show()
```

***
