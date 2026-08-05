# Power-BI_Data-Analytics
# Sales & Profit Analytics Dashboard 📊


An interactive Power BI dashboard built during my Data Analytics internship (Week 3 task) on a retail sales dataset. The goal was to go through the full BI workflow end to end: import raw tables, build a proper data model, choose the right chart for each question, connect everything with filters, add KPI cards, polish the design, and finish with custom DAX measures.

I worked through this in six checkpoints, each one building on the last, so the sections below follow that same order.

## Overview 📖


The dataset is structured as a retail sales dataset with a central sales fact table and separate dimensions for customers, products, geography, employees, and dates — basically a mini version of what a real e-commerce or retail company's data warehouse would look like. Instead of just throwing every column onto a page, I focused on picking visuals that actually answer a specific business question (regional performance, category profitability, loyalty tier value, YoY growth), and linking them with slicers so the whole dashboard reacts together.

The final dashboard, along with every checkpoint's `.pbix` file, is in this repository.

## Checkpoint 1 — Data Import & Data Modeling 📑


Before touching any visuals, I imported the source tables into Power BI and built the relationships in Model view. I ended up with a 7-table star schema:

- **FactSales** — the central fact table (sales, profit, discount at transaction level)
- **FactSalesTarget** — sales targets, used later for the "vs. previous period / target" KPI comparisons
- **DimCustomer** — customer info, including LoyaltyTier (Bronze / Silver / Gold / Platinum)
- **DimProduct** — Category, SubCategory, Size, ProductKey
- **DimGeography** — Country, Region
- **DimEmployee** — sales rep / employee details
- **DimDate2** — a proper date table for time intelligence (Year, and YoY calculations later on)

I kept it as a star schema on purpose — one fact table surrounded by single-direction relationships to the dimension tables — instead of snowflaking things further, because it keeps the DAX simpler and the filtering behavior predictable once slicers are added in a later checkpoint.

## Checkpoints 2–5 — Building the Dashboard 🎨

**Chart selection (Checkpoint 2).** I deliberately picked five different visual types, each matched to what it's actually good at showing, rather than defaulting to bar charts everywhere:

- **Scatter (bubble) chart** — *Discount vs Profit by Category*: plots average discount against profit margin %, bubble size by sales, so I can see at a glance which categories stay profitable even with heavier discounting.
- **Treemap** — *Count of ProductKey by Category and SubCategory*: a hierarchy view is the natural fit here since it shows both the category breakdown and the relative size of each subcategory in one shape.
- **Line chart** — *Annual Profit Trend*: profit by year, because a trend over time belongs on a line, not a bar.
- **Combo chart (line + clustered column)** — *Total Sales and Profit Margin % by Region*: sales as columns, margin % as a line on a secondary axis, so volume and profitability per region are readable together instead of needing two separate charts.
- **Clustered column chart** — *Total Sales and Total Profit by LoyaltyTier*: side-by-side columns to directly compare sales vs. profit across the four loyalty tiers.

**Interactivity (Checkpoint 3).** Three slicers filter the whole page at once: **Country**, **Size**, and a **Year** range slider (2018–2025). Every visual on the page responds to all three, so I can, for example, isolate a single country's 2020–2022 performance and watch every chart update together.

**KPI cards (Checkpoint 4).** Four summary cards sit at the top: **Total Sales**, **Sales YoY %**, **Total Profit**, and **Profit YoY %** — giving an immediate read on both scale and growth direction before drilling into any chart.

**Design (Checkpoint 5).** I built a custom "Teal & Amber" theme and added a dashboard title banner instead of leaving the default white background and Power BI's default palette. The layout is arranged so the KPI row sits right under the title, the trend and hierarchy visuals take the top half, and the three comparison charts sit in a row underneath — the idea being: headline numbers first, trends and structure second, comparisons last.

