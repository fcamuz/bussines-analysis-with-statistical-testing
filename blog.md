![Download Dataset](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/title.png)

# Bussiness Analysis with Statistical Testing

The goal of this project is to do statistical analysis and hypothesis testing to generate valuable analytical insights for the company.  In this I will share one A/B test with you a one question. The question is **where and when Northwind company should start offering a new line of cheese.**
 

## Dataset:
Nortwind Dataset

![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/Northwind_ERD.png)

## Metodology:

- Scientific Method 
- EDA (Exploratory Data Analysis)

The general structure of an experiment is as follows:

1. Make an Observation and Come up with the Question

The first step of the scientific method is to observe something that you want to test. 

2. Examine the Data

In statistics, exploratory data analysis (EDA) is an approach to analyzing data sets to summarize their main characteristics, often with visual methods. A statistical model can be used or not, but primarily EDA is for seeing what the data can tell us beyond the formal modeling or hypothesis testing task. The main pillars of EDA are data cleaning, data preparation, data exploration, and data visualization. 

3. Form a Hypothesis

During this stage, you'll formulate 2 hypotheses to test--your educated guess about the outcome is called the Alternative Hypothesis, while the opposite of it is called the Null Hypothesis.

4. Run the Test

This step is the part of the scientific method that will be the focus of this section. You must pick the right test for you case. Moreover, each staistical test has a set of assumptions to be considered accurate. Make sure those assumptions are met and run the test.

5. Analyze Experimental Results

During this step, you will tease out relationships, filter out noise, and try to determine if something that happened is statistically significant or not. Depend on the test result, you will compare t,f,z, mostly p value with the alpha value to come up with conclusions.

6. Draw Conclusions

You've asked a question, looked at experimental results from others that could be related to your question, made an educated guess, designed an experiment, collected data and analyzed the results. The reality of this step is that you use your analysis of the data to do one of two things: either reject the null hypothesis or fail to reject the null hypothesis.


![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/new_the-scientific-method.png)



## Steps  
- Question
- Query the data
- EDA
- Write null hypothesis and alternative hypothesis
- Decide the test and Alpha level
- Compute the test statistics
- Analyze the result

## Statistical Tests

Statistical tests that performed for this A/B testing:
- two-way Anova test
- KS test 
- Shapiro test
- Tukey’s HSD test


## Bussines Problem

Where and when should Northwind company start offering a new line of cheese?
To be able answer explore the answer to this question we need to ask another question. **Does the revenue of dairy product (cheese) differs by any combination of region and the time of the years?** The answer is yes or no. We can figure this by A/B yesting. 

The data is in a SQL database. There are several ways to retreive data from such database. You can load all of the data in to dataframe at once and deal within just Python. But I will only query the data parts that I will use as a good practice of reducing the load for the processor. Since SQL is a lot faster than Pyhton in data processing, I also will do filtering and extracting data in SQL query as much as I can.

### Loading Data

I will create an engine to connect to SQL Database and send quesries as I needed.

#### Creating engine to connect to Database and send queries 

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

I will need from a few tables and do some feature engineering to calculate revenue.
```python
get_columns_info('Category')
```
```
Table Name: Category

Name: Id 	 Type: INTEGER
Name: CategoryName 	 Type: VARCHAR(8000)
Name: Description 	 Type: VARCHAR(8000)
```
```python
data1 = pd.read_sql_query('''
                            SELECT   *
                            FROM Category
                            
                            ;''', engine)
```

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
```python
byregion=data.groupby(['Region']).sum()
```
```python
ax1 = byregion.plot.barh( y='Revenue', rot=0)
ax2 = byregion.plot.barh( y='Quantity', rot=0)
```
![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/img1.png)

```python
bymonth=data.groupby(['OrderMonth']).sum()
ax2 = bymonth.plot.bar( y='Quantity', rot=0 )
```
![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/img2img2.png)


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

#### ANOVA Assumptions

* Each group sample is drawn from a normally distributed population
* All populations have a common variance
* All samples are drawn independently of each other
* Within each sample, the observations are sampled randomly and independently of each other
* Factor effects are additive
* Dependent variable should be continuous 
* Independent variables should be in categorical, independent groups

