# Heroes of Pymoli Data Analysis
* Observed trend 1

  Males are the dominant players in this game, account for 80% of all players.
  
  <br>

* Observed trend 2

  Players between age 20 and 24 make most of the purhcases.  Players over 40 years old buy the least amount of items, but the average price of items bought are the highest among all age groups.
  
  <br>
  

* Observed trend 3

  Item 'Arcane Gem' and 'Retribution Axe' are the most popular items by both sales count and sales value.
  
  <br>
  
  [Link to codes and results](https://github.com/nelsonxw/Pandas_Analysis/blob/master/HeroesOfPymoli_results.ipynb)



```python
# import dependencies
import pandas as pd
import os
```


```python
# define source data locations, import and combine data
file_1 = os.path.join('HeroesOfPymoli','purchase_data.json')
file_2 = os.path.join('HeroesOfPymoli','purchase_data2.json')
df_1 = pd.read_json(file_1)
df_2 = pd.read_json(file_2)
df = df_1.append(df_2,ignore_index=True)
```


```python
# Report unique player count
player_count = len(df['SN'].unique())
pd.DataFrame({'Total Players':[player_count]})
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Players</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>612</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Count unique items
item_count = len(df['Item Name'].unique())
# Calculate average price and change format
average_price = '${:,.2f}'.format(df['Price'].mean())
# Count total number of purchases
purchase_count = df['Item Name'].count()
# Calculate total revenue and change format
revenue = '${:,.2f}'.format(df['Price'].sum())
# Report purchase analysis
pd.DataFrame([[item_count,average_price,purchase_count,revenue]],
             columns=['Number of Unique Items',
                      'Average Purchase Price',
                      'Total Number of Purchases',
                      'Total Revenue'])
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Number of Unique Items</th>
      <th>Average Purchase Price</th>
      <th>Total Number of Purchases</th>
      <th>Total Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>180</td>
      <td>$2.93</td>
      <td>858</td>
      <td>$2,514.43</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Remove duplicate players from the purchase list
unique_df = df.drop_duplicates(['SN'])
# Count unique players by gender
gender_df = pd.DataFrame(unique_df['Gender'].value_counts())
# Change column header
gender_df.rename(columns = {'Gender':'Total Count'},inplace=True)
# Calculate the total number of unique players
gender_total = gender_df['Total Count'].sum()
# Add new column to show % of total players by gender
gender_df['Percentage of Players'] = (gender_df['Total Count'] / gender_total * 100).apply('{:,.2f}'.format)
# Move columns around and report out gender demographics
gender_df = gender_df[['Percentage of Players','Total Count']]
gender_df

```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Percentage of Players</th>
      <th>Total Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Male</th>
      <td>80.88</td>
      <td>495</td>
    </tr>
    <tr>
      <th>Female</th>
      <td>17.65</td>
      <td>108</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>1.47</td>
      <td>9</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Define column names
column_name = {'count':'Purchase Count','mean':'Average Purchase Price','sum':'Total Purchase Value'}
# Group by gender and count number of purchases, average price and total purchase price
purchase_df = df.groupby(['Gender']).agg({'Gender':'count','Price':['mean','sum']}).rename(columns = column_name)
# Remove level 0 of multi-index on the columns
purchase_df.columns = purchase_df.columns.droplevel(0)
# Calculate normalized totals and change the format
purchase_df['Normazlied Totals'] = (purchase_df['Total Purchase Value'] / gender_df['Total Count']).apply('${:,.2f}'.format)
# Change the format of average price
purchase_df['Average Purchase Price'] = purchase_df['Average Purchase Price'].apply('${:,.2f}'.format)
# Change the format of total purchase price
purchase_df['Total Purchase Value'] = purchase_df['Total Purchase Value'].apply('${:,.2f}'.format)
# Report out purchasing analysis by gender
purchase_df
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
      <th>Normazlied Totals</th>
    </tr>
    <tr>
      <th>Gender</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Female</th>
      <td>149</td>
      <td>$2.85</td>
      <td>$424.29</td>
      <td>$3.93</td>
    </tr>
    <tr>
      <th>Male</th>
      <td>697</td>
      <td>$2.94</td>
      <td>$2,052.28</td>
      <td>$4.15</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>12</td>
      <td>$3.15</td>
      <td>$37.86</td>
      <td>$4.21</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Create bins for different age groups
bins = [0,9,14,19,24,29,34,39,100]
# Define group labels
group_name = ['<10','10-14','15-19','20-24','25-29','30-34','35-39','40+']
# Assign age to differnt age groups
age_df = pd.cut(df['Age'],bins,labels = group_name)
# Add new column to show age group
df['Age group'] = age_df
# Count number of purchase by age group
age_group_count = df['Age group'].value_counts()
```


```python
# Group data by age group and calculate number of purchase, average price and total purchase price, rename column headers
age_df = df.groupby(['Age group']).agg({'SN':'count','Price':['mean','sum']}).rename(columns = column_name)
# Remove level 0 of multi-index on the columns
age_df.columns = age_df.columns.droplevel(0)
# Calculate normalized totals and change the format
age_df['Normalized Totals'] = (age_df['Total Purchase Value'] / age_group_count).apply('${:,.2f}'.format)
# Change the format of average price
age_df['Average Purchase Price'] = age_df['Average Purchase Price'].apply('${:,.2f}'.format)
# Change the format of total purchase price
age_df['Total Purchase Value'] = age_df['Total Purchase Value'].apply('${:,.2f}'.format)
# Report out age demographics and purhcase analysis
age_df
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
      <th>Normalized Totals</th>
    </tr>
    <tr>
      <th>Age group</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;10</th>
      <td>33</td>
      <td>$2.95</td>
      <td>$97.28</td>
      <td>$2.95</td>
    </tr>
    <tr>
      <th>10-14</th>
      <td>38</td>
      <td>$2.79</td>
      <td>$105.91</td>
      <td>$2.79</td>
    </tr>
    <tr>
      <th>15-19</th>
      <td>144</td>
      <td>$2.89</td>
      <td>$416.83</td>
      <td>$2.89</td>
    </tr>
    <tr>
      <th>20-24</th>
      <td>372</td>
      <td>$2.92</td>
      <td>$1,087.66</td>
      <td>$2.92</td>
    </tr>
    <tr>
      <th>25-29</th>
      <td>134</td>
      <td>$2.96</td>
      <td>$396.44</td>
      <td>$2.96</td>
    </tr>
    <tr>
      <th>30-34</th>
      <td>71</td>
      <td>$2.97</td>
      <td>$211.14</td>
      <td>$2.97</td>
    </tr>
    <tr>
      <th>35-39</th>
      <td>48</td>
      <td>$2.93</td>
      <td>$140.77</td>
      <td>$2.93</td>
    </tr>
    <tr>
      <th>40+</th>
      <td>18</td>
      <td>$3.24</td>
      <td>$58.40</td>
      <td>$3.24</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Group data by player and calcuate the number of purchases, average price and total purchase value
top_spender_df = df.groupby(['SN']).agg({'SN':'count','Price':['mean','sum']}).rename(columns = column_name)
# Remove level 0 of multi-index on the columns
top_spender_df.columns = top_spender_df.columns.droplevel(0)
# Select the top 5 players
top_spender_df = top_spender_df.nlargest(5,'Total Purchase Value')
# Change the format of average price
top_spender_df['Average Purchase Price'] = top_spender_df['Average Purchase Price'].apply('${:,.2f}'.format)
# Change the format of total purchase price
top_spender_df['Total Purchase Value'] = top_spender_df['Total Purchase Value'].apply('${:,.2f}'.format)
# Report out top spender analysis
top_spender_df
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>SN</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Undirrala66</th>
      <td>5</td>
      <td>$3.41</td>
      <td>$17.06</td>
    </tr>
    <tr>
      <th>Aerithllora36</th>
      <td>4</td>
      <td>$3.77</td>
      <td>$15.10</td>
    </tr>
    <tr>
      <th>Saedue76</th>
      <td>4</td>
      <td>$3.39</td>
      <td>$13.56</td>
    </tr>
    <tr>
      <th>Sondim43</th>
      <td>4</td>
      <td>$3.25</td>
      <td>$13.02</td>
    </tr>
    <tr>
      <th>Mindimnya67</th>
      <td>4</td>
      <td>$3.18</td>
      <td>$12.74</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Define column names
column_name = {'count':'Purchase Count','min':'Item Price','sum':'Total Purchase Value'}
# Group data by items and calcuate the number of purchases, item price (min price or max price) and total purchase value
popular_df = df.groupby(['Item ID','Item Name']).agg({'Item ID':'count','Price':['min','sum']}).rename(columns = column_name)
# Remove level 0 of multi-index on the columns 
popular_df.columns = popular_df.columns.droplevel(0)
# Select the top 5 sold items
popular_df = popular_df.nlargest(5,'Purchase Count')
# Change the format of item price
popular_df['Item Price'] = popular_df['Item Price'].apply('${:,.2f}'.format)
# Change the format of total purchase price
popular_df['Total Purchase Value'] = popular_df['Total Purchase Value'].apply('${:,.2f}'.format)
# Report out most popular items
popular_df
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Purchase Count</th>
      <th>Item Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th>Item Name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>84</th>
      <th>Arcane Gem</th>
      <td>12</td>
      <td>$2.23</td>
      <td>$29.34</td>
    </tr>
    <tr>
      <th>39</th>
      <th>Betrayal, Whisper of Grieving Widows</th>
      <td>11</td>
      <td>$2.35</td>
      <td>$25.85</td>
    </tr>
    <tr>
      <th>31</th>
      <th>Trickster</th>
      <td>10</td>
      <td>$2.07</td>
      <td>$23.22</td>
    </tr>
    <tr>
      <th>13</th>
      <th>Serenity</th>
      <td>9</td>
      <td>$1.49</td>
      <td>$13.41</td>
    </tr>
    <tr>
      <th>34</th>
      <th>Retribution Axe</th>
      <td>9</td>
      <td>$4.14</td>
      <td>$37.26</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Define column names
column_name = {'count':'Purchase Count','min':'Item Price','sum':'Total Purchase Value'}
# Group data by items and calcuate the number of purchases, item price (min price or max price) and total purchase value
profitable_df = df.groupby(['Item ID','Item Name']).agg({'Item ID':'count','Price':['min','sum']}).rename(columns = column_name)
# Remove level 0 of multi-index on the columns 
profitable_df.columns = profitable_df.columns.droplevel(0)
# Select the top 5 items by total purchase value
profitable_df = profitable_df.nlargest(5,'Total Purchase Value')
# Change the format of item price
profitable_df['Item Price'] = profitable_df['Item Price'].apply('${:,.2f}'.format)
# Change the format of total purchase price
profitable_df['Total Purchase Value'] = profitable_df['Total Purchase Value'].apply('${:,.2f}'.format)
# Report out most profitable items
popular_df
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Purchase Count</th>
      <th>Item Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th>Item Name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>84</th>
      <th>Arcane Gem</th>
      <td>12</td>
      <td>$2.23</td>
      <td>$29.34</td>
    </tr>
    <tr>
      <th>39</th>
      <th>Betrayal, Whisper of Grieving Widows</th>
      <td>11</td>
      <td>$2.35</td>
      <td>$25.85</td>
    </tr>
    <tr>
      <th>31</th>
      <th>Trickster</th>
      <td>10</td>
      <td>$2.07</td>
      <td>$23.22</td>
    </tr>
    <tr>
      <th>13</th>
      <th>Serenity</th>
      <td>9</td>
      <td>$1.49</td>
      <td>$13.41</td>
    </tr>
    <tr>
      <th>34</th>
      <th>Retribution Axe</th>
      <td>9</td>
      <td>$4.14</td>
      <td>$37.26</td>
    </tr>
  </tbody>
</table>
</div>