<p align="center">
  <img src="./Screenshot- Checkpoint 5.png" alt="Final dashboard - Checkpoint 5" width="900">
</p>

<details>
<summary>See the dashboard at earlier checkpoints (click to expand)</summary>

Checkpoint 2 — chart types, before slicers and KPI cards were added:

![Checkpoint 2 screenshot](./Screenshot-%20checkpoint%202.png)

Checkpoint 3 — after adding the Country / Size / Year slicers:

![Checkpoint 3 screenshot](./Screenshot-%20checkpoint3.png)

</details>

## Checkpoint 6 — Calculated Measures 🧮

Two DAX measures I added to go beyond what's available out of the box:

**1. Regional Sales Contribution %**

```dax
Regional Sales Contribution % =
DIVIDE(
    [Total Sales],
    CALCULATE(
        [Total Sales],
        ALL(DimGeography[Region])
    ),
    0
)
```

This shows how much each region contributes to overall sales. It divides the selected region's sales by total sales across *all* regions — the `ALL(DimGeography[Region])` is what makes the denominator ignore the region filter, so every region is measured against the same company-wide total rather than against itself.

**2. Sales Performance**

```dax
Sales Performance =
SWITCH(
    TRUE(),
    [Sales YoY %] >= 0.10, "Strong Growth",
    [Sales YoY %] >= 0.05, "Moderate Growth",
    [Sales YoY %] >= 0,    "Stable",
    [Sales YoY %] >= -0.05, "Slight Decline",
    "Significant Decline"
)
```

This turns the raw YoY % into a plain-language label — Strong Growth, Moderate Growth, Stable, Slight Decline, or Significant Decline — using `SWITCH(TRUE())` to evaluate each threshold in order. It makes it much faster to scan performance across regions or categories without mentally converting percentages every time.

## Findings & Insights 💡


- **Sales are stable across regions, margin isn't.** Total sales per region sit in a tight band (14.0M–14.3M), but profit margin % ranges from 35.34% (East) to 35.73% (West) — so volume alone doesn't explain profitability; something region-specific (cost, discounting behavior, product mix) is driving the margin gap.
- **Discount level doesn't fully explain margin differences between categories.** In the scatter chart, average discount across categories barely moves (roughly 3.88%–3.92%), yet profit margin swings from about 32% up to 40%. That points to category-level cost structure mattering more than discounting strategy.
- **Bronze-tier customers drive the bulk of both sales and profit** — around 43M in sales and 15M in profit, far ahead of Silver, Gold, and Platinum combined. Platinum, despite presumably being the "top" tier, contributes the least in absolute terms, most likely reflecting a much smaller customer count rather than lower value per customer.
- **Profit isn't growing in a straight line.** The annual trend has a clear dip mid-period before recovering — flat totals year over year would have hidden that dip completely, so having the trend as its own visual card, separate from the KPI cards, was worth it.
- **Kitchen and Clothing are the largest categories by product count**, based on the treemap sizing, with Furniture and Electronics noticeably smaller — useful context for reading the category-level charts elsewhere on the dashboard.

## Tools Used 🛠️


- **Power BI Desktop** — data modeling, DAX, and all visuals
- **DAX** — YoY calculations, contribution %, and performance classification measures
- **Star schema modeling** — FactSales + FactSalesTarget + 5 dimension tables

## Repository Structure


| File | Description |
|---|---|
| `Checkpoint1.pbix` | Data import + star schema model |
| `Checkpoint-2.pbix` | Chart type selection |
| `Checkpoint-3.pbix` | Slicers added (Country, Size, Year) |
| `Checkpoint-4.pbix` | KPI cards added |
| `Checkpoint-5.pbix` | Final design + theme |
| `Checkpoint-6.pbix` | Calculated DAX measures |
| `Screenshot- checkpoint 2.png` / `Screenshot- checkpoint3.png` / `Screenshot- Checkpoint 5.png` | Progress screenshots |
