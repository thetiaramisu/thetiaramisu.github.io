---
layout: post
title:      "Exploring the King County Housing Market"
date:       2020-04-27 09:06:41 +0000
permalink:  exploring_the_king_county_housing_market
---


After a deep dive into learning Python, statistics, and data science processes these past few weeks, I've gotten to apply what I've learned to my first data science project. As you're learning fundamentals, it's easy to look at each lesson independently and study in a way that is more textbook. But it was amazing getting to see how much I can do when putting all these building blocks together.

For this project, I analyzed a dataset of housing sales for King County in Washington. This data includes publicly released data for a certain time period in King County, regarding house features, location specifications, and the selling price and date for each house. The objective of the project was to create a regression model that predicts the selling price for houses in King County, based on the provided historic data.

The process I used for this project utilized the OSEMN framework. The process consists of the following steps:

* **Obtain:** Understanding the details of the assignment, the objective in addressing the questions at hand, and obtaining the relevant data.
* **Scrub:** Filtering through the collected data to get it to a useful and workable condition. This may include removing irrelevant or cumbersome data, and addressing the issues of messy values in the data set.
* **Explore:** Taking steps to gain an understanding of the data set, such as creating visualizations to further explore the distributions and relationships amongst variables.
* **Model:** Building and tuning models using machine learning algorithms to develop a final model that can make future predictions with confidence.
* **iNterpret:** Drawing conclusions from the data, evaluating the meaning and implications of the findings, and communicating the results.

With the challenge to explore the dataset, I focused on three questions. I used visualizations to help find meaningful interpretations and conclusions from the given data for each question.

**Does the time of year impact the selling value?**

In exploring the selling patterns throughout different months of the year, I analyzed the trends in the number of sales per month, as well as the average price that the homes sell for each month.

I took the provided date information and broke down the data into months, then used Seaborn's barplot to visualize it. This is what my code and results looked like.

```
months = ['January','February','March','April','May','June','July','August','September','October','November','December']
monthly_sales = []

for i in range(1,13):
    monthly_sales.append(sum(kc.date.dt.month == i))
    
plt.figure(figsize=(10,4))
sns.barplot(months, monthly_sales)
plt.title('Number of sales by month', fontdict={'fontsize': 16})
plt.show()
```

