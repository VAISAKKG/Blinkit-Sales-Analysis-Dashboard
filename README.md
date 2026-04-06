# 🛒 Blinkit Sales Analysis Dashboard — Power BI
     Using Power BI Desktop, SQL Server &amp; DAX

An end-to-end **sales analysis dashboard** for Blinkit — India's last-minute grocery delivery app and Zomato subsidiary. The project covers the full analytics pipeline: **SQL-based data cleaning**, **Power Query transformation**, **DAX KPI measures**, and an interactive **Power BI dashboard** with slicers, charts, and a matrix visual.

---

## 📸 Dashboard Preview

> *Dashboard displaying Blinkit's sales performance across outlets, item types, fat content, and location tiers.*

**Top-line KPIs:**

| Metric | Value |
|---|---|
| 💰 Total Sales | $1.20M |
| 📊 Average Sales | $140.99 |
| 📦 No. of Items | 9K |
| ⭐ Average Rating | 3.92 |

---

## 🗂️ Project Structure

```
blinkit-powerbi-dashboard/
│
├── Blinkit.pbix                    # Power BI Desktop project file
├── BlinkIT_Grocery_Data.xlsx       # Source dataset (~8,500 rows)
├── SQuery_Doc.docx                 # Full SQL query documentation
├── README.md                       # This file
└── assets/
    └── dashboard_image.png         # Dashboard screenshot
```

---

## 📋 Dataset Description

The source dataset is an Excel file (`BlinkIT_Grocery_Data.xlsx`) containing approximately **8,500 rows** of Blinkit transaction and outlet data.

| Column | Description |
|---|---|
| Item Type | Product category (Fruits, Snacks, Dairy, etc.) |
| Item Fat Content | Low Fat / Regular |
| Item Visibility | Shelf/display visibility score |
| Item Weight | Product weight |
| Outlet Size | Small / Medium / High |
| Outlet Location Type | Tier 1 / Tier 2 / Tier 3 |
| Outlet Type | Supermarket Type 1/2/3, Grocery Store |
| Outlet Establishment Year | Year the outlet was set up |
| Sales | Revenue per transaction |
| Rating | Customer satisfaction rating |

---

## 🧹 Data Cleaning (SQL)

Data cleaning was performed in **SQL Server** before loading into Power BI. The primary task was standardising the `Item_Fat_Content` field, which contained inconsistent entries (`LF`, `low fat`, `reg`).

### View Raw Data

```sql
SELECT * FROM blinkit_data;
```

### Standardise Fat Content Categories

```sql
UPDATE blinkit_data
SET Item_Fat_Content =
  CASE
    WHEN Item_Fat_Content IN ('LF', 'low fat') THEN 'Low Fat'
    WHEN Item_Fat_Content = 'reg'              THEN 'Regular'
    ELSE Item_Fat_Content
  END;
```

### Verify the Cleaning

```sql
SELECT DISTINCT Item_Fat_Content FROM blinkit_data;
```

> Additional steps: handling missing values, removing duplicates, and formatting data types were handled in Power Query Editor after import.

---

## 📐 SQL Queries & KPIs

### A. Key Performance Indicators

**Total Sales (in Millions)**
```sql
SELECT CAST(SUM(Total_Sales) / 1000000.0 AS DECIMAL(10,2)) AS Total_Sales_Million
FROM blinkit_data;
```

**Average Sales**
```sql
SELECT CAST(AVG(Total_Sales) AS INT) AS Avg_Sales
FROM blinkit_data;
```

**Number of Items**
```sql
SELECT COUNT(*) AS No_of_Orders
FROM blinkit_data;
```

**Average Rating**
```sql
SELECT CAST(AVG(Rating) AS DECIMAL(10,1)) AS Avg_Rating
FROM blinkit_data;
```

---

### B. Total Sales by Fat Content

```sql
SELECT Item_Fat_Content,
       CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales
FROM blinkit_data
GROUP BY Item_Fat_Content;
```

---

### C. Total Sales by Item Type

```sql
SELECT Item_Type,
       CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales
FROM blinkit_data
GROUP BY Item_Type
ORDER BY Total_Sales DESC;
```

---

### D. Fat Content by Outlet Location (PIVOT)

```sql
SELECT Outlet_Location_Type,
       ISNULL([Low Fat], 0) AS Low_Fat,
       ISNULL([Regular], 0) AS Regular
FROM (
    SELECT Outlet_Location_Type, Item_Fat_Content,
           CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales
    FROM blinkit_data
    GROUP BY Outlet_Location_Type, Item_Fat_Content
) AS SourceTable
PIVOT (
    SUM(Total_Sales)
    FOR Item_Fat_Content IN ([Low Fat], [Regular])
) AS PivotTable
ORDER BY Outlet_Location_Type;
```

> **Why PIVOT?** Transforms `Item_Fat_Content` rows into columns per outlet tier. `ISNULL(..., 0)` replaces missing combinations with zero rather than NULL.

---

### E. Sales by Outlet Establishment Year

```sql
SELECT Outlet_Establishment_Year,
       CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales
FROM blinkit_data
GROUP BY Outlet_Establishment_Year
ORDER BY Outlet_Establishment_Year;
```

---

### F. Sales Percentage by Outlet Size