The presence of outliers can also cause problems. In addition, we need to make sure that the F statistic is well behaved. In particular, the F statistic is relatively robust to violations of normality provided:

* The populations are symmetrical and uni-modal.
* The sample sizes for the groups are equal and greater than 10

In general, as long as the sample sizes are equal (called a balanced model) and sufficiently large, the normality assumption can be violated provided the samples are symmetrical or at least similar in shape (e.g. all are negatively skewed).


#### Normality Check

```python
#KS-Test for checking normality
stats.kstest(data.Revenue, 'norm', args=(0,2))

KstestResult(statistic=0.9999893114742251, pvalue=0.0)
```
p_value is 0, so we can not say that the distribution is normal however when the sample size is sufficiently large, Anova can still perform well.
#### Equal Variances Check

```python
stats.levene(data['Revenue'][data['Region'] == "Western Europe"],
             data['Revenue'][data['Region'] == "South America"],
             data['Revenue'][data['Region'] == "Scandinavia"],
             data['Revenue'][data['Region'] == "North America"],
             data['Revenue'][data['Region'] == "Southern Europe"],
             data['Revenue'][data['Region'] == "Northern Europe"],
             data['Revenue'][data['Region'] == "British Isles"],
             data['Revenue'][data['Region'] == "Central America"],
             data['Revenue'][data['Region'] == "Eastern Europe"]
            )
LeveneResult(statistic=1.7172829986494036, pvalue=0.0932244407555185)
stats.levene(data['Revenue'][data['OrderMonth'] == "01"],
             data['Revenue'][data['OrderMonth'] == "02"],
             data['Revenue'][data['OrderMonth'] == "03"],
             data['Revenue'][data['OrderMonth'] == "04"],
             data['Revenue'][data['OrderMonth'] == "05"],
             data['Revenue'][data['OrderMonth'] == "06"],
             data['Revenue'][data['OrderMonth'] == "07"],
             data['Revenue'][data['OrderMonth'] == "08"],
             data['Revenue'][data['OrderMonth'] == "09"],
             data['Revenue'][data['OrderMonth'] == "10"],
             data['Revenue'][data['OrderMonth'] == "11"],
             data['Revenue'][data['OrderMonth'] == "12"]
            )
LeveneResult(statistic=1.6804048575969928, pvalue=0.07632782503794072)
```

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

