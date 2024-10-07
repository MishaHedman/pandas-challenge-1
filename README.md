# pandas-challenge-1

import pandas as pd
file = "../Resources/client_dataset.csv"
df   = pd.read_csv('Resources/client_dataset.csv')
df.head()

# View the column names in the data
column_names = df.columns
print(df.columns)

# Use the describe function to gather some basic statistics
df.describe()

# Use this space to do any additional research
# and familiarize yourself with the data.
df.head()

most_order_year = df.groupby('order_year')['qty'].sum().idxmax()
most_order_year

least_order_year =df.groupby('order_year')['qty'].sum().idxmin()
least_order_year

no_order_year = df.groupby('order_year')['qty'] == 0
no_order_year

# What three item categories had the most entries?
most_entries_df = df['category'].value_counts().head(3)
most_entries_df
consumables_df = most_entries_df
most_entries_df

# For the category with the most entries, which subcategory had the most entries?
top_sub_df = df.loc[df['category'] == 'consumables','subcategory'].value_counts()
top_sub_df

# Which five clients had the most entries in the data?
most_client_entries_df = df['client_id'].value_counts().head()
most_client_entries_df

# Store the client ids of those top 5 clients in a list.
top_5_clients = most_client_entries_df.index.tolist()
top_5_clients

# How many total units (the qty column) did the client with the most entries order order?
top_client_total_units = df.loc[df['client_id'] == 33615, 'qty'].sum()
top_client_total_units

#Part Two Transform The Data

# Create a column that calculates the subtotal for each line using the unit_price and the qty
df['subtotal'] = df['unit_price'] * df['qty']
df.head()

# Create a column for shipping price.
# Assume a shipping price of $7 per pound for orders over 50 pounds and $10 per pound for items 50 pounds or under.
df['total_weight'] = df['unit_weight'] * df['qty']
df['shipping_price'] = df['total_weight'].apply(lambda x: 7 * x if x > 50 else 10 * x)
df.head()

# Create a column for the total price using the subtotal and the shipping price along with a sales tax of 9.25%
df['total_price'] = round((df['subtotal'] + df['shipping_price']) * 1.0925, 2)
df.head()

# Create a column for the cost of each line using unit cost, qty, and
# shipping price (assume the shipping cost is exactly what is charged to the client).
df['line_cost'] = df['unit_cost'] + df['qty'] + df['shipping_price']
df.head()

# Create a column for the profit of each line using line cost and line price
df['profit'] = df['total_price'] - df['line_cost']
df.head()

#Part Three: Confirm Your Work

# Check your work using the totals above
order_ids = [2742071, 2173913, 6128929]

def check_order_totals(order_ids):
    line_totals = df.loc[df['order_id'] == order_ids, 'total_price'].sum()
        # Return the calculated total
        return round(line_totals, 2)  

# Call the function and print the result
for order in order_ids: 
    print(f"{check_order_totals (order)}, {order}")

#Part Four: Summarize and Analyze

# How much did each of the top 5 clients by quantity spend? Check your work from Part 1 for client ids.
top_5 = list(df['client_id'].value_counts().head(5).index)

#top_clients_by_qty = df.groupby('client_id')['qty'].sum().nlargest(5)

def client_totals(top_5_clients, column):
    client_df = df.loc[df['client_id'] == top_5_clients, column]
    return round(client_df.sum(), 2)

for client_id in top_5_clients:
    print(f"{client_id}: ${client_totals(client_id, 'total_price')}")

print(df.columns)

# Create a summary DataFrame showing the totals for the for the top 5 clients with the following information:
# total units purchased, total shipping price, total revenue, and total profit. 
summary_df = {'client_id': top_5} 
columns = ['qty', 'shipping_price', 'line_cost', 'profit']
for col in columns:
    summary_df[col] = [client_totals(client_id, col) for client_id in top_5] 

summary_df1 = pd.DataFrame(summary_df)
print(summary_df1) 

# Format the data and rename the columns to names suitable for presentation.
summary_df1.rename(columns={'client_id' : 'Client_ID',  'qty' : 'Quantity', 'shipping_price' : 'Shipping Price', 'line_cost' : 'Total Cost', 'profit' : 'Profit'}, inplace=True)
summary_df1
# Define the money columns. 
money_columns = ['Client_ID', 'Quantity', 'Shipping Price', 'Total Cost', 'Profit']
summary_df1[money_columns] = summary_df1[money_columns].astype(float)

# Define a function that converts a dollar amount to millions.
#def convert_dollar(row):
def convert_dollar(row):
        if isinstance(row, (int, float)):
            return row / 1_000_000
        else:
            return row
       
# Apply the currency_format_millions function to only the money columns. 
summary_df1[money_columns] = summary_df1[money_columns].apply(convert_dollar)
currency_format_millions = summary_df1[money_columns]
currency_format_millions.head()

# Rename the columns to reflect the change in the money format. 
currency_format_millions = currency_format_millions.rename(columns={
    'Client_ID' : 'Client ID',
    'Quantity' : 'Quantity',
    'Shipping Price' : 'Shipping Price(millions)', 
    'Total Cost' : 'Total Cost(millions)', 
    'Profit' : 'Total Profit (millions)'})
currency_format_millions

#currency_format_millions = summary_df1[money_columns].applymap(lambda x: '${:,.2f}'.format(x))
print(currency_format_millions)

# Sort the updated data by "Total Profit (millions)" form highest to lowest and assign the sort to a new DatFrame.

Total_Profit_Sorted = currency_format_millions.sort_values(by='Total Profit (millions)', ascending=False)
new_summary_df = Total_Profit_Sorted
new_summary_df.head()

[Findings: The company's largest order year is 2023.  They sell more comsumable products than any other product they sell.]