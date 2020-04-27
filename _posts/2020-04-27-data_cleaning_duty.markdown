---
layout: post
title:      "(Data) Cleaning Duty"
date:       2020-04-27 10:06:31 +0000
permalink:  data_cleaning_duty
---


![](https://thetiaramisu.files.wordpress.com/2020/04/snow-white-gif.gif?w=488&zoom=2)

Cleaning is one of those things we love to hate and hate to love. It is the embodiment of the word “chore,” which Cambridge dictionary spot on defines as “a job or piece of work that is often boring or unpleasant but needs to be done regularly.” But as frustratingly repetitive as it is, we can’t get by without it.

Using messy or uncleaned data does more harm than good, as it leads to inaccurate conclusions. That’s why one of the first things to do after acquiring new data is check out what cleaning tasks are necessary, and if the dataset is even usable to begin with.

As I began finding my own datasets to do projects on, I started the process of cleaning several different datasets, each of which I found unusable, before I found one that I could work with. Some issues that I discovered upon cleaning were:

* Text data that were cut off at a certain character limit,
* Excessively inconsistent or subjective user input data, and
* Datasets in which the majority of the columns turned out to be derivations of the information from the same two or three columns.

![](https://thetiaramisu.files.wordpress.com/2020/04/data-scrub.jpg)

In this post, I will share some of the tools I typically use to inspect data and the processes I take to clean them. I will also share some examples of what it looks like when I use them.

First off, this is a function I typically use for every notebook. I run it right after I import my main packages. Rather than just showing me the first few rows, it gives me a preview including both the head and tail of the dataframe, which proves to be very useful sometimes!

```
def view(dataframe, n: int=3):
    """Displays the preview of the first and last rows of the dataframe
    Args:
        dataframe: The dataframe being called
        n: Number of rows to select for preview. Defaults to 3.
    Returns:
        The first n and the last n rows of the dataframe
    """

    with pd.option_context('display.max_rows',n*2):
        display(dataframe)
```
				
![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-27-at-2.04.13-am.jpg?w=1576)

### Irrelevant Data

![](https://thetiaramisu.files.wordpress.com/2020/04/irrelevant-meme.jpg)

When you gather data, it’s frequently the case that not everything is pertinent to the purposes of the problem you’re trying to solve. As you familiarizing yourself with the dataset, you may find columns that don’t add any value to your project. You may also be intending to only work with a subset of the whole dataset. For example, if you are building a model tracking health conditions for a certain demographic, you can remove all information for individuals that do not fit that demographic.

Another instance of irrelevant data is duplicate observations. Oftentimes you find that the same observation appears more than once within the dataset. This can easily happen when data is combined from multiple places, through an input oversight, or when a user hits the submit button twice.

These are some of my go-to codes in dealing with irrelevant data.

```
# View with condition

view(df[df.str.contains("words_in_condition")],3)
```

![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-27-at-1.57.23-am.jpg)

```
# View duplicate values

print(f'Number of duplicates: {(df[column_name].value_counts() > 1).sum()}')
```

![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-27-at-1.54.46-am-1.jpg?w=1576)

```
# Drop columns

df = df.drop(column=["unnecessary_column"]
```

![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-27-at-2.02.07-am.jpg?w=1576)

### Incorrect Data Types

![](https://thetiaramisu.files.wordpress.com/2020/04/download.jpeg)

It is common for data types to be misidentified. Categorical data can very easily be mistaken for numeric data, especially when it is coded with numbers. (This doesn’t necessarily need to be changed, but it does need to be acknowledged.) Numerical data with input issues can get mistaken as strings, such as when “NA” is put in for a value. Datatypes are set by the column, so all the entries within a column need to read as the same datatype. Because of this, errors will arise when trying to use data when there is an inconsistency in the understood data type and what it actually is.

Datetime information very frequently needs to be corrected as well. Especially when input by users, people use various formats to indicate time that pandas takes in as a string of characters because it doesn’t fit its datetime format. For example, to describe one individual date, you could type it in a variety of ways, including:

* July 4, 2019
* 07/04/19
* 7/5/2019 12:00:00 AM
* 7-4-19
* Independence Day, 2019!🎇🎉🎆

These are some of my go-to codes for fixing incorrect data types.

```
# Fix data types

df['column_name'] = df['column_name'].astype(str) #change to string

df['column_name'] = df['column_name'].astype(int) #change to int

df['column_name'] = df['column_name'].astype(float) #change to float
```

![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-27-at-2.17.36-am.jpg?w=1576)

```
# Change column values to boolean

df['column_name'] = df['column_name'].replace('positive', True).replace('negative', False)
```

![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-27-at-2.13.34-am.jpg?w=1576)

```
# Change to datetime format

df['column_name'] = pd.to_datetime(df.['column_name'])
```

![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-27-at-1.37.23-am.jpeg)

### Structural Errors

![](https://thetiaramisu.files.wordpress.com/2020/04/na-na-na-na-batman.jpg)

Structural errors vary greatly. Syntax errors can include issues such as white spaces, typos, capitalization differences. Often times you’ll find filler values, which can end up greatly skewing data analysis if not caught. Depending on the data, fillers values could look like “NA,” an arbitrary date put in such as 1/1/1900, or an unrealistically extreme value such as 0 or 9999.

These are some of my go-to codes for addressing structural errors.

```
# Replace values when condition is false

df['column_name'] = df['column_name'].where(df['column_name'].str.contains("values_to_ignore"), "wanted_value")
```

This is a function I use to rename values in a column.

```
def rename(dataframe, column, original, rename):
    """Replaces all instances of a specific string in one column with a new string
    Args:
        dataframe: The dataframe being called
        column: The column being called
        oringal: Original string to be replaced
        rename: New string to replace original string
    """

    dataframe.loc[dataframe[column].str.contains(original), column] = str(rename)
    return
```

I use this when I see a trend in the alternate spellings of the same string. Here’s an example.

![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-27-at-1.45.44-am.jpg)

### Missing Data

![](https://thetiaramisu.files.wordpress.com/2020/04/missing-data-grandma.jpg)

Values are sometimes left blank if the particular aspect a column asks for is irrelevant to the entry. For example, if you have a dataset containing data on property values and there is a column for the year a property was renovated, there could be nothing entered for that entry if the house wasn’t renovated. This would get entered as NaN (Not a Number).

Null values can vary widely in their meaning and you really can’t stick to one strict method of handling. In some cases, it could be reasonable to replace the null values with the modal value from the non-null values in the column. In other cases, doing so would drastically skew the data. In some cases, it makes sense to just drop a column that has more null values than not. But in other cases, doing so could make you unknowingly forfeit valuable insights you could have gained through exploring the data. Sometimes you can glean just as much information from the absence of values as you can from the presence of them.

There is no generic answer for what you should do in every case, so you will need to use your own judgment and common sense to discern the best way to handle the data.

These are some of my go-to codes when handling missing data.

```
# Check Nulls

nulls = pd.DataFrame({"total nulls":df.isna().sum(),\
        "percentage %":round(df.isna().sum()/len(df)*100,2)})
nulls
```

Screen Shot 2020-04-27 at 1.13.12 AM

```
# Drop nulls

df = df[~df.isna()]
```

Screen Shot 2020-04-27 at 1.35.27 AM

```
# Fill null values

df['column_name'].fillna("new_value", inplace=True)
```

![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-27-at-1.33.32-am.jpg)

### Outliers

![](https://thetiaramisu.files.wordpress.com/2020/04/lion-king-outlier-baboon.jpg?w=450&h=600)

An outlier is an observation that lies an abnormal distance from other values in the overall pattern of a distribution. They are easy to identify through graphs such as histograms or box plots.

One conventional definition for an outlier is a point that falls more than 1.5 times the interquartile range above the third quartile or below the first quartile. In a box and whisker plot, these values are identified as dots outside of the whiskers that demarcate the upper and outer ranges of the main trend in the distribution.

For modeling purposes, outliers can sometimes cause big issues. For example, with linear regression models, outliers can heavily skew the data, yielding biased results. Removing an outlier in such a situation will help the model’s performance. But in other cases, such as decision trees, outliers may not be as big of a threat and could provide some very valuable information to the model. An outlier shouldn’t be considered bad data that needs to be removed, unless you find good reason to remove it, such as reason to suspect that it’s inaccurate data.

These are some of my go-to codes in assessing outliers.

This is a function I use to see the statistical distribution and the box plot of a single column.

```
def stats(dataframe, column):
    """Generates single column stats and boxplot
    Args:
        dataframe: The dataframe being called
        column: The column being called
    Returns:
        Descriptive statistics and boxplot of the column
    """
    
    print(f'Data type: {dataframe[column].dtype}')
    print(f'Count: {dataframe[column].shape[0]}')
    print(f'Number of Unique Values: {dataframe[column].nunique()}')
    print(f'Number of nulls: {dataframe[column].isna().sum()} - {round(100*dataframe[column].isna().sum()/len(dataframe),2)}%')
    print()
    print(f'Mean: {dataframe[column].mean()}')
    print(f'Minimum: {dataframe[column].min()}')
    print(f'Lower Quartile: {dataframe[column].quantile(.25)}')
    print(f'Median: {dataframe[column].quantile(.5)}')
    print(f'Upper Quartile: {dataframe[column].quantile(.75)}')
    print(f'Maximum: {dataframe[column].max()}')

    plt.figure(figsize=(14,1))
    plt.title(f'Boxplot Distrubution of {column}', fontsize=14, fontweight='bold')
    sns.boxplot(x=dataframe[column])
    plt.xlabel(column, fontsize=13, fontweight='bold')
    plt.show()
```
		
![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-27-at-1.08.55-am.jpg?w=1574)

```
# See boxplot with outliers removed

sns.boxplot(df.list_price, showfliers=False)
plt.show()
```

![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-27-at-1.09.25-am.jpg?w=1576)

Those are some of the tools in my tool box!

The order in which you do these will often depend on the data, which you’ll find as you run into warnings about inconsistencies that keep you from fixing certain issues until you address others. Likewise, the decision process for how you will address each issue will differ from dataset to dataset and from feature to feature. As mentioned earlier, there is no generic answer for what you should do in every case, so you will need to use your own judgment and common sense to discern the best way to handle the data.

It’s a tedious but rewarding process. And after you gone through the grind —

![](https://thetiaramisu.files.wordpress.com/2020/04/lego-clean-hair.gif?w=500&zoom=2)

— you’ll be ready to rock and roll!
