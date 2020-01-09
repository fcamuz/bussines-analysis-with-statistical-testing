![Download Dataset](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/title.png)

# Bussiness Analysis with Statistical Testing

The goal of this project is to do statistical analysis and hypothesis testing to generate valuable analytical insights for the company. 

## Dataset:
Nortwind Dataset

![Download Dataset](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/Northwind_small.sqlite)

![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/Northwind_ERD.png)

## Metodology:

- Scientific Method 
- EDA (Exploratory Data Analysis)

The general structure of an experiment is as follows:

1. Make an Observation and Come up with the Question

The first step of the scientific method is to observe something that you want to test. During this step, you must observe phenomena to help refine the question that you want to answer.

2. Examine the Data

In statistics, exploratory data analysis (EDA) is an approach to analyzing data sets to summarize their main characteristics, often with visual methods. A statistical model can be used or not, but primarily EDA is for seeing what the data can tell us beyond the formal modeling or hypothesis testing task. The main pillars of EDA are data cleaning, data preparation, data exploration, and data visualization. 

The particular graphical techniques employed in EDA are often quite simple, consisting of various techniques of: 
Plotting the raw data (such as data traces, histograms, bihistograms, probability plots, lag plots, block plots, and Youden plots.

Plotting simple statistics such as mean plots, standard deviation plots, box plots, and main effects plots of the raw data.

Positioning such plots so as to maximize our natural pattern-recognition abilities, such as using multiple plots per page.

3. Form a Hypothesis

During this stage, you'll formulate 2 hypotheses to test--your educated guess about the outcome is called the Alternative Hypothesis, while the opposite of it is called the Null Hypothesis.

4. Run the Test

This step is the part of the scientific method that will be the focus of this section. You can only test a hypothesis by gathering data from a well-structured experiment.You must pick the right test for you case. Moreover, each staistical test has a set of assumptions to be considered accurate. Make sure those assumptions are met and run the test.

5. Analyze Experimental Results

During this step, you will tease out relationships, filter out noise, and try to determine if something that happened is statistically significant or not. Depend on the test result, you will compare t, f,z, mostly p value with the alpha value to come up with conclusions.

6. Draw Conclusions

This step is the logical endpoint for an experiment. You've asked a question, looked at experimental results from others that could be related to your question, made an educated guess, designed an experiment, collected data and analyzed the results. The reality of this step is that you use your analysis of the data to do one of two things: either reject the null hypothesis or fail to reject the null hypothesis.


![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/new_the-scientific-method.png)

## Questions
3 questions to explore

- How to make good use of discounts?
- Is there a generation gap to fill in the performance?
- Where and when to start offering new cheese types?

## Steps  
The Order of Hyphotesis Testing for each question as follows:
- Question
- Get the Data
- EDA
- Write null Hypothesis and alternative Hypothesis
- Decide the test and Alpha level
- Compute the test statistics
- Analyze the result

## Statistical Tests
Statistical tests that performed in this project:
- Welch's t-test
- one-way Anova test
- two-way Anova test
- KS Test 
- Shapiro Test
- Cohen's d test
- Tukey

## Libraries
Python libraries used for this project:
sqlalchemy, statmodels, itertools, pandas, numpy, stats from scipy, matplotlib, seaborn

## Project Summary & Code Highlishts

I only query the data parts that I will use as a good practice of reducing the load for the processor. Since SQL is a lot faster than Pyhton in data processing, I also did filtering and extracting data in SQL query as much as I can.

### Creating engine to connect to Database and send queries 

```python
# connecting to the Database using SQLAlchemy
engine = create_engine("sqlite:///Northwind_small.sqlite")
​
# create an inspector object
inspector = inspect(engine)
​
# get names of tables in database
print(inspector.get_table_names())
['Category', 'Customer', 'CustomerCustomerDemo', 'CustomerDemographic', 'Employee', 'EmployeeTerritory', 'Order', 'OrderDetail', 'Product', 'Region', 'Shipper', 'Supplier', 'Territory']
# write a quick function that prints the columns of a table and each column's type
​
def get_columns_info(table_name):    
    """ Function takes in table name and prints columns' names
    and datatypes """
    
    cols_list = inspector.get_columns(table_name)    
    print(f"Table Name: {table_name}\n")
    for col in cols_list:
        print(f"Name: {col['name']} \t Type: {col['type']}")
# calling the function to test it
get_columns_info('OrderDetail')
```



## Question 1

1.1 - Does discount amount have a statistically significant effect on the quantity of a product in an order? If so, at what level(s) of discount?

1.2 - Retrieve the data with SQL query
Out of all the columns, I will only need Quantity and Discount columns from Orderdetails table.

```python
data = pd.read_sql_query('''SELECT Discount, Quantity
                            FROM OrderDetail
                            
                            ;''', engine)
data.head()
Discount	Quantity
0	0.0	12
1	0.0	10
2	0.0	5
3	0.0	9
4	0.0	40
```
```python
I want those columns to be grouped by and ordered by Discount levels.

discount_levels = pd.read_sql_query('''SELECT Discount, sum(Quantity) as Quantity
                            FROM OrderDetail
                            GROUP BY Discount
                            ORDER BY Discount
        
                            ;''', engine)
discount_levels
```
```           
         
Discount	Quantity
0	0.00	28599
1	0.01	2
2	0.02	4
3	0.03	5
4	0.04	1
5	0.05	5182
6	0.06	2
7	0.10	4366
8	0.15	4456
9	0.20	4351
10	0.25	4349

```

This question explored in two steps. First I verified if there is a statisticaly significant difference between the orders have discount and the orders have not discount. Then I determined the effect of different levels of discounts.


### 1.3 Hypothesis

𝐻0: Offering discount does not affect the average number of product in an order.

𝐻0:𝜇1=𝜇2

𝐻1: Offering discount affects the average number of product in an order.

𝐻0:𝜇𝑖≠𝜇𝑗
I set my significance level (Alpha): 0.05
𝛼 = 0.05

### 1.4 Statistical Analysis

Welch’s t-test is a nonparametric univariate test that tests for a significant difference between the mean of two unrelated groups. It is an alternative to the independent t-test when there is a violation in the assumption of equality of variances. In our case we have slighly different variences so I will use Welch's t-test. With this close standart deviation, probably, Student's t-test would also be ok but let's go by Welch's t-test beacuse, it is also best for the categorical indipendant variables.

#### Welch’s t-test Assumptions

Like every test, this inferential statistic test has assumptions. The assumptions that the data must meet in order for the test results to be valid are:

The independent variable (IV) is categorical with at least two levels (groups) 

The dependent variable (DV) is continuous which is measured on an interval or ratio scale

The distribution of the two groups should follow the normal distribution; 
We can check the normality of a distribution with a few different method such KS Test or Shapiro Test. The first thing we need to do is import scipy.stats as stats and then test our assumptions. We can test the assumption of normality using the stats.shapiro(). The first value in the tuple is the W test statistic, and the second value is the p-value.

#### Normality Check
```python
stats.shapiro(disc.Quantity)
(0.8673426508903503, 6.471277454941499e-26)
stats.shapiro(non_disc.Quantity)
(0.8434571027755737, 3.803856556577728e-34)
#KS Test for normality
stats.kstest(non_disc.Quantity, 'norm', args=(0,2))
​KstestResult(statistic=0.9187836569660197, pvalue=0.0)
```


p_value is very small for both tests, so we can not say that the distribution is normal however when the sample size is sufficiently large, Welch's t-test can still perform well.
#### Equal variance check 
```python
#checkign if the variances are equal
disc.std()==non_disc.std()
```
Discount    False
Quantity    False
dtype: bool

#### Performing Welch's t-test
```python
from scipy import stats​
stats.ttest_ind(disc.Quantity, non_disc.Quantity, equal_var=False)
​
Ttest_indResult(statistic=6.511219067380875, pvalue=1.0051255540843165e-10)
```


P _value is very small (1.7401995278127614e-11), it is safe to reject the null hypothesis suggesting the alternative. We can say that the discount has a statistically significant effect on the quantity average of the product in an order. But we do not know how much of a difference it makes. To figure that out, let's check the effect size.

#### Effect Size
```python
Cohen_d(disc.Quantity,non_disc.Quantity)
0.2997078720940889
```

Cohen suggested that d=0.2 be considered a 'small' effect size, 0.5 represents a 'medium' effect size and 0.8 a 'large' effect size. Eventhough we did the two tailed test (equla or not equal check), we can say that the impact is pozitive by looking at the effect size. Since it is positive, discounts resulted increased quantity in orders.

Having 0.28 means that if two groups' means differ by 0.28 standard deviations the effect is small but, still statistically signficant.

The second part of question is in what level the discount make significant difference in the order quantity
I will check the effect sizes for all different discount values in a simple loop.
```python

discount_levels=[0.05,0.1,0.15,0.20,0.25]
effect_sizes_on_quantity= []
#effect_sizes['Discount']= discount_levels
for i in discount_levels:
    sample=disc.loc[disc['Discount']==i]['Quantity']
    effect_sizes_on_quantity.append(round ( Cohen_d(sample, non_disc.Quantity),3))
    print('Effect Size for',i, 'Discount :', round ( Cohen_d(sample, non_disc.Quantity),3))
​
```

## Question 2

2.1 - Does discount increase the profit? If so in what level it allows best profit?
2.2 - Retrieve the data with SQL Query

I need the profit amount for each product to test. Lets check if there is such information in product and orderdetail tables.

```python
product = pd.read_sql_query('''SELECT *
                            FROM Product
                            LIMIT 11
                            ;''', engine)
product
```


To be able to calculate the profit, I need to know the buy and the sale prices of each product. In the entire dataset, there is no mention to buy or sell price distinction. However, there are two UnitPrice value available in both OrderDetail and Product tables. But they are not the same number. In fact in the product table, most of the unit prices are more expensive then ones in the OrderDetail table. Another insight is, QuantityPerUnit is in Product table is for bulk amounts such as 10 boxes of 20 bags or 50 bags x 30 sausgs. Most probably the unit price for these bulk amounts are the price that company pays to suppliers. And the unit price in the OrderDetail is the company's sale price for one unit in that bulk. So in that case, 10 boxes x 20 bags of Chai are bought for 18.00 dolars from the supplier and 1 box of 20 bags of Chai is sold for 14.00 dolars to the end customer. That makes perfect sense.

So, to be able calculate the profit, I need to do a little feature engineering and math here. First I need to extract the Quantity of units from the Quantityperunit column which is a string value. I checked the entire column, it seems consistent that the number of units are the first number in the string until the space. So I will use .split() method to get that number and convert to integer and store in another column. Let's get started.

```python
product_order = pd.read_sql_query('''
                                SELECT  p.Id,p.ProductName, p.QuantityPerUnit, p.UnitPrice as BulkPrice, 
                                        o.Unitprice as UnitSalePrice, o.Quantity QuantityInOrder, o.Discount
​
                                FROM Product p
                                JOIN OrderDetail o ON ProductId
                               
                        
                                ;''', engine)
product_order.head()
```

#### Extracting the number of units in a bulk
The number of units are the first number in the string until the space however, there was one exception that caused error. I spotted that and add an if statement to catch it an convert it to a number manually.

```python
quantityinbulk=[]
for i in range(len(product_order.QuantityPerUnit)):
        n=product_order.QuantityPerUnit[i].split()[0]
        if n =='1k':
            n=1000
        quantityinbulk.append(int(n))
   
     
product_order['QuantityInBulk']= quantityinbulk
product_order['QuantityInBulk'].head()
0    10
1    24
2    12
3    48
4    36
Name: QuantityInBulk, dtype: int64
product_order.head()
​
```

### 2.3 Hypothesis
𝐻0: Offering discount does not affect the average profit in an order.

𝐻0:𝜇1=𝜇2

𝐻1: Offering discount affects the average profit in an order.

𝐻0:𝜇𝑖≠𝜇𝑗
I set my significance level (Alpha): 0.05
𝛼 = 0.05

### 2.4 Statistical Analysis
Welch’s t-test is a nonparametric univariate test that tests for a significant difference between the mean of two unrelated groups. It is an alternative to the independent t-test when there is a violation in the assumption of equality of variances. In our case we have slighly different variences so I will use Welch's t-test. With this close standart deviation, probably, Student's t-test would also be ok but let's go by Welch's t-test beacuse, it is also best for the categorical indipendant variables.

Welch’s t-test Assumptions check 
Welch's t-test
EEffect Size


![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/Slide5.png)


These 2  charts show the effects of discount levels on quantity (on the left) and profit (on the right) , comparing to the sales with no discount. 
As it shows on the left chart, all discount levels increase the sale amount. However,  for 10%, 15% and 20% discounts, the sale amount increases but that amount of sale does not compensate the decreased price (as you can see on the left chart). So the company makes less money than they would have made without discount. Simply put, if no (10, 15, 20% ) discount would have applied, company would sell less product but make more profit.  
 According to this data, I would suggest to stick with the 5% and 25% discounts. 
5% discount exceptionally works well for making more sale and more profit. 

## Question 3

3.1 - Does the employee's generation effect the average quantity of products sold?
3.2 - Retrieve the data and perform EDA


#### Birth year should be extracted from the BirthDate column. It is a varchar type so I can use split.

```python
birth_year=[]
for i in range(len(birthday.BirthDate)):
        birth_year.append(int(birthday.BirthDate[i].split('-', 1)[0]))
```

#### labeling generations
```python

generations=[]
for n,i in enumerate (birth_year):
    #print (i, n)
    if i<=1945:
        generations.append('Silent')
    elif i>=1946 and i<=1964:
        generations.append('Boomer')
    elif i>=1965 and i<=1980:
        generations.append('Gen_x')
    elif i>=1981 and i<=1996:
        generations.append('Millennial')
    else :
        generations.append('Gen_z')


```
### 3.3 Hypothesis
𝐻0: Employee's generation does not have impact on the number of products sold.

𝐻0:𝜇1=𝜇2=𝜇3=𝜇4...=𝜇𝑖=𝜇𝑗

𝐻1: Employee's generation has impact on the number of products sold.

𝐻0:𝜇𝑖≠𝜇𝑗
I set my significance level (Alpha): 0.05
𝛼 = 0.05


### 3.4 Statistical Analysis

The one-way ANOVA compares the means between the groups you are interested in and determines whether any of those means are statistically significantly different from each other. If, however, the one-way ANOVA returns a statistically significant result, we accept the alternative hypothesis (HA), which is that there are at least two group means that are statistically significantly different from each other. So ANOVA would be good for this situation.

ANOVA Assumptions
Each group sample is drawn from a normally distributed population
All populations have a common variance
All samples are drawn independently of each other
Within each sample, the observations are sampled randomly and independently of each other
Factor effects are additive
The presence of outliers can also cause problems. In addition, we need to make sure that the F statistic is well behaved. In particular, the F statistic is relatively robust to violations of normality provided:

The populations are symmetrical and uni-modal.
The sample sizes for the groups are equal and greater than 10
In general, as long as the sample sizes are equal (called a balanced model) and sufficiently large, the normality assumption can be violated provided the samples are symmetrical or at least similar in shape (e.g. all are negatively skewed).

Normality Check
Equal Variances Check
```python
# using Levene’s test to test for equal variances between groups
stats.levene(birthday['Quantity'][birthday['Generation'] == "Millennial"],
             birthday['Quantity'][birthday['Generation'] == "Gen_z"],
             birthday['Quantity'][birthday['Generation'] == "Gen_x"])
LeveneResult(statistic=0.06774940223865293, pvalue=0.9344966195919657)
​
```
Performing One-way ANOVA
​
```python
formula = 'Quantity ~ C(Generation)'
lm = ols(formula, birthday).fit()
table = sm.stats.anova_lm(lm, typ=2)
print(table)
```
 ![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/Slide6.png)

 Three different generations are working together at Northwind Company. Majority of the sale force population are millennials who are  in between 22-37 years old.  There are employees from Gen X and Gen Z as well.  If there is any significant difference between the generations performances,  the company would update their training strategies.

According to the result of the statistical tests, there is no significant difference between  performances of different generations in the company.  Employees from all ages would perform almost equally in terms of sales. We can confidently say that companies training policy is working across the generations. 
That is good news. 

We also observe experienced employees perform slightly better than the less experienced ones.Which is highly expected.  
I would suggest mentorship program among the employees to support the young ones to catch up with the others. 


## Question 4
4.1 - Does the revenue of dairy product (cheese) differs by any combination of region and the time of the years?

4.2 - Retrieve the data with SQL query

I will need from a few tables and do some feature engineering to calculate revenue.
I will only select the dairy product for cheeses, calculate and select the revenue, region and month of the order from data. To be able to get the month information, I need to select the month info as a substring from the orderdate column because it is all VARCHAR format.
```python
data = pd.read_sql_query('''
                            SELECT   
                                ((od.Quantity * od.UnitPrice)* (1-od.Discount)) as Revenue , c.Region, od.Quantity,
                                SUBSTR(o.OrderDate, 6, 2) OrderMonth 
                            FROM Product p 
                            JOIN OrderDetail od ON p.Id =od.ProductId
                            JOIN [Order] o ON od.OrderId = o.Id
                            JOIN Customer c ON o.CustomerId = c.Id
​
                            WHERE p.CategoryId=4
                            
                          
                                    
                        ;''', engine)
​
```



### 4.3 Hypothesis

Main effect of factor A ("Order Region"):

𝐻0: The average revenue in an order for dairy products are equal for all regions. 

𝐻1: The average revenue in an order for dairy products are different at least for one region.

Main effect of factor B ("Order Month"):

𝐻0: The average revenue for in an order dairy products are equal for all months of a year. 𝐻1: The average revenue for in an order dairy products are different at least for one month of the year.

Factor A x factor B interactions:

𝐻0: There is no interaction between "Order Region" and "Order Month". 𝐻1: There is an interaction between "Order Region" and "Order Month".

I set my significance level (Alpha): 0.05
𝛼 = 0.05

### 4.4 Statistical Analysis

The two-way ANOVA examines the effect of two factors (month and region) on a dependent variable – in this case revenue, and also examines whether the two factors affect each other to influence the continuous variable.

ANOVA Assumptions

Each group sample is drawn from a normally distributed population

All populations have a common variance

All samples are drawn independently of each other

Within each sample, the observations are sampled randomly and independently of each other

Factor effects are additive

Dependent variable should be continuous

Independent variables should be in categorical, independent groups

The presence of outliers can also cause problems. In addition, we need to make sure that the F statistic is well behaved. In particular, the F statistic is relatively robust to violations of normality provided:


The populations are symmetrical and uni-modal.

The sample sizes for the groups are equal and greater than 10

In general, as long as the sample sizes are equal (called a balanced model) and sufficiently large, the normality assumption can be violated provided the samples are symmetrical or at least similar in shape (e.g. all are negatively skewed).


#### Normality Check
#### Equal Variances Check

#### Performing two-way ANOVA Test
```python
formula = 'Revenue ~ C(Region)*C(OrderMonth)'
lm2 = ols(formula, data).fit()
table = sm.stats.anova_lm(lm2, typ=2)
print(table)
​
                               sum_sq     df         F    PR(>F)
C(Region)                3.899516e+06    8.0  1.223844  0.289722
C(OrderMonth)            3.794354e+05   11.0  0.086607  0.999906
C(Region):C(OrderMonth)  1.961990e+07   88.0  0.559783  0.997149
Residual                 1.059440e+08  266.0       NaN       NaN
```

By looking at the p value, we fail to reject null hypothesis for factor A ,C and Factor A and B interaction. That mean we can suggest that the average revenue for dairy products are not significantly different at anytime of the year and at any region. However, we can still look at those categories to see the small differences.

There are a few different methods of post-hoc testing to find a difference between groups of factors. I will use Tukey’s HSD from 2 different libraries. It basically compares each pair combination available in a group.

```python
from statsmodels.stats.multicomp import pairwise_tukeyhsd
​
tukey = pairwise_tukeyhsd(endog=data.Revenue,     # Data
                          groups=data.OrderMonth,   # Groups
                          alpha=0.05)          # Significance level
​
tukey.plot_simultaneous()  
```

As it shows in the reject column, non of those pairs are significantly different from each other in termes of mean revenue. However, you can see the small monthly flactuations from the plot.

Another way of doing this tst is directly through the statsmodels.stats.multicomp library. This time we can see the plots showing the confedence level of each group individually.

```python
import statsmodels.stats.multicomp 
# for comparing the product categories, we use visualizations
posthoc = statsmodels.stats.multicomp.MultiComparison(data['Revenue'], data['Region'])
posthoc_results = posthoc.tukeyhsd()
print('\n', posthoc_results)
​
```
​


According to ANOVA tests result, we fail to reject the null hypothesis. So, neither region nor the time of the year does not effect the average revanue from dairy product significantly.

That would mean Cheese is a timeless product. It can be sold anywhere and anytime.
The company can invest on advertising so that it attracts more customer without worrying about the season and location.
Western Europe region had the most number of orders and the largest revenue.
Eastern Europe region are lowest on cheese sale.
Averadly the largest volume of cheese sale happens in January



## Conclusion

The reason for Eastern Europe having low orders should be investigated.
Conclusions for all 4 tests

I have run the two sample Welch's t-test to investigate discount effects on quantity and another Welch's t-test on the profit. All discount levels increase the sale amount. However, since the price drops with the discounts, at some levels, discount does not help for overall profit to increase. For 10%, 15% and 20% discounts, the sale amount increases but the new sale amount does not compromise the decreased price. So the company makes less money then the average profit on those sales with no discount. Simply put, if no discount would have applied, company would sale less product but make more money on 10%, 15% and 20% discounts. According to this data, I would suggest stick with the 5% and 25% discounts. 5% discount exceptionally works well for making more sale and more profit.

Three different generations are working together at Northwind Company. Majority of the sale force population are millennials who are in between 22-37 years old. There are employees from Gen X and Gen Z as well. According to the result of one-way ANOVA test, there is no significant difference between performances of different generations in the company. Employees from all ages would perform almost equally in terms of sales. We can confidently say that companies sales policy and trainings are working across the generations in the company. That is good news. We also observe experienced employees perform slightly better than the less experienced ones. I would suggest mentorship among the employees to support the young ones to catch up with the others.

Company sells cheese in 8 regions year round. I have run two-way ANOVA test to see if there is any difference of the cheese sale in between regions. Also tested the months of the year if there is any significant increase or decrease on the sales. Also tested if there is any best combination of the time of the year and the region. Test results suggest that no significant difference between any of the region nor time of the year for cheese sales. However, largest volume of the cheese sale happens in the Western Europe and overall cheese sales are highest in the month of January. I would advertise and kick-off the sale of the new line of cheese in Januaryin Western Europe region.

