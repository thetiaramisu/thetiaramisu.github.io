---
layout: post
title:      "Query Functions for SQLite"
date:       2020-04-27 09:17:13 +0000
permalink:  query_functions_for_sqlite
---


For Flatiron School, we were assigned a project focusing on hypothesis testing. We used the Northwind Traders database, which was created by Microsoft in 2000 as a representation of a fictitious trading company. The database was provided in SQLite format. In this post, I will highlight some of the functions that I created to optimize my workflow as I explored the database. I will then share how I used these functions in my process of hypothesis testing.

**Importing the necessary packages and the sqlite file containing the database**

```
from sqlalchemy import create_engine
from sqlalchemy import inspect

engine = create_engine("sqlite:///Northwind_small.sqlite")
con = engine.connect()
insp = inspect(engine)
```

Once this sqlite file is read into my notebook, I can use my functions with it.

**Reading SQL queries**

```
def query(statement):
    """Simplifies the process of making an SQL query in Pandas \
    by eliminating the need to type the pre- and post- text \
    of the query itself
    Arg:
        statement: the SQL query
    Returns: the dataframe result of the SQL query
    """
    return pd.read_sql_query(statement, con)
```

This function simplifies the process of performing SQLite queries and receiving the results in a pandas format. Having this function allowed for quick searches in a cleaner-looking format.

As opposed to typing:

```
disc_grouped = pd.read_sql_query("SELECT SUM (Quantity) \
FROM OrderDetail GROUP BY OrderId HAVING SUM (Discount) != 0", con)

full_grouped = pd.read_sql_query("SELECT SUM (Quantity) \
FROM OrderDetail GROUP BY OrderId HAVING SUM (Discount) = 0", con)
```

I just typed:

```
disc_grouped = query("SELECT SUM (Quantity) FROM OrderDetail \
                      GROUP BY OrderId HAVING SUM (Discount) != 0")

full_grouped = query("SELECT SUM (Quantity) FROM OrderDetail \
                      GROUP BY OrderId HAVING SUM (Discount) = 0")
```

It may not seem like a big difference, but when you multiply the number of queries you have to make by 50, it's much less annoying to have a function that minimized the work!

Creating a Pandas data frame from a whole SQL table

```
def table(each):
    statement = "SELECT * FROM ["+each+"];"
    return query(statement)
```

The Northwind database came with 13 separate tables. With my query function from earlier, I was quickly and easily able to create a pandas dataframe for each of those tables using the table names. I could then call on each dataframe and combine or manipulate them easily.

```
# Declaring a pandas dataframe variable for each table

categories = table("Category")
customers = table("Customer")
cust_custdemos = table("CustomerCustomerDemo")
demographics = table("CustomerDemographic")
employees = table("Employee")
emp_territories = table("EmployeeTerritory")
orders = table("Order")
order_dets = table("OrderDetail")
products = table("Product")
regions = table("Region")
shippers = table("Shipper")
suppliers = table("Supplier")
territories = table("Territory")
```

See for yourself!

```
customers = pd.read_sql_query('''SELECT * FROM Customer''', con)
```

```
customers = table("Customer")
```

These two lines of code accomplish the same task in creating a dataframe from the given sqlite table, Customer. However, my functions save me the trouble of fully typing the full query out every time.

Here is an example of how I used my query function.

**Question: Do discounts impact order volume?**

One of the hypothesis tests I did for this question was to determine the difference in individual order volume when a discount is present.

**H0:** The average individual order volume is the same, whether or not the customer's order contains discounted products.

**Ha:** There is a significant difference in the average individual order volume, depending on whether or not the order contains any discounted products.

For this hypothesis test, I used a two-tailed t-test, as the question asks whether the presence of discounted products increases or decreases the average order volume. I needed information from the Quantity and Discount columns in the OrderDetail table.

I used my query function to gather this data into a pandas dataframe.

```
disc_grouped = query("SELECT SUM (Quantity) FROM OrderDetail \
                      GROUP BY OrderId HAVING SUM (Discount) != 0")
full_grouped = query("SELECT SUM (Quantity) FROM OrderDetail \
                      GROUP BY OrderId HAVING SUM (Discount) = 0")
```