```sql
SELECT Outlet_Size,
       CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales,
       CAST((SUM(Total_Sales) * 100.0 / SUM(SUM(Total_Sales)) OVER()) AS DECIMAL(10,2)) AS Sales_Percentage
FROM blinkit_data
GROUP BY Outlet_Size
ORDER BY Total_Sales DESC;
```

> Uses a **window function** `SUM(...) OVER()` to compute the grand total for percentage calculation without a subquery.

---

### G. Sales by Outlet Location

```sql
SELECT Outlet_Location_Type,
       CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales
FROM blinkit_data
GROUP BY Outlet_Location_Type
ORDER BY Total_Sales DESC;
```

---

### H. All Metrics by Outlet Type

```sql
SELECT Outlet_Type,
       CAST(SUM(Total_Sales)     AS DECIMAL(10,2)) AS Total_Sales,
       CAST(AVG(Total_Sales)     AS DECIMAL(10,0)) AS Avg_Sales,
       COUNT(*)                                     AS No_Of_Items,
       CAST(AVG(Rating)          AS DECIMAL(10,2)) AS Avg_Rating,
       CAST(AVG(Item_Visibility) AS DECIMAL(10,2)) AS Item_Visibility
FROM blinkit_data
GROUP BY Outlet_Type
ORDER BY Total_Sales DESC;
```

---

## ⚙️ Power Query Transformation

After SQL cleaning, data was loaded into Power BI via Power Query Editor:

- Confirmed and corrected data types for numeric columns
- Applied additional category standardisation (`LF → Low Fat`, `reg → Regular`) as a fallback
- Renamed columns for readability
- Verified no nulls remained in key fields
- Clicked **Close & Apply**

---

## 📊 DAX Measures

All KPI cards are driven by DAX measures defined in the `Metrics` table:

```dax
Total Sales   = SUM('BlinkIT Grocery Data'[Sales])
Avg Sales     = AVERAGE('BlinkIT Grocery Data'[Sales])
No of Items   = COUNTROWS('BlinkIT Grocery Data')
Avg Rating    = AVERAGE('BlinkIT Grocery Data'[Rating])
```

---

## 🖥️ Dashboard Design

The dashboard (`Blinkit.pbix`) consists of three pages:

| Page | Content |
|---|---|
| Introduction | Project overview and context |
| Overview | High-level KPI summary |
| Dashboard | Full interactive analysis canvas |

### Visuals Used

| Visual | Purpose |
|---|---|
| KPI Cards | Total Sales, Avg Sales, No of Items, Avg Rating |
| Donut Chart | Fat Content distribution (Low Fat vs Regular) |
| Bar Chart | Sales by Item Type (horizontal) |
| Stacked Bar | Fat content breakdown by outlet tier |
| Line Chart | Sales trend by Outlet Establishment Year |
| Donut Chart | Sales share by Outlet Size |
| Bar Chart | Sales by Outlet Location (Tier 1/2/3) |
| Matrix Table | All metrics by Outlet Type |
| Slicers | Outlet Location, Outlet Size, Item Type |

### Slicers (Filters)

Three slicers allow full interactivity:
- **Outlet Location** — filter by Tier 1, Tier 2, Tier 3
- **Outlet Size** — filter by Small, Medium, High
- **Item Type** — filter by product category

---

## 📈 Key Insights

- **Medium-sized outlets** generate the highest total sales volume.
- **Tier 3 locations** outperform Tier 1 and Tier 2, indicating strong expansion potential in smaller cities.
- **Fruits & Vegetables** and **Snack Foods** are the top revenue-generating item types (~$0.18M each).
- **Low Fat products** account for the majority of sales ($776.32K vs $425.36K for Regular), reflecting health-conscious consumer trends.
- The **2018 establishment cohort** shows the highest sales ($205K), pointing to maturity effects in outlet performance.
- **Supermarket Type 1** dominates outlet-type sales at $787.55K with 5,577 items.
- Average ratings are consistent across outlet types (3.91–3.93), showing uniform service quality.

---

## 🎯 Strategic Recommendations

1. **Expand medium-sized outlets in Tier 3 cities** — this combination drives the highest returns.
2. **Prioritise stock of Low Fat Fruits, Vegetables, and Snacks** — top revenue items aligned with demand trends.
3. **Target rating improvements** — average of 3.92 leaves room for improvement; focus on item visibility and delivery quality.
4. **Study the 2018 outlet cohort** — understand what operational practices drove peak performance.

---

## 🔮 Future Enhancements

- [ ] Integrate real-time data via Blinkit's API
- [ ] Add ML-based demand forecasting visuals
- [ ] Customer segmentation and RFM analysis
- [ ] Advanced DAX for predictive KPIs (rolling averages, forecasts)
- [ ] Publish to Power BI Service with scheduled refresh
- [ ] Mobile layout optimisation

---

## 🛠️ Tools & Technologies

| Tool | Usage |
|---|---|
| Microsoft Power BI Desktop | Dashboard design, DAX, data modelling |
| Microsoft Excel (.xlsx) | Source dataset |
| SQL Server | Data cleaning and exploratory queries |
| Power Query (M) | Final transformation and loading |
| DAX | KPI measures and calculated columns |

---

## 🙏 Acknowledgements

- Dataset sourced from publicly available Blinkit grocery sales data
- Dashboard designed by **Vaisak**

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

## Author
## Vaisak Gopalakrishnan

*Built with 💛 using Power BI — Blinkit's yellow and green aesthetic.*