You may check the OLS Regression Results and summary with summary() method.
```python
lm2.summary()
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

tukey.summary() 
```
```

Multiple Comparison of Means - Tukey HSD,FWER=0.05
group1	group2	meandiff	lower	upper	reject
01	02	-41.4511	-585.56	    502.6578	False
01	03	-165.2788	-632.914	302.3564	False
01	04	-56.087	    -510.3031	398.129	    False
01	05	-417.0923	-928.9763	94.7917	    False
01	06	-231.7972	-835.9195	372.325	    False
01	07	-20.8351	-538.3136	496.6434	False
01	08	-265.5782	-802.2951	271.1387	False
01	09	-89.9756	-634.0845	454.1333	False
01	10	-110.9483	-603.5997	381.7031	False
01	11	-67.8206	-574.4544	438.8132	False
01	12	-246.5678	-720.4803	227.3447	False
02	03	-123.8277	-662.478	414.8226	False
02	04	-14.636	    -541.6783	512.4064	False
02	05	-375.6412	-953.1243	201.8418	False
02	06	-190.3461	-850.9709	470.2787	False
02	07	20.616	    -561.8318	603.0638	False
02	08	-224.1271	-823.7323	375.4781	False
02	09	-48.5245	-654.7555	557.7064	False
02	10	-69.4972	-630.0031	491.0087	False
02	11	-26.3695	-599.2039	546.4648	False
02	12	-205.1167	-749.2256	338.9922	False
03	04	109.1918	-338.4709	556.8544	False
03	05	-251.8135	-757.8915	254.2644	False
03	06	-66.5184	-665.729	532.6922	False
03	07	144.4437	-367.2923	656.1796	False
03	08	-100.2994	-631.4817	430.8829	False
03	09	75.3032	    -463.3471	613.9535	False
03	10	54.3305	    -432.2854	540.9465	False
03	11	97.4582	    -403.3087	598.225	    False
03	12	-81.289	    -548.9242	386.3463	False
04	05	-361.0053	-854.71	    132.6995	False
04	06	-175.7102	-764.508	413.0877	False
04	07	35.2519	    -464.251	534.7549	False
04	08	-209.4912	-728.8987	309.9164	False
04	09	-33.8886	-560.9309	493.1538	False
04	10	-54.8612	-528.5959	418.8735	False
04	11	-11.7336	-499.9927	476.5255	False
04	12	-190.4807	-644.6968	263.7353	False
05	06	185.2951	-449.0517	819.6419	False
05	07	396.2572	-156.2066	948.721	    False
05	08	151.5141	-419.0095	722.0377	False
05	09	327.1167	-250.3664	904.5997	False
05	10	306.144	    -223.1362	835.4243	False
05	11	349.2717	-193.0474	891.5908	False
05	12	170.5245	-341.3595	682.4085	False
06	07	210.9621	-427.9077	849.832	    False
06	08	-33.781	    -688.331	620.769	    False
06	09	141.8216	-518.8032	802.4464	False
06	10	120.849	    -498.0824	739.7803	False
06	11	163.9766	-466.1412	794.0944	False
06	12	-14.7706	-618.8928	589.3517	False
07	08	-244.7431	-820.2915	330.8053	False
07	09	-69.1405	-651.5883	513.3073	False
07	10	-90.1132	-624.8059	444.5796	False
07	11	-46.9855	-594.5882	500.6172	False
07	12	-225.7327	-743.2112	291.7459	False
08	09	175.6026	-424.0027	775.2078	False
08	10	154.6299	-398.703	707.9629	False
08	11	197.7576	-368.0602	763.5753	False
08	12	19.0104	    -517.7065	555.7273	False
09	10	-20.9726	-581.4785	539.5332	False
09	11	22.155	    -550.6793	594.9894	False
09	12	-156.5921	-700.701	387.5167	False
10	11	43.1277	    -481.0766	567.3319	False
10	12	-135.6195	-628.2709	357.0319	False
11	12	-178.7472	-685.3809	327.8866	False
```
![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/img3.png)



As it shows in the reject column, non of those pairs are significantly different from each other in termes of mean revenue. However, you can see the small monthly flactuations from the plot.

Another way of doing this tst is directly through the statsmodels.stats.multicomp library. This time we can see the plots showing the confedence level of each group individually.

```python
import statsmodels.stats.multicomp 
# for comparing the product categories, we use visualizations
posthoc = statsmodels.stats.multicomp.MultiComparison(data['Revenue'], data['Region'])
posthoc_results = posthoc.tukeyhsd()
print('\n', posthoc_results)​
```
![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/img4.png)

```python
posthoc_results
```
```
<statsmodels.sandbox.stats.multicomp.TukeyHSDResults at 0x1c28950c18>
```
```python
regions=data.Region.unique()
```
```python
for i in regions:
    posthoc_results.plot_simultaneous(comparison_name=i, figsize=(6,4));
```
![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/img5.png)
![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/img6.png)
![img](https://github.com/fcamuz/bussines-analysis-with-statistical-testing/blob/master/images/img7.png)

## Conclusion

According to ANOVA tests result, we fail to reject the null hypothesis. So, neither region nor the time of the year does not effect the average revanue from dairy product significantly.

That would mean Cheese is a timeless product. It can be sold anywhere and anytime.
The company can invest on advertising so that it attracts more customer without worrying about the season and location.

Western Europe region had the most number of orders and the largest revenue.
Eastern Europe region are lowest on cheese sale.
The largest volume of cheese sale happens in January


Company sells cheese in 8 regions year round. I have run two-way ANOVA test to see if there is any difference of the cheese sale in between regions. Also tested the months of the year if there is any significant increase or decrease on the sales. Also tested if there is any best combination of the time of the year and the region. Test results suggest that no significant difference between any of the region nor time of the year for cheese sales. However, largest volume of the cheese sale happens in the Western Europe and overall cheese sales are highest in the month of January. 

## Based on A/B testing and data, my suggestion is Northwind Company would advertise and kick-off the sale of the new line of cheese in January in Western Europe region.

