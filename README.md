# Fashion Retail Analytics & Customer Segmentation
An end-to-end Business Intelligence project using Power BI, Advanced DAX, and Power Query.

Author: Sadia A. Ruhi

### Disclaimer
This project simulates a real-world analytics engagement for a fictional global fashion retail brand. All financial figures, customer counts, and business metrics shown in this project are derived entirely from a public, anonymized transactional dataset and do not reflect any real company's actual performance. The custom visual branding used in the dashboard is for design and storytelling purposes only.

### Project Overview
This project simulates a full-cycle analytics engagement for a retail brand. The goal was to take raw, messy transactional data all the way through to a decision-ready business intelligence dashboard, using the same workflow a Data Analyst or Business Analyst would follow inside a real company: define the business problem first, clean and validate the data, model it for performance, build the analytical logic, and deliver the insight through a premium, interactive Power BI report.

### Business Problem
The business has no structured view of customer value, and retention efforts are reactive instead of targeted. Furthermore, leadership lacks visibility into how aggressive discounting is impacting overall Gross Profit.

### Business Questions Answered
- Who are our most valuable customers, and how much revenue do they actually contribute?
- Which customers have gone quiet and are at risk of churning?
- How does customer value and retention differ across demographics (Age/Gender)?
- Which product categories and SKUs are driving high volume but destroying margins?
- What specific, concrete actions should the business take for each customer segment?

### Key Findings
- The 80/20 Rule: "Champions" represent roughly 27% of the customer base but generate the bulk of the revenue.
- Churn Risk: 29% of the customer base is classified as "Churned" (365-day inactivity threshold), indicating a significant retention opportunity.
- Demographic Insights: The 18-24 age demographic is the highest revenue driver, but aggressive discounting is stagnating their Gross Margin.
- Product Profitability: Apparel and Bags are the primary profit engines, whereas accessories and shoes have significantly lower margins.

### Data Source
A public, anonymized transactional dataset containing consumer demographics and purchase histories. The data required significant cleaning to handle orphaned records, currency formatting, and date hierarchy mapping.

### Tools and Tech Stack
- Power Query (M):	Data import, cleaning, deduplication, and resolving data type anomalies (e.g., currency symbols, orphaned IDs).
- Power BI Desktop:	Building a premium, 4-page interactive dashboard with a custom dark-mode UI.
- Advanced DAX:	Creating dynamic measures for RFM analysis, time-intelligence (YoY), and What-If parameters.
- Data Modeling:	Designing a robust Star Schema (Fact and Dimension tables) to ensure high performance and accurate filtering.

### Core Concepts Demonstrated
- RFM Analysis: Scoring customers on Recency, Frequency, and Monetary value using RANKX and SUMMARIZE to quantify customer worth.
- Customer Segmentation: Translating raw RFM scores into business-meaningful groups (Champions, Loyal, At Risk, New, Needs Attention, Lost).
- What-If Parameter: A dynamic discount simulator allowing users to forecast profit changes.
- Interactive BI Design: Slicers, bookmark-based toggles (Revenue vs. Profit view), custom tooltips, and a hidden drill-through page.

### Project Workflow
- Step 1: Business Requirements Document: Defined scope, business questions, key definitions, and success criteria before any data work began.
- Step 2: Data Cleaning (Power Query): Resolved data type mismatches, removed orphaned consumer records, and built a dynamic DateTable for time intelligence.
- Step 3: Data Modeling: Built a Star Schema connecting Fact (Purchases info) and Dimension (Consumers info) tables to ensure 1-to-Many relationships.
- Step 4: DAX & Analytics: Wrote complex DAX to dynamically score and segment 3,000+ customers into the RFM Table calculated table.
- Step 5: UI/UX Dashboard Build: Designed a premium 4-page report (Executive Overview, Customer Insights, Product & Geography, and an Explorer page).

### Dashboard Structure
- Page 1: Executive Overview: Total Revenue, Gross Profit, Margin %, and AOV as headline KPIs, along with a monthly trend line and a dynamic Discount Simulator.
- Page 2: Customer Insights: RFM Segmentation donut chart, demographic breakdowns (Age/Gender), and Churn Rate analysis. Includes a Bookmark toggle to swap the entire page between Revenue and Profit views.
- Page 3: Product & Geography: Global revenue map, Treemap for SKU profitability, Waterfall Chart showing profit contribution by category, and a Scatter Plot analyzing the impact of Discount % on Gross Margin.
- Hidden Page: Explorer: A row-level detail page. Users can right-click any data point and drill through to see raw transaction data.

### Sample DAX: The RFM Segmentation Table
```dax RFM Table = 
VAR SnapshotDate = CALCULATE(MAX('Purchases info'[Period]), ALL('Purchases info'))
VAR CleanTx = FILTER('Purchases info', NOT ISBLANK('Purchases info'[Consumer ID]))
VAR CustomerSummary =
    GROUPBY(CleanTx, 'Purchases info'[Consumer ID],
        "LastOrder", MAXX(CURRENTGROUP(), 'Purchases info'[Period]),
        "Monetary", SUMX(CURRENTGROUP(), 'Purchases info'[Line Value (net discount)]))
VAR RFMBase =
    ADDCOLUMNS(CustomerSummary,
        "Recency", DATEDIFF([LastOrder], SnapshotDate, DAY),
        "Frequency", VAR CustID = 'Purchases info'[Consumer ID]
            RETURN CALCULATE(DISTINCTCOUNT('Purchases info'[Order ID]), 'Purchases info'[Consumer ID] = CustID))
VAR TotalCustomers = COUNTROWS(RFMBase)
VAR RFMScored =
    ADDCOLUMNS(RFMBase,
        "R_Score", VAR AscRank = RANKX(RFMBase, [Recency] * 1e9 + 'Purchases info'[Consumer ID], , ASC, SKIP) RETURN 6 - ROUNDUP((AscRank / TotalCustomers) * 5, 0),
        "F_Score", VAR AscRank = RANKX(RFMBase, [Frequency] * 1e9 + 'Purchases info'[Consumer ID], , ASC, SKIP) RETURN ROUNDUP((AscRank / TotalCustomers) * 5, 0),
        "M_Score", VAR AscRank = RANKX(RFMBase, [Monetary] * 1e9 + 'Purchases info'[Consumer ID], , ASC, SKIP) RETURN ROUNDUP((AscRank / TotalCustomers) * 5, 0))
RETURN
ADDCOLUMNS(RFMScored, "Segment",
        SWITCH(TRUE(),
            [R_Score] >= 4 && [F_Score] >= 4 && [M_Score] >= 4, "Champions",
            [R_Score] >= 3 && [F_Score] >= 3, "Loyal",
            [R_Score] <= 2 && [F_Score] >= 3, "At Risk",
            [R_Score] >= 4 && [F_Score] <= 2, "New",
            [R_Score] = 3  && [F_Score] <= 2, "Needs Attention",
            "Lost"),
    "IsChurned", IF([Recency] > 365, 1, 0))
```
