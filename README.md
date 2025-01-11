# lookerstudio-dashboard-structural-framework

---

## Step 1: Data Loading
1. **Import necessary libraries**: `pandas`, `matplotlib.pyplot`, `os`.
2. **Load Data**: Load `campaign_data.csv` and `search_query_data.csv` into DataFrames using `pd.read_csv()`.
3. **Verify Data**: Display the first few rows of each DataFrame to confirm the data is loaded correctly.

---

## Step 2: Data Merging
1. **Merge DataFrames**: Perform an inner join between `campaign_data.csv` and `search_query_data.csv` on common columns such as `Date` and `SearchQuery` (or `ASIN` if available).
2. **Verify Merging**: Display the first few rows of the merged DataFrame to ensure successful merging.

---

## Step 3: Data Cleaning and Preparation
1. **Convert Date Format**: Convert the `Date` column to `datetime` format using `pd.to_datetime()`.
2. **Create New Date Dimensions**:
   - `WeekNumber`: Extract week of the year starting on Sunday.
   - `Month`, `Quarter`, and `Year`: Extract corresponding date features from the `Date` column.
3. **Ensure Numeric Columns**: Convert numeric columns (`Clicks`, `Impressions`, `Spend`, `TotalOrders`, `TotalSales`) to proper numeric types and replace any invalid values with 0.
4. **Rename Columns for Clarity**:
   - `Customer Search Term` → `SearchQuery`
   - `7 Day Total Orders (#)` → `TotalOrders`
   - `7 Day Total Sales` → `TotalSales`
5. **Preview Data**: Display the first few rows of the cleaned DataFrame for review.

---

## Step 4: Calculating Metrics
1. **Derived Metrics**:
   - **CTR (%)**: `(Clicks / Impressions) * 100`
   - **CVR (%)**: `(TotalOrders / Clicks) * 100`
   - **ACoS (%)**: `(Spend / TotalSales) * 100`
   - **ROAS**: `(TotalSales / Spend)`
   - **AOV**: `(TotalSales / TotalOrders)`
2. **Handle Missing Values**: Replace any infinite or NaN values with 0.
3. **Preview Data**: Display the DataFrame with calculated metrics for verification.

---

## Step 5: Data Cleaning
1. **Remove Duplicates**: Ensure there are no duplicate rows in the dataset.
2. **Replace Infinite Values**: Replace all occurrences of `inf` with 0 in the dataset.
3. **Round Percentages**: Round all percentage values to two decimal places.

---

## Step 6: Data Aggregation
1. **Aggregate Data**: Group the data by `Date`, `WeekNumber`, `Month`, `Quarter`, `Year`, `ASIN`, and `SearchQuery`.
2. **Calculations**:
   - **Sums**: Aggregate `Impressions`, `Clicks`, `Spend`, `TotalOrders`, `TotalSales`.
   - **Averages**: Calculate the averages for metrics such as `CTR (%)`, `CVR (%)`, `ACoS (%)`, `ROAS (%)`, `AOV (%)`.
3. **Reset Index**: Reset the index to prepare the aggregated DataFrame for export.
4. **Preview Data**: Display the aggregated DataFrame for review.

---

## Step 7: Merging ASINs Without Adding Rows
1. **Aggregate the Key Data**: Ensure the ASIN mapping table (`key_data.csv`) has one unique ASIN per `SearchQuery`. Use:
   ```python
   key_data_unique = key_data.groupby('SearchQuery').agg({'ASIN': 'first'}).reset_index()
   ```
2. **Perform LEFT JOIN**: Merge the weekly aggregated data with the ASIN mapping using a LEFT JOIN to ensure no extra rows are introduced:
   ```python
   final_data = pd.merge(aggregated_data, key_data_unique, on='SearchQuery', how='left')
   ```
3. **Verify Rows**: Confirm that the row count of the merged DataFrame matches the original aggregated data.
4. **Preview Final Data**: Display the merged DataFrame for review, ensuring ASINs are added correctly without duplicates.

---

## Step 8: Exporting Data
1. **Export Final Data**: Save the cleaned and aggregated dataset with added ASINs to a CSV file:
   ```python
   final_data.to_csv('final_data_with_asins.csv', index=False)
   ```
2. **Verify Export**: Check that the file is saved with the correct structure and data.

---


