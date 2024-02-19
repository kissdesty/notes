  

```Python
def momentum_score(df: object, end_date: object = '2022-01-25') -> object:
    MOM = 0
    for i in range(1, 13):
        MOM += 0 if (df[:end_date].pct_change(i)[end_date] < 0) else 1
    return MOM
```
 
![](https://skinny-wash-0dd.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fc0fb9408-a27b-44c1-8c69-f2c9b43c704e%2Fd9e02ca9-2352-4fa6-b518-2dc8ef9c7500%2FUntitled.png?table=block&id=a981b1b1-97db-4da2-b8db-a28b1f1c2c59&spaceId=c0fb9408-a27b-44c1-8c69-f2c9b43c704e&width=1190&userId=&cache=v2)

```Python

@jit(nopython=True)
def calculate_change(prices, end_date_index):
    MOM = 0
    for i in range(1, 13):  # Loop through the last 12 periods
        if end_date_index - i >= 0:  # Ensure we don't go out of bounds
            change = (prices[end_date_index] - prices[end_date_index - i]) / prices[end_date_index - i]
            MOM += int(change >= 0)
    return MOM

# Modify the momentum_score function to use the JIT-compiled function
def momentum_score(df, end_date = '2022-01-25'):
    # print(f"Inspecting DataFrame up to {end_date}:")
    df_sliced = df.loc[:end_date]
    # print(df_sliced.tail())  
    
    # Find the numeric index of the specified end_date within the DataFrame's index.
    # This is done by comparing each date in the index to end_date (converted to a pandas Timestamp),
    # using np.where to find where the comparison is True, and selecting the first matching index.
    # This index is needed for the calculate_change function to identify the end point for calculation.
    end_date_index = np.where(df.index == pd.to_datetime(end_date))[0][0]
    
    # Extract the values (prices) from the sliced DataFrame for processing.
    # This conversion to a numpy array is necessary for Numba-optimized calculations

    prices = df_sliced.values


    return calculate_change(prices, end_date_index)

```