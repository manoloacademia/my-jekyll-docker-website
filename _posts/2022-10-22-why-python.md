---
layout: post
title: "Why python being Process Engineer?"
categories: personal
---

## A Pythonist is born

Long time ago, a Process Engineer in the middle of nowhere, struggles with the short time he has to analyze several Excel files. He notices that he is doing the same 12 actions file after file wasting his time.

"How could I shorten these repetitions?" He wondered.

Luckily to him, there is a magical solution beyond his power (at that time...) that might help him to diminish his time.

![From Excel to Python](https://user-images.githubusercontent.com/76537245/197362887-3abd02b4-60d9-4a04-ba96-2016b14fcb67.png)

## From my point of view (at last)

Enough with the __dungeons and dragons__ thing.

A couple of years ago, I realized that my time to analysis was about one hour after downloading all the Excel files from work's database. This is a long time to keep doing repetitive filtering actions.

Thus, I started to look that a programming languages could allow me to set some rules to avoid these repetitions.
I found that Python could be the one.

## Where to study it?

The first option was to engage myself to a course in ITBA University. Despite the fact that it was oriented to Finances, it allowed me to learn the basic data types and discover [Pandas](https://pandas.pydata.org) library.

![Pandas](https://user-images.githubusercontent.com/76537245/197362894-a16ea4f1-d5ba-4536-b8af-a9d53b5b0db8.png)

## How do I solve the long time?

With Pandas I could apply all the filtering work inside the Excel report:

```python
import pandas as pd # this is a standard way

# Define a funtion to filter all the necessary data
def filter_file(df: pd.DataFrame) -> pd.DataFrame:
    """ This function helps to solve the filtering long time """
    df = pd.read_excel("work_file.xls")
    df[column_1] = df[column_1].apply(lambda x: x.lower())
    .
    .
    return df_filtered
```

And using [REGular EXpressions](https://docs.python.org/3/library/re.html) operations and [Bash Scripting](https://www.gnu.org/software/bash/manual/bash.html), I could finally run my analysis faster than ever (*__minutes not hours!!__*):

```python
#!/usr/bin/env python3 
# That was the shebang for python

from helper import * # helper file, where the filters lay
import re # regex playing
import os # OS playing

regex = r"production report \([0-9]\)*.xlsx"
dirs = os.listdir(os.getcwd())
files = re.findall(regex, str(dirs))

lots = get_lots(files) # function from helper
lots.to_excel("results.xlsx")
```

Not the best script, but come on! my first one :D

## What next?

In next chapters, I'm going share the rest of the history (2 years and counting).

I'm glad to meet you!


## *__Cheers, Pablo.__*
