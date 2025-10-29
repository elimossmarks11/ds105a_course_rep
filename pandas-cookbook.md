## DataFrames

All example code will specify which DataFrame is being used. Any amendments to the DataFrame will be outlined in the examples. Assume pandas is imported for all code. 

**DataFrame1**

``` python

df = pd.DataFrame(
    {"AAA": [4, 5, 6, 7], "BBB": [10, 20, 30, 40], "CCC": [100, 50, -30, -50]}
)

# Output

    AAA  BBB CCC
0    4   10  100
1    5   20   50
2    6   30  -30
3    7   40  -50
```

**DataFrame2**

``` python

df = pd.DataFrame({
    'month': ['Jan', 'Jan', 'Feb', 'Feb', 'Jan', 'Jan', 'Feb', 'Feb'],
    'product_category': ['Electronics', 'Electronics', 'Electronics', 'Electronics', 'Clothing', 'Clothing', 'Clothing', 'Clothing'],
    'revenue': [450, 380, 420, 350, 200, 180, 220, 195]
})

# Output
month     product_category  revenue
0   Jan      Electronics      450
1   Jan      Electronics      380
2   Feb      Electronics      420
3   Feb      Electronics      350
4   Jan         Clothing      200
5   Jan         Clothing      180
6   Feb         Clothing      220
7   Feb         Clothing      195
```
**DataFrame3**
```python

api_response = {
    'store': 'London Branch',
    'location': {
        'city': 'London',
        'postcode': 'SW1A 1AA'
    },
    'sales': [
        {
            'date': '2024-01-15',
            'product': {
                'category': 'Electronics',
                'name': 'Laptop'
            },
            'revenue': 450,
            'units': 25
        },
        {
            'date': '2024-01-22',
            'product': {
                'category': 'Electronics',
                'name': 'Phone'
            },
            'revenue': 380,
            'units': 18
        }
    ]
}
```

## Key commands to be familiar with

| Command | What it Does | Pattern |
|:--------:|:-------------:|:--------:|
| `.loc[]` | Pandas' location-based indexer that lets you select specific rows AND columns | `df.loc[row_selector, column_selector]` |
|`.groupby()`| Separates DataFrames, applies a calculation, returns results. | `df.groupby('column_to_group_by')['column_to_aggregate'].aggregation_function()`. Some common aggregation functions include `.mean(), .sum(), .min(), .max(), .count()`|
|`.dt` | Unlocks datetime-specific operations | `df['date'].dt.component`. Some common components are `.dt.year, .dt.month, .dt.day, .dt.hour, .dt.day_name(), .dt.month_name()`|
`.sort_values()`| Reorders entire DataFrame based on column values. | `df.sort_values('column_name', ascending=False)`|
|`pd.to_datetime()`| Converts UNIX timestamps to datetime objects. | `pd.to_datetime(df['unix_column'], unit='s', utc=True)`|
`pd.json_normalize()`| Flattens nested JSON into DataFrame columns| `pd.json_normalize(api_response['key'])`


## Building Multiple Conditions with pandas

**DataFrame:** DataFrame1

**What it does:** Translate pure python `and`, `or` logic and introduces other conditions for parsing pandas DataFrames.

### Two conditions using '&' / '|'

**Example:** 

The code and explanation below are the same for both `&` and `|` (or) conditions. They both return a Series:

```python
df.loc[(df["BBB"] < 25) & (df["CCC"] >= -40), "AAA"]
```
**Explanation / Key Points:** 

This code creates a _boolean mask_ (True/False value) for each row:

`df["BBB"] < 25` -> True for rows where column BBB is less than 25. 
`df["CCC"] >= -40` -> True for rows where column CCC is greater than or equal to -40. 

The two conditions _must_ be wrapped in parantheses for pandas' boolean logic to work.

### Two conditions with assignment 

**What it does:** Using the same code and logic as above, we can modify the DataFrame if certain conditions are met. 

**Example:**
```python
df.loc[(df["BBB"] < 25) | (df["CCC"] >= -40), "AAA"] = 999
```
**Explanation:**

For each row where _either_ `"BBB" < 25` or `"CCC" >= -40`, AAA will be replaced with 999. 

## Creating New Columns

**DataFrame:** DataFrame2

**What it does:** Adds new columns to DataFrames based on calculations / transformations. 

**Example:**
```python
# Create a new column from calculation
df['profit'] = [600, 400, 400, 400, 300, 200, 400, 205]

df['profit_margin'] = (df['profit'] / df['revenue']) * 100
```
**When to Use:**
Extracting datetime components (e.g. `df['year'] = df['date'].dt.year`), creating calculated fields, adding new categorical variables. 

