# Wide World Importers - Sales & Profit Analytics Dashboard

## 1. Project Overview

Wide World Importers (WWI) is a wholesale novelty goods importer and distributor based in the San Francisco Bay area, selling to specialty stores, supermarkets, computing stores, tourist shops, individuals, and other wholesalers across the United States.

This project is a three-page Power BI report that gives WWI management a structured view of sales, profit, quantity, and stock performance, broken down by sales territory, buying group, state, and fiscal period, with drill-through to product, employee, and customer detail.

### Technical Stack

- **BI Tool:** Power BI Desktop
- **Data Modeling:** Star schema (one fact table, five dimension tables)
- **Logic Framework:** DAX (time intelligence, field parameters, rule-based conditional formatting, dynamic Top-N)
- **Interactivity:** Fiscal Year slicers, field parameter switching, drill-through, Smart Narrative summary

---

## 2. Data Architecture

### Data Model Structure

- **Fact Table:** `FactSale` – invoice-level sales with quantity, profit, and totals including / excluding tax.
- **Dimension Tables:**
  * `DimCity`: Sales Territory and State Province.
  * `DimCustomer`: Customer and Buying Group.
  * `DimDate`: Calendar table with a fiscal year starting in **November**.
  * `DimEmployee`: Employee details used for profit attribution.
  * `DimStockItem`: Stock item catalog and unit price.
- **Supporting Tables:**
  * Field Parameter – switches visuals between **Sales Territory** and **Buying Group**.
  * `Yearly Avg Sales` – reference table for the heatmap's yearly average.

> **Data quality note:** ~38% of sales are linked to *Unknown* customers. A caution icon on the Overview page flags this.

<!-- Add a data model screenshot: ![Data Model](images/data-model.png) -->

Source-data cleaning steps are documented in [`docs/DataTransformation.md`](docs/DataTransformation.md).

---

## 3. Page 1: Overview

### Objective
A high-level summary of Sales, Profit, Quantity, and Stock performance, with a Smart Narrative button that summarizes the page.

<!-- ![Overview](images/overview.png) -->

### Visual Components

1. **KPI Cards:** Number of Sales, Total Sales Incl Tax, Total Profit, Total Quantity, and Total Stock Price, each with its Last Year value.
2. **Combo Chart:** Total Sales Excl Tax (columns) and Gross Margin % (line), switchable between Sales Territory and Buying Group via the field parameter.
3. **Donut Chart:** Total Profit distribution, also driven by the field parameter.
4. **Matrix:** Monthly sales by fiscal year vs. the same period last year, with YoY %.

### Page Measures

```dax
Gross Margin %  = DIVIDE([Total Profit], [Total Sales Excl Tax])
Total Stock Price = SUMX(FactSale, FactSale[Quantity] * RELATED(DimStockItem[Unit Price]))
Base Measure LY = CALCULATE([Base Measure], SAMEPERIODLASTYEAR(DimDate[Date]))
```

The full KPI list is in the [Report Handbook](docs/ReportHandbook.md).

---

## 4. Page 2: Sales & Profit by Region

### Objective
Analyze regional performance and monthly trends, and act as the entry point for drill-through.

<!-- ![Sales & Profit](images/sales-profit.png) -->

### Visual Components

1. **Filled Map:** Sales (Incl Tax) and Profit by State Province, with tooltips.
2. **Scatter Chart:** Total Sales Incl Tax vs. Total Profit, bubble size by Total Quantity.
3. **Monthly Sales Heatmap (Matrix):** Territory and State by Fiscal Year and Month, colour-coded against the yearly average.
4. **Yearly Avg Sales (Table):** Reference table behind the heatmap.

### Page Measures

```dax
Yearly Avg Sales =
CALCULATE(
    AVERAGEX(VALUES(DimDate[Fiscal Month Number]), [Total Sales Incl Tax]),
    ALLEXCEPT(DimDate, DimDate[Fiscal Year])
)
```

```dax
CF Sales Color =
VAR Diff = [Diff Total vs Yearly Avg Sales Amount %]
RETURN
    SWITCH(
        TRUE(),
        ISBLANK([Total Sales Incl Tax]), "#FFFFFF",
        Diff <= -0.90, "#D64550",
        Diff <= -0.70, "#D76E32",
        Diff <= -0.60, "#D99017",
        Diff <= -0.40, "#D9AF03",
        Diff <= -0.20, "#99B743",
        Diff <=  0.20, "#73B96A",
        "#57BB88"
    )
```

---

## 5. Page 3: Territory Insights (Drill-through)

### Objective
Drill-through analysis for the selected Sales Territory, State Province, and Fiscal Year.

<!-- ![Territory Insights](images/territory-insights.png) -->

### Visual Components

| Visual | Measure | Dimension | Top N |
| --- | --- | --- | --- |
| Top Products by Sales | Total Sales Incl Tax | Stock Item | 5 |
| Top Products by Stock Price | Total Stock Price | Stock Item | 5 |
| Top Employees by Profit | Total Profit | Employee | 3 |
| Top Buying Group | Total Sales Incl Tax | Buying Group | 1 |
| Top Customers in Top Buying Group | Total Sales Incl Tax | Customer | 5 |

---

## 6. Key Insights

- **2015** was the highest-grossing year: **$62M** Total Sales (Incl Tax) and **$27M** Total Profit from roughly **71K** sales transactions.
- The **Southeast** territory contributed the most Sales (Excl Tax) in 2015 at **22.07%**.
- **Great Lakes** had the highest Gross Margin at **50.57%**.
- Strongest YoY growth: **April 2015** at **23.88%**.
- **Texas** led the Southwest territory; **Florida** led the Southeast, while **Mississippi** and **Tennessee** lagged.

See [`docs/ReportInsights.md`](docs/ReportInsights.md) for the full write-up.

---

## 7. Visual Design and UX Patterns

- **Drill-through:** Page 2's map, scatter chart, and heatmap drill through to Territory Insights on Sales Territory, State Province, and Fiscal Year.
- **Field Parameter:** One set of visuals toggles between Sales Territory and Buying Group, keeping the report within three pages.
- **Conditional Formatting:** The heatmap uses a rule-based colour measure (`CF Sales Color`): white for no sales, a red-to-orange-to-yellow scale below the yearly average, and greens at or above it.

---

## 8. Repository Structure

```
├── Wide_World_Importers_Report.pbix   # Power BI report (3 pages)
├── README.md
├── docs/
│   ├── DataTransformation.md          # Source data transformation steps
│   ├── ReportHandbook.md              # Page-by-page build specification
│   └── ReportInsights.md              # Findings from the report
└── images/                            # Report screenshots used in this README
```

## 9. How to Use

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/).
2. Download `Wide_World_Importers_Report.pbix` and open it.
3. Use the Fiscal Year slicer and field parameter on the Overview, then right-click a territory or state on Page 2 and choose **Drill through → Territory Insights**.