What this query did was it made a series of the total sum of the quantities from each order for the orders that included a discount, and a series of the total sum of the quantities from each order for the orders that did not include a discount.

```
# Visualizing the difference in order volume for orders that are full-priced vs. discounted

plt.figure(figsize=(8,5))
plt.hist(np.array(disc_grouped), bins=20, alpha=.25, label="Contains Discounted Products", density=True, color="red")
plt.hist(np.array(full_grouped), bins=20, alpha=.25, label="All Items Full Price", density=True, color="blue")
plt.title("Order Volume by Discount Present")
plt.ylabel("Relative Frequency of Order Quantity")
plt.xlabel("Individual Order Volume")
plt.legend()
plt.show()
```

![](https://thetiaramisu.files.wordpress.com/2019/03/order-volume-disccout-present-1.png)

These distributions were quite skewed, so I normalized them before performing tests a t-test.

```
# Log-transforming to normalize

disc_grouped_log = disc_grouped.copy(deep=True)
disc_grouped_log["SUM (Quantity)"] = np.log(disc_grouped_log["SUM (Quantity)"])

full_grouped_log = full_grouped.copy(deep=True)
full_grouped_log["SUM (Quantity)"] = np.log(full_grouped_log["SUM (Quantity)"])
```

After the log transformation, I reran the visualization to make sure it was appropriate to feed into the t-test.

```
# Visualizing the difference in order volume for orders that are full-priced vs. discounted after log transformation

plt.figure(figsize=(8,5))
plt.hist(np.array(disc_grouped_log), bins=20, alpha=.25, label="Contains Discounted Products", density=True, color="red")
plt.hist(np.array(full_grouped_log), bins=20, alpha=.25, label="All Items Full Price", density=True, color="blue")
plt.title("Order Volume by Discount Present (Log-Transformed)")
plt.ylabel("Relative Frequency of Order Quantity")
plt.xlabel("Transformed Individual Order Volume")
plt.legend()
plt.show()
```

![](https://thetiaramisu.files.wordpress.com/2019/03/order-volume-by-discount-present-log-transformed-1.png)

Seeing that it was decently normal, I went ahead and ran the two-sample t-test.

```
from scipy.stats import ttest_ind
tstat, pval = ttest_ind(disc_grouped_log,full_grouped_log)
print(f"The p-value is: {float(pval)}")
```

The returned p-value was: 9.032446365816016e-15

I then used a function to find Cohen's D.

```
# Function to test the Effect Size as Cohen's D

def Cohen_d(group1, group2):
    """Displays the calculation of Cohen's D - the Effect Size, \
    i.e. the standardized difference between two means
    Args: 
        group1: all values from one set of numbers
        group2: all values from a second set of numbers
    Returns: 
        The mean of the first set of numbers
        The mean of the second set of numbers
        The difference in these two means
        Cohen's d, the effect size between the two means
    """
		
    diff = group1.mean() - group2.mean()
    n1, n2 = len(group1), len(group2)
    var1 = group1.var()
    var2 = group2.var()

    # Calculate the pooled threshold as shown earlier
    pooled_var = (n1 * var1 + n2 * var2) / (n1 + n2)

    # Calculate Cohen's d statistic
    d = diff / np.sqrt(pooled_var)
    
    return print(f"First average = {round(float(group1.mean()),2)}\n \
        Second average = {round(float(group2.mean()),2)}\n \
        Difference in means = {round(float(diff),2)}\n \
        Cohen's d = {round(float(d),2)}")
```

`Cohen_d(disc_grouped,full_grouped)`

These were the results:

        First average = 72.94
        Second average = 52.44
        Difference in means = 20.5
        Cohen's d = 0.41

**Interpreting the Results:** The two-sample t-test returned a significant p-value. As a result, I rejected the null hypothesis, and accepted the alternative hypothesis stating that there is a significant difference in the average individual order volume when a discounted product is included.

Cohen's D showed us that there is a moderate effect size between the two means. On average, including a discounted product in the order increases the order volume by about 21 items.

If you are interested in seeing my project, you can find the complete Jupyter Notebook [here](https://thetiaramisu.wordpress.com/2019/03/13/query-functions-for-sqlite/).