![](https://thetiaramisu.files.wordpress.com/2020/03/kcimg1.png)

Here we can see that the number of houses in the dataset that were sold each month had a pretty clear trend. Late spring is the most popular period for housing sales, contrasting with the winter months. The most popular selling month is May, while the least popular is January.

To get a sense of what the selling prices looked like over the span of the months, I created a small dataset just to see the average selling price, using just the date and price columns from the original. I then found the average selling price for each month by grouping the data by month.

```
just_time = kc[['date','price']]
averages = just_time.groupby(kc.date.dt.month).mean().sort_values('price',ascending=True)
averages.sort_index(axis=0,ascending=True,inplace=True)
```

With that smaller dataset, I found the following trends.

```
months = ['January','February','March','April','May','June','July','August','September','October','November','December']
monthly_avgs = []

for i in range(1,13):
    monthly_avgs.append(averages.price[i])
    
plt.figure(figsize=(10,4))
sns.barplot(months, monthly_avgs)
plt.title('Average sell price by month', fontdict={'fontsize': 16})
plt.show()
```

![](https://thetiaramisu.files.wordpress.com/2020/03/kcimg2.png)

Here we can see that there is a similar trend in the average sell price by month, with prices trending higher in the spring and lower in the winter. To get a closer view of the monthly variation, I made Seaborn pointplots to look at the same data above.

```
plt.figure(figsize=(12,4))
sns.pointplot(months, monthly_sales)
plt.title('Number of sales by month', fontdict={'fontsize': 16})
plt.show()
    
plt.figure(figsize=(12,4))
sns.pointplot(months, monthly_avgs)
plt.title('Average sell price by month', fontdict={'fontsize': 16})
plt.show()
```

![](https://thetiaramisu.files.wordpress.com/2020/03/kcimg3.png)

![](https://thetiaramisu.files.wordpress.com/2020/03/kcimg4.png)

Using both visualizations, I can gather that late spring is an ideal time to sell a house. It is a more common time for houses to sell, and the average price per house sale is also higher in this time period.

**Is condition or grade of house a bigger factor in price?**

My second question aimed at analyzing the difference in the impact that condition and grade have on price. Both of these features are assessed on a scale. From the [King County Residential Glossary of Terms](https://info.kingcounty.gov/assessor/esales/Glossary.aspx?type=r), I found the breakdown of what each means. Condition is coded on a 1-5 scale,  representing the condition of the house relative to its age and grade. Grade is coded on a 3-13 scale, representing the construction quality of the house. Both scales go from low to high quality as the numbers increase.

To answer this question, I created two small data sets for just, grouping condition by average price and grade by average price.

```
conditions = kc[['condition','price']]
cond_avg = conditions.groupby(kc.condition).mean()
cond_avg.sort_index(axis=0,ascending=True,inplace=True)
```

```
grades = kc[['grade','price']]
grades.grade = pd.to_numeric(grades.grade)
grade_avg = grades.groupby(kc.grade).mean().sort_values('grade',ascending=True)
```

I then made a pointplot for each set to compare. This is what the plots looked like.

```
plt.figure(figsize=(12,4))
sns.pointplot(conditions.condition, conditions.price)
plt.title('avg price by condition', fontdict={'fontsize': 16})
plt.show()
    
plt.figure(figsize=(12,4))
sns.pointplot(grades.grade, grades.price)
plt.title('avg price by grade', fontdict={'fontsize': 16})
plt.show();
```

![](https://thetiaramisu.files.wordpress.com/2020/03/kcimg5.png)

![](https://thetiaramisu.files.wordpress.com/2020/03/kcimg6.png)

Based on the graphs, I concluded that grade has a more consistent impact on price than condition does. There is a sizable jump in the average price from houses in conditions 1-2 to those in 3-4, and a minor increase from houses in conditions 3-4 to those in 5. With grade, there is a clear increase as you climb up the grade scale, which gets more significant the higher the grade gets. It is interesting and leads me to assume that at a certain point, the condition of the does not make a major impact on the price anymore, meaning people are more willing to accept a price that is average or good, and that very good houses are not always worth spending way more money on.

**Do renovated houses sell for more than houses that were not renovated?**

Knowing that the grade has a very steady impact on the price, I wanted to look further into the trend to see if renovations made a further difference. To do this, I again made two smaller datasets, this time with one dataset grouping the average price per grade for houses that were renovated and the other dataset doing the same for houses that were not renovated. The original dataset has a column indicating the year renovated for each house, with the value 0 entered for houses that have not been renovated.

```
kc_renod = kc[kc.yr_renovated>0][['grade','price']]
kc_not_renod = kc[kc.yr_renovated==0][['grade','price']]
```

Upon grouping them, I found that from the dataset, 739 houses have been renovated, while 20,330 are still in their original condition. Amongst those renovated and not renovated houses, these visualizations shows the average price for each grade respectively.


```
plt.figure(figsize=(12,4))
kc_renod.grade = pd.to_numeric(kc_renod.grade)
sns.pointplot(kc_renod.grade, kc_renod.price)
plt.title('price per grade for renovated', fontdict={'fontsize': 16})
plt.show()
    
plt.figure(figsize=(12,4))
kc_not_renod.grade = pd.to_numeric(kc_not_renod.grade)
sns.pointplot(kc_not_renod.grade, kc_not_renod.price)
plt.title('price per grade for not renovated', fontdict={'fontsize': 16})
plt.show()
```

![](https://thetiaramisu.files.wordpress.com/2020/03/kcimg7.png)

![](https://thetiaramisu.files.wordpress.com/2020/03/kcimg8.png)

In comparing houses that were renovated and houses that were not renovated, we can see that house renovations seem to make a significant impact on the house value. The difference seems higher for the comparison on the higher grade houses than the lower ones, with the exception of grade 12.

The average renovated house sold for $215,000 more than the average house that was not renovated. Also, as we saw from the previous question that grade also plays a significant factor in the price, we compared the price per grade for houses that were renovated and houses that were not. Percentage-wise, houses that were renovated are generally of higher grade than houses that are not. Breaking that down further, the average price per house was also higher for each grade level for renovated houses over houses that were not renovated.

Thank you for reading and exploring the King County data set with me! Check out my full project [here](https://github.com/thetiaramisu/dsc-1-final-project-online-ds-sp-000)!
