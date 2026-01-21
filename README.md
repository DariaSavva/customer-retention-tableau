# Customer Retention Cohort Analysis

A Tableau cohort analysis dashboard that visualizes customer retention rates over time, enabling businesses to track customer loyalty, identify retention patterns, and measure the effectiveness of customer success initiatives.

![Tableau](https://img.shields.io/badge/Tableau-E97627?style=flat&logo=Tableau&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-success)

## 📊 Live Dashboard

**[View on Tableau Public](https://public.tableau.com/app/profile/daria.savvateeva/viz/CustomerLifetimeRetention/RetentionRate#1)**

---

## 🖼️ Dashboard Preview

![Customer Retention Cohort Analysis](screenshots/customer retention.png)

*Cohort retention matrix showing quarterly customer retention rates from acquisition through 15+ quarters*

---

## 💼 Business Value

### What is Cohort Analysis?

Cohort analysis groups customers by their acquisition date (cohort) and tracks their behavior over time. This retention matrix shows what percentage of each customer cohort continues to make purchases in subsequent periods.

### Why Cohort Analysis Matters

**Without cohort analysis**, you might see:
- Total customer count growing → Assume business is healthy
- Average retention rate → Doesn't show if newer or older customers are more loyal

**With cohort analysis**, you can:
- **Identify if retention is improving** - Are recent cohorts more loyal than older ones?
- **Measure product-market fit** - High early retention suggests strong value proposition
- **Calculate customer lifetime value (CLV)** - Predict long-term revenue from each customer
- **Validate business changes** - Did that new feature launched in Q2 2022 improve retention?
- **Detect warning signs early** - Declining retention in recent cohorts signals problems

### Critical Business Questions Answered

1. **"Are we getting better at keeping customers?"**  
   Compare retention curves between early cohorts (2021) and recent cohorts (2023-2024)

2. **"What's our expected customer lifetime?"**  
   Track when retention rates flatten out (customer maturity)

3. **"Which acquisition periods brought the most loyal customers?"**  
   Identify best-performing cohorts and replicate those conditions

4. **"When do most customers churn?"**  
   Spot drop-off periods and implement intervention strategies

5. **"What's the ROI of our customer success program?"**  
   Compare retention before/after program implementation

---

## ✨ Key Features

- **Quarterly Cohort Grouping**: Customers grouped by acquisition quarter
- **15-Quarter Tracking**: Follow retention up to 15 quarters post-acquisition
- **Color-Coded Heat Map**: Darker blue = higher retention, lighter = lower retention
- **Triangular Pattern**: Recent cohorts show fewer periods (haven't existed long enough)
- **New Customer Count**: Shows cohort size for context
- **100% Baseline**: Quarter 0 always shows 100% (all customers start here)
- **Percentage Display**: Clear retention rate labels in each cell

---

## 📈 How to Read This Dashboard

### Understanding the Table

**Rows (Acquisition Quarter)**: When customers first made a purchase  
**Columns (Quarter Number)**: Time since acquisition  
**Values**: % of original cohort still active

### Example Interpretation

**2021 Q1 Cohort:**
- Started with 123 customers (100%)
- Quarter 1: 27% still active (33 customers)
- Quarter 2: 24% still active (30 customers)
- Quarter 15: 54% still active (66 customers)

**Key Insight**: This cohort shows increasing retention over time, possibly indicating customers who stay past the first year become very loyal.

### What Good Looks Like

✅ **Strong retention curve**: Gradual decline that flattens out  
✅ **Improving cohorts**: Recent cohorts retain better than old ones  
✅ **High long-term retention**: 30%+ still active after 8+ quarters  
✅ **Small initial drop**: <50% churn in first 2-3 quarters

### Warning Signs

⚠️ **Steep drop-off**: Losing 50%+ in first quarter  
⚠️ **Declining trend**: Newer cohorts perform worse  
⚠️ **No plateau**: Continuous decline without flattening  
⚠️ **Inconsistent patterns**: Widely varying retention across similar periods

---

## 🛠️ How to Build This in Tableau

### Prerequisites

- Tableau Desktop or Tableau Public
- Dataset with:
  - **Customer ID** (unique identifier)
  - **Order Date** (transaction date)
  - **Order ID** (to count transactions)

**Sample Data Structure:**
```
Customer ID | Order Date  | Order ID
------------|-------------|----------
CUST001     | 2021-01-15  | ORD-001
CUST001     | 2021-04-20  | ORD-002
CUST002     | 2021-02-10  | ORD-003
```

**Note**: This dashboard uses Tableau's built-in "Sample - EU Superstore" dataset. You can use any dataset with customer purchase history.

---

### Step 1: Identify First Purchase Date

Create a calculated field to find when each customer first purchased:

#### First Purchase Date (LOD Calculation)
```tableau
{FIXED [Customer ID]: MIN([Order Date])}
```

**What this does**: For each customer, finds their very first order date regardless of filters.

---

### Step 2: Determine Acquisition Quarter

Group customers by the quarter they were acquired:

#### Acquisition Quarter
```tableau
DATETRUNC('quarter', [First Purchase Date])
```

**Alternative with better labels**:
```tableau
DATENAME('quarter', [First Purchase Date]) 
+ " " + 
STR(YEAR([First Purchase Date]))
```
Example output: "Q1 2021"

---

### Step 3: Calculate Quarters Since Acquisition

Determine how many quarters have passed since each customer's first purchase:

#### Quarters Since Acquisition
```tableau
DATEDIFF('quarter', [First Purchase Date], [Order Date])
```

**What this does**: 
- If order was in same quarter as first purchase → 0
- If order was 1 quarter later → 1
- If order was 2 quarters later → 2
- And so on...

---

### Step 4: Count New Customers per Cohort

Create a calculation that counts unique customers acquired in each quarter:

#### New Customers
```tableau
{FIXED [Acquisition Quarter]: COUNTD([Customer ID])}
```

**Purpose**: Shows cohort size for context (larger cohorts may have different patterns).

---

### Step 5: Count Retained Customers

For each cohort and quarter combination, count how many customers are still active:

#### Retained Customers
```tableau
COUNTD([Customer ID])
```

**Note**: This is just a distinct count - Tableau will aggregate based on your dimensions.

---

### Step 6: Calculate Retention Rate

The key metric - what percentage of the original cohort is still active:

#### Retention Rate
```tableau
COUNTD([Customer ID]) / [New Customers]
```

**Format**: Percentage, 0 decimal places

**Logic**:
- Numerator: How many customers from this cohort made purchases in this period
- Denominator: How many customers were in this cohort originally
- Result: % still active

---

### Step 7: Build the Cohort Table

#### 7.1 Create Worksheet
1. Create new worksheet
2. Name it "Retention Rate" or "Cohort Analysis"

#### 7.2 Add Dimensions
1. **Drag [Acquisition Quarter] to Rows**
   - Right-click → Sort → Descending (most recent on bottom)
   
2. **Drag [Quarters Since Acquisition] to Columns**
   - Ensure it's discrete (blue pill)
   - Right-click → Sort → Ascending

#### 7.3 Add Metrics
1. **Drag [Retention Rate] to Text**
   - This displays the percentage in each cell

2. **Drag [Retention Rate] to Color**
   - Creates heat map visualization
   - High retention = darker color
   - Low retention = lighter color

3. **Drag [New Customers] to Rows**
   - Place it after Acquisition Quarter
   - Right-click → Dual Axis → No (keep separate)
   - This shows cohort size

#### 7.4 Change Mark Type
1. Click on Marks card
2. Change from Automatic to **Square**
3. Increase size using Size slider

---

### Step 8: Format the Visualization

#### 8.1 Color Scheme
1. Click Color on Marks card
2. Choose a sequential palette:
   - **Blue** (light to dark) - recommended
   - **Teal** or **Blue-Green** work well
3. Click "Reversed" if needed (darker = higher retention)
4. Adjust stepped color if desired

#### 8.2 Text Labels
1. Click Label on Marks card
2. Format: **Percentage, 0 decimals**
3. Font: Bold, appropriate size (10-12pt)
4. Alignment: Center
5. Click "Show mark labels"

#### 8.3 Remove Headers (Optional)
- For cleaner look, keep headers
- For presentation, you can customize axis titles

#### 8.4 Borders
1. Format → Borders
2. Row Divider: Light gray
3. Column Divider: Light gray
4. Cell: White or light gray

#### 8.5 Title
- Right-click title → Edit Title
- Example: "Customer Retention by Quarter"
- Add subtitle explaining the view

---

### Step 9: Add Context and Labels

#### 9.1 Column Headers
The quarters since acquisition (0, 1, 2, 3...) are self-explanatory.

Optionally add a text note:
- "0 = Acquisition quarter"
- "1 = One quarter later"
- etc.

#### 9.2 Cohort Size Column
The "New Customers" column shows cohort size - this is crucial context:
- Larger cohorts may have different retention patterns
- Helps assess statistical significance

#### 9.3 Dashboard Title
Add a clear title explaining the metric:
- "Customer Retention by Acquisition Cohort"
- "Quarterly Cohort Retention Analysis"

---

### Step 10: Add Interactivity (Optional)

#### 10.1 Filters
Add filters for:
- **Date Range**: Focus on specific time periods
- **Product Category**: Compare retention across product lines
- **Customer Segment**: B2B vs B2C, etc.

#### 10.2 Tooltips
Enhance tooltips to show:
```
Acquisition Quarter: <Acquisition Quarter>
Quarters Since Acquisition: <Quarters Since Acquisition>
Retention Rate: <AGG(Retention Rate)>
Customers Active: <COUNTD(Customer ID)>
Original Cohort Size: <New Customers>
```

#### 10.3 Reference Lines
Add reference lines:
- Industry benchmark retention rate
- Company target retention rate

---

## 📊 Advanced Variations

### Monthly Cohorts
Change granularity for more detailed analysis:
```tableau
// Acquisition Month
DATETRUNC('month', [First Purchase Date])

// Months Since Acquisition  
DATEDIFF('month', [First Purchase Date], [Order Date])
```

**Use case**: More granular for businesses with short sales cycles

---

### Revenue Retention
Track revenue instead of customer count:
```tableau
// Revenue Retention Rate
SUM([Sales]) / {FIXED [Acquisition Quarter]: SUM([Sales])}
```

**Insight**: Shows if retained customers are spending more or less over time

---

### Cumulative Retention
Show cumulative percentages:
```tableau
// Customers who have returned at least once by this quarter
COUNTD(IF [Quarters Since Acquisition] <= [Current Quarter] 
       THEN [Customer ID] END) 
/ [New Customers]
```

---

### Churn Rate View
Flip the metric to show churn instead of retention:
```tableau
// Churn Rate
1 - [Retention Rate]
```

**Use case**: Some stakeholders prefer to see churn directly

---

## 🎯 Business Applications

### 1. Product-Market Fit Validation
**Question**: "Do customers love our product enough to stick around?"  
**How**: Look at long-term retention (quarters 8-12)  
**Good**: >30% retention at quarter 12  
**Action**: If low, investigate product value proposition

### 2. Customer Success Program ROI
**Question**: "Is our onboarding/CS program working?"  
**How**: Compare cohorts before/after program launch  
**Example**: 2022 Q1 vs 2021 Q1 retention curves  
**Action**: Expand or modify program based on results

### 3. Pricing Strategy Validation
**Question**: "Did the price increase hurt retention?"  
**How**: Compare retention of cohorts before/after price change  
**Watch for**: Steeper drop-off in post-change cohorts  
**Action**: Consider grandfathering or value adds

### 4. Marketing Channel Effectiveness
**Question**: "Which acquisition channels bring quality customers?"  
**How**: Segment cohorts by acquisition channel  
**Compare**: Paid ads vs organic vs referrals  
**Action**: Invest more in channels with better retention

### 5. Seasonal Insights
**Question**: "Do Q4 holiday customers stick around?"  
**How**: Compare Q4 cohort retention to other quarters  
**Finding**: Often lower (gift buyers, deal seekers)  
**Action**: Targeted re-engagement campaigns

---

## 🔄 Next Steps After Building

### 1. Establish Baselines
- Calculate your current retention metrics
- Document as benchmarks for future comparison

### 2. Set Targets
- Define "good" retention for your business
- Set improvement goals (e.g., +5% retention by Q4)

### 3. Segment Analysis
Build separate views for:
- Customer segments (B2B vs B2C)
- Product categories
- Geographic regions
- Acquisition channels

### 4. Regular Monitoring
- Review monthly or quarterly
- Look for trends in recent cohorts
- Investigate sudden changes

### 5. Action Planning
- Identify weak retention periods
- Develop targeted interventions
- Measure impact on subsequent cohorts

---

## 📝 Files Included

```
customer-retention-tableau/
├── dashboards/
│   └── Customer_Lifetime_Retention.twbx    # Tableau workbook
├── screenshots/
│   └── customer_retention.png              # Dashboard preview
├── .gitignore
├── LICENSE
└── README.md                               # This file
```

**Note**: This dashboard uses Tableau's built-in "Sample - EU Superstore" dataset. No separate data files needed.

---

## 🚀 Getting Started

### View Online
Visit the [Tableau Public link](https://public.tableau.com/app/profile/daria.savvateeva/viz/CustomerLifetimeRetention/RetentionRate#1) to interact with the dashboard.

### Download and Customize
1. Download `Customer_Lifetime_Retention.twbx` from the `dashboards/` folder
2. Open in Tableau Desktop or Tableau Public
3. Connect to your own customer data
4. Adjust calculations for your business logic (monthly vs quarterly cohorts)
5. Customize colors and formatting

---

### Key Metrics to Track Alongside
- **Customer Acquisition Cost (CAC)**
- **Customer Lifetime Value (CLV)**
- **Monthly Recurring Revenue (MRR)**
- **Churn Rate**
- **Net Revenue Retention**

---

## 📄 License

This project is available under the MIT License - feel free to use and modify for your own purposes.

---

## 👤 Author

**Daria Savvateeva**
- Tableau Public: [@daria.savvateeva](https://public.tableau.com/app/profile/daria.savvateeva)
- GitHub: [@DariaSavva](https://github.com/DariaSavva)

---

## 🙏 Acknowledgments

- Built using Tableau's Sample - EU Superstore dataset
- Cohort analysis methodology based on industry best practices
- Inspired by retention analysis frameworks from leading SaaS companies

---

**Built with Tableau | Last Updated: January 2025**