## Conditional Assignment

**DataFrame:** DataFrame1

**What it does:** Changes values in one column based on conditions in another column.

**Example:** 
```python
df.loc[df.AAA >= 5, "BBB"] = -1
```

**Explanation:**

The `df.loc` command sets BBB to -1 if AAA ≥ 5. 

The condition `df.AAA >= 5` creates a True/False mask selecting which rows to modify.

The selected rows are set to -1.

**When to use:** 

Creating categorical variables (e.g. for air quality: poor/moderate/good). A more simple alternative to `np.select()`. 

## Sorting DataFrames

**DataFrame:** DataFrame2

**What it does:** Reorders rows based on column values.

**Example:**
```python
# Sort the entire DataFrame according to a single column (ascending)
df_sorted = df.sort_values('revenue')

# Sort descending
df_sorted = df.sort_values('revenue', ascending=False)

# Sort by multiple columns
df_sorted = df.sort_values(['month', 'revenue'], ascending=[True, False])

# Sort by date (essential for time series plots)
df_sorted = df.sort_values('date')
```
**When to use:**
- **Essential for time series plots** where dates must be in order.
- Finding top/bottom values in context. 
- Enhanced readability. 
- Preparing data for sequential operations.

## Grouping

**DataFrame:** DataFrame2

### Basic Grouping

**What it does:**

**Splits** DataFrames, **applies** a function to each group, **returns** results.

**Example:**

``` python
df.groupby('month')['revenue'].mean()
```
**Explanation:**

All df.groupby commands fit the following pattern:

```python
 df.groupby('CATEGORY_COLUMN')['VALUE_COLUMN'].AGGREGATION()
#                 ↑                 ↑                ↑
#            How to split     What to measure  How to summarize 
```

In this case, data is split by ```'month'```, ```'revenue'``` data is measured, before being summarized by its mean. 

### Counting Group Sizes

**What it does:** Count items in each category.

**Example:**
```python
df.groupby('quality').size()
```

### Aggregating with multiple functions

**What it does:** Calculates several statistics at once for each group. Returns a DataFrame with one column per aggregation function.

**Example:**
```python
df.groupby('month')['revenue'].agg(['mean', 'max', 'min', 'count'])

# Output
  
         mean  max  min  count
month
Feb    296.25  420  195      4
Jan    302.50  450  180      4
```

**When to use:** When you need comprehensive summary statistics.

### Grouping by Multiple Columns

**What it does:** Creates nested groups for detailed analysis. 

**Example:**
```python
df.groupby(['month', 'product_category'])['revenue'].mean()

# Output
month  product_category
Feb    Clothing            207.5
       Electronics         385.0
Jan    Clothing            190.0
       Electronics         415.0

Name: revenue, dtype: float64
```
**When to use:** When you need to break down data by multiple dimensions e.g. the mean revenue for each category per month. 

### Multiple Column Aggregations

**What it does:** Calculates statistics for several columns simultaneously. 

**Example:**

```python
# Add units_sold and profit columns

df['units_sold'] = [25, 18, 22, 16, 45, 38, 50, 42]
df['profit'] = [135, 114, 126, 105, 60, 54, 66, 59]

# Generate statistics for multiple columns

df.groupby('month')[['revenue', 'units_sold', 'profit']].mean()

# Output

       revenue  units_sold  profit
month
Feb     296.25        32.5   89.00
Jan     302.50        31.5   90.75
```
**Explanation:** Use _double brackets_ to select multiple columns. This returns a DataFrame with one column per metric and one row per month.

**When to use:** Comparing multiple metrics across groups. 

### Resetting Index (after groupby)

**DataFrame:** DataFrame2

**What it does:** Converts index back to regular columns (useful after groupby).

**Example:**
```python
# After groupby, year becomes the index
yearly_avg = df.groupby('year')['revenue'].mean()
print(yearly_avg)
# Output:
  year
  2020    350
  2021    400
  Name: revenue, dtype: int64

# Reset index to make 'year' a column again
yearly_avg = yearly_avg.reset_index()
print(yearly_avg)
# Output:
     year  revenue
  0  2020      350
  1  2021      400
```
**When to use:**
- After groupby, before creating plots
- When you need to use the grouped column in further operations
- Making results easier to work with

## Timeseries

**DataFrame:** DataFrame2

### Converting Unix timestamps to Datetime

**What it does:** Converts Unix timestamps (seconds since January 1, 1970) into pandas datetime objects that are readable. 

