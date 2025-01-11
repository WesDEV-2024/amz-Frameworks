# Framework for Analysis and Python Code Implementation

## Objective:
To analyze data from the SQP Report and Business Report to identify opportunities for low CTR ASINs, improve their performance, and provide actionable insights in a structured format.

---

## Steps Followed:

### 1. Data Preparation:
**Input Files:**
- `SQP Report.csv`
- `Business Report.csv`

**Cleaning:**
- Invalid values (e.g., `#VALUE!`) in percentage columns were replaced with `NaN`.
- Percentage values (e.g., `20%`) were converted to numeric format by stripping the `%` sign and dividing by 100.

---

### 2. Keyword-Level Metrics:
**Grouped data by ASIN and Search Query to calculate average metrics:**
- **Clicks: Click Rate (%)**: Average click-through rate.
- **ASIN CTR**: Comparison of CTR for individual ASINs.
- **ASIN CVR**: Conversion rate at the keyword level.

**Added scenario-based incremental sales estimates:**
- CTR increases (e.g., 20%, 30%, 40%, 50%).
- CVR increases (e.g., 5%, 10%, 15%, 20%).

---

### 3. ASIN-Level Metrics:
**Grouped data by ASIN to calculate overall metrics:**
- **ASIN CTR and CVR**: Aggregated at the ASIN level.
- **Comparisons of ASIN CTR** against benchmarks like Click Rate % and Share %.

**Scenario-based incremental sales estimates added** similar to the keyword-level metrics.

---

### 4. CTR & CVR Opportunity Matrix:
**Revenue Impact Matrix:**
- For all ASINs and separately for low CTR ASINs.
- Low CTR ASINs identified as those with a CTR below a defined threshold (10% in this analysis).
- **Estimated revenue impact** for combinations of CTR and CVR improvements.

---

### 5. Analysis Table for Low CTR ASINs:
- **Combined keyword-level insights** for ASINs identified as low CTR.
- **Recommendations** in four focus areas:
  1. **Keyword Optimization:** Targeting and relevance improvements.
  2. **Ad Campaigns:** Use of PPC to drive traffic.
  3. **Content Improvements:** Enhancing titles, bullet points, and images.
  4. **Pricing and Offers:** Evaluate pricing strategies and promotions.

---

## Python Code Implementation:

```python
import pandas as pd
import ace_tools as tools

# Load the input files
sq_report_path = 'SQP Report.csv'
business_report_path = 'Business Report.csv'
sq_report = pd.read_csv(sq_report_path)
business_report = pd.read_csv(business_report_path)

# Step 1: Data Cleaning
columns_to_clean = ['Clicks: Click Rate %', 'Clicks: ASIN Share %', 'ASIN CTR', 'ASIN CVR']
for column in columns_to_clean:
    sq_report[column] = pd.to_numeric(sq_report[column].str.rstrip('%'), errors='coerce') / 100

# Step 2: Keyword-Level Metrics
ctr_increments = [0.2, 0.3, 0.4, 0.5]
cvr_increments = [0.05, 0.1, 0.15, 0.2]
keyword_level = sq_report.groupby(['ASIN', 'Search Query']).agg({
    'Clicks: Click Rate %': 'mean',
    'Clicks: ASIN Share %': 'mean',
    'ASIN CTR': 'mean',
    'ASIN CVR': 'mean'
}).reset_index()
for increment in ctr_increments:
    keyword_level[f'Scenario {int(increment * 100)}% CTR Increase'] = (
        keyword_level['ASIN CTR'] * (1 + increment) - keyword_level['ASIN CTR']
    ) * 100
for increment in cvr_increments:
    keyword_level[f'Scenario {int(increment * 100)}% CVR Increase'] = (
        keyword_level['ASIN CVR'] * (1 + increment) - keyword_level['ASIN CVR']
    ) * 100

tools.display_dataframe_to_user(name="Keyword Level Metrics", dataframe=keyword_level)

# Step 3: ASIN-Level Metrics
asin_level = sq_report.groupby('ASIN').agg({
    'Clicks: Click Rate %': 'mean',
    'Clicks: ASIN Share %': 'mean',
    'ASIN CTR': 'mean',
    'ASIN CVR': 'mean'
}).reset_index()
for increment in ctr_increments:
    asin_level[f'Scenario {int(increment * 100)}% CTR Increase'] = (
        asin_level['ASIN CTR'] * (1 + increment) - asin_level['ASIN CTR']
    ) * 100
for increment in cvr_increments:
    asin_level[f'Scenario {int(increment * 100)}% CVR Increase'] = (
        asin_level['ASIN CVR'] * (1 + increment) - asin_level['ASIN CVR']
    ) * 100

tools.display_dataframe_to_user(name="ASIN Level Metrics", dataframe=asin_level)

# Step 4: CTR & CVR Opportunity Matrix
low_ctr_threshold = 0.1
low_ctr_asins = asin_level[asin_level['ASIN CTR'] < low_ctr_threshold]
low_ctr_base_revenue = low_ctr_asins['ASIN CTR'].mean() * 1000000
low_ctr_impact_matrix = pd.DataFrame(
    index=[f"CVR {int(cvr * 100)}%" for cvr in cvr_increments],
    columns=[f"CTR {int(ctr * 100)}%" for ctr in ctr_increments],
    data=[[low_ctr_base_revenue * (1 + ctr) * (1 + cvr) for ctr in ctr_increments] for cvr in cvr_increments]
)
asin_column = ['; '.join(low_ctr_asins['ASIN'].tolist())] * len(low_ctr_impact_matrix)
low_ctr_impact_matrix.insert(0, "ASINs", asin_column)
tools.display_dataframe_to_user(name="Low CTR ASINs: CTR & CVR Opportunity Matrix (With ASINs)", dataframe=low_ctr_impact_matrix)

# Step 5: Analysis Table for Low CTR ASINs
low_ctr_keywords = keyword_level[keyword_level['ASIN'].isin(low_ctr_asins['ASIN'])]
analysis_table = low_ctr_keywords[['ASIN', 'Search Query', 'Clicks: Click Rate %', 'ASIN CTR', 'ASIN CVR']].copy()
analysis_table['Keyword Optimization Recommendation'] = (
    "Focus on improving targeting and relevance for keyword: " + analysis_table['Search Query']
)
analysis_table['Ad Campaign Recommendation'] = "Consider PPC campaigns to increase CTR for this ASIN."
analysis_table['Content Improvement Recommendation'] = "Optimize product titles, bullet points, and images."
analysis_table['Pricing and Offers Recommendation'] = "Evaluate pricing strategies and add promotions to improve CTR and CVR."
tools.display_dataframe_to_user(name="Analysis Table for Low CTR ASINs", dataframe=analysis_table)

```

## Outputs:
- Keyword Level Metrics: Detailed keyword-level CTR and CVR data with scenarios.
- ASIN Level Metrics: Aggregated ASIN-level metrics with actionable insights.
- CTR & CVR Opportunity Matrix for Low CTR ASINs: Revenue potential based on increases in CTR and CVR.
- Analysis Table for Low CTR ASINs: Specific recommendations for optimizing low-performing ASINs.