**Example:**

```python
# Add a date column to the sales data containing UNIX timestamps

df['unix_timestamp'] =[1705276800, 1705881600, 1707264000, 1708387200, 1704844800, 1706400000, 1707782400, 1708905600]

# Convert to datetime objects

df['date'] = pd.to_datetime(df['unix_timestamp'], unit='s', utc=True)
print(df[['unix_timestamp', 'date', 'revenue']].head())

# Output

  unix_timestamp          date                revenue
0      1705276800 2024-01-15 00:00:00+00:00      450
1      1705881600 2024-01-22 00:00:00+00:00      380
2      1707264000 2024-02-07 00:00:00+00:00      420
3      1708387200 2024-02-20 00:00:00+00:00      350
4      1704844800 2024-01-10 00:00:00+00:00      200

```
**Explanation:**

`unit='s'` -> timestamps are in seconds (most common). Other unit options include `unit='ms'` for milliseconds, `unit='us'` for microseconds (check API documentation
)
`utc=True` = tells pandas that the timestamps are in UTC timezone.

### Datetime Accessors

**What it does:** Extracts specific components from datetime columns.

```python

# Now, we can extract year/month/day

df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day_name'] = df['date'].dt.day_name()

print(df[['year', 'month', 'day_name']].head())

# Output

   year  month   day_name
0  2024      1     Monday
1  2024      1     Monday
2  2024      2  Wednesday
3  2024      2    Tuesday
4  2024      1  Wednesday
```
### Using Datetime Accessors with groupby

**Example**
```python
# Group sales by day of the week
df.groupby(df['date'].dt.day_name())['revenue'].mean()
```

## Data In / Out Operations

**DataFrame:** DataFrame2

### Reading and Writing CSV Files

**What is does:** Saves DataFrames to CSV files and loads them back. 

**Example: Writing CSV**

```python
# Save sales data to a CSV file in a 'data' folder.
df.to_csv('data/sales_data.csv', index=False)
```
**Explanation:** The `index=False` prevents pandas from saving row numbers as a column. 

**Example: Reading CSV**
```python
# Load CSV back into DataFrame
df_loaded = pd.read_csv('data/sales_data.csv')
```

**Example: Save only specific columns**
```python
df.to_csv('data/revenue_only.csv', columns = ['date', 'revenue'], index=False)
```
### JSON Normalization

**DataFrame:** DataFrame3

**What it does:** Flattens nested JSON structures (e.g. API responses) into DataFrame columns. 

**Example**
```python
# Print original nested structure of the API response (DataFrame3)

print(api_response['sales'][0])

# Output

{'date': '2024-01-15', 'product': {'category': 'Electronics', 'name': 'Laptop'}, 'revenue': 450, 'units': 25}

# Use json_normalize to flatten the nested data

df = pd.json_normalize(api_response['sales'])
print(df)

# Output

         date  revenue  units product.category product.name
0  2024-01-15      450     25      Electronics       Laptop
1  2024-01-22      380     18      Electronics        Phone
```
**Explanation:***
`pd.json_normalize()` transforms:
1. Nested keys into column names with dots e.g. `product['category']` becomes `product.category`.
2. Lists of dictionaries into DataFrame rows.

### Renaming Columns

**DataFrame:** DataFrame3

**What it does:** Changes column names to remove nested prefixed / ensure clarity. 

**Example:**
```python
# After json_normalize, two columns are prefaced by 'product.' e.g. 'product.category'. These can be removed:
df.columns = df.columns.str.replace('product.', '')

# Rename specific columns
df = df.rename(columns={'old_name': 'new_name', 'another_old': 'another_new'})

# Rename multiple columns at once
df.columns = ['date', 'income', 'amount', 'category', 'name'] # This must match the number of columns!
```

**Explanation:** 
```python
 df.columns.str.replace('being_replaced', 'replacing_with')
```
**When to use:** 
- Cleaning up after `json_normalize`
- Making column names more readable
- Following naming conventions. 

### Selecting Columns

**DataFrame:** DataFrame2

**What it does:** Extracts specific columns from a DataFrame.

**Example:**
```python
# Single column (returns Series)
revenue = df['revenue']

# Multiple columns (returns DataFrame)
subset = df[['month', 'revenue']]

# Drop columns you don't need
df_clean = df.drop(columns=['unix_timestamp', 'extra_column'])
```

**When to use:**
- After `json_normalize()` to keep only relevant data
- Creating focused analysis datasets
- Removing unnecessary columns before saving CSV