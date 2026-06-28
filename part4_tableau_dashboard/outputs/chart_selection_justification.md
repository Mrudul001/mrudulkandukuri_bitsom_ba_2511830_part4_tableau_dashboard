# Chart Selection Justification

For each major view in the executive dashboard, this document explains the business question it answers, why the chart type was chosen, the field encodings used, the design principle applied, and the mistake avoided.

---

## 1. Sales Trend View — Line Chart
- **Question answered:** How are sales (and profit) changing over time, and is there seasonality?
- **Why this chart type:** Line charts are the standard for showing a continuous measure over a continuous time axis; the eye naturally reads trend, direction, and repeating patterns (seasonality) from a line far better than from bars at this granularity (24 months).
- **Encoding:** X = `order_date` (Month/Year), Y = `SUM(sales)`, dual-axis or color-split line for `SUM(profit)`. Color reserved for the sales-vs-profit distinction only.
- **Design principle applied:** Consistent, minimal color — one color per measure, used nowhere else on the dashboard for an unrelated purpose.
- **Mistake avoided:** Did not truncate the Y-axis at a non-zero baseline, which would visually exaggerate month-to-month swings that are actually modest in percentage terms.

## 2. Regional Performance View — Map + Bar Chart
- **Question answered:** Which regions/states perform best on sales and profit margin?
- **Why this chart type:** A filled/symbol map gives an immediate geographic read for a leadership audience (states map directly to sales territories), paired with a bar chart for an exact ranked comparison the map alone can't give precisely.
- **Encoding:** Map: `state`/`region` for geography, color = `Profit Margin`, size or label = `SUM(sales)`. Bar chart: region on the axis, sorted descending by `SUM(sales)`, color = `Profit Margin` for a quick consistency check between volume and profitability.
- **Design principle applied:** Sorting — bars sorted by the primary measure rather than alphabetically, so the ranking is immediately legible.
- **Mistake avoided:** Did not use a single diverging color scale across both sales and margin on the same map (which confuses two different measures into one color); each measure has its own clearly labeled legend.

## 3. Category Profitability View — Bar Chart (Category → Sub-Category drill)
- **Question answered:** Which categories and sub-categories drive profit, and which are dragging it down?
- **Why this chart type:** A horizontal bar chart, sorted by profit, makes it immediate which sub-categories are net contributors versus laggards — more precise for ranking than a treemap, and more readable than a pie chart for 13 sub-categories.
- **Encoding:** Y = `sub_category` (sorted by `SUM(profit)` descending), X = `SUM(profit)`, color = `Profit Margin` (diverging scale, low margin highlighted), with `category` used as a filter/highlight action to drill from category to sub-category.
- **Design principle applied:** Clear hierarchy — category is the top-level filter, sub-category is the supporting detail, avoiding a single flat list of 13+3 items competing for attention.
- **Mistake avoided:** Avoided a pie/donut chart, which would make it hard to compare 13 sub-categories of similar-looking size and would obscure the negative/weak performers.

## 4. Customer Segment View — Highlight Table / Bar Chart
- **Question answered:** How do Consumer, Corporate, and Home Office segments compare on sales, margin, and return rate?
- **Why this chart type:** With only three segments and three metrics, a highlight table (or small grouped bar set) lets leadership scan all three KPIs per segment at once without needing to mentally cross-reference separate charts.
- **Encoding:** Rows = `customer_segment`, columns = `SUM(sales)`, `Profit Margin`, `Return Rate`; color intensity on each column independently to flag the segment that stands out (e.g., Home Office's higher return rate).
- **Design principle applied:** Focus on business interpretation — the table is built around the comparison leadership actually needs (which segment is the outlier), not just a generic listing of segments.
- **Mistake avoided:** Avoided cramming sales, margin, and return rate onto one shared color scale, which would make the return-rate outlier (a small percentage) invisible next to large sales values.

## 5. Shipping Performance View — Bar Chart
- **Question answered:** Which shipping modes have the longest delivery delay, and does delay relate to returns or satisfaction?
- **Why this chart type:** A simple bar chart comparing `ship_mode` against average delivery days is the clearest way to rank four discrete categories; a secondary reference line or paired bar adds return rate for direct comparison.
- **Encoding:** X = `ship_mode`, Y = `AVG(delivery_days)`, color/secondary bar = `Return Rate`, label = order count so leadership can see how much volume sits behind each mode.
- **Design principle applied:** Avoidance of misleading scales — both bars share a sensibly scaled, zero-based axis so the (large) delivery-day differences aren't confused with the (much smaller) return-rate differences.
- **Mistake avoided:** Did not imply a delay-causes-returns relationship through chart design alone (e.g., a trend line forcing a correlation) when the underlying data shows return rate is actually fairly flat across delay buckets — the chart presents the comparison neutrally and lets the data speak.

## 6. Discount vs. Profit View — Scatter Plot
- **Question answered:** How does discount level relate to order profit/margin?
- **Why this chart type:** A scatter plot is the correct choice for showing the relationship between two continuous numeric measures across thousands of individual orders — a chart type built specifically for this question, not used for decoration.
- **Encoding:** X = `discount`, Y = `profit` (or `Profit Margin`), color = `category` to see if the discount-profit relationship differs by category, size = `sales` optionally to show order scale.
- **Design principle applied:** Correct chart selection — matching chart type to the analytical question (relationship between two continuous variables) rather than defaulting to a bar chart.
- **Mistake avoided:** Avoided over-plotting confusion by using moderate point opacity/size and category-based color rather than 13 sub-category colors, which would be unreadable at this density.

## 7. Return Analysis View — Bar Chart / Highlight Table
- **Question answered:** Where are returns concentrated — by category, sub-category, segment, or region?
- **Why this chart type:** A sorted bar chart of return rate by sub-category immediately surfaces the small cluster of high-return items (Bookcases, Furnishings, Chairs) that a table of raw counts would bury.
- **Encoding:** X/Y = `sub_category` sorted by `Return Rate` descending, color = `category` to visually group Furniture's sub-categories together, label = `Return Rate` value.
- **Design principle applied:** Minimal clutter — one sorted measure, one grouping color, no unnecessary gridlines or decoration competing with the ranking itself.
- **Mistake avoided:** Avoided showing raw return *counts* without context (which would simply reflect order volume); used the calculated `Return Rate` so categories with more orders aren't unfairly flagged just for having more transactions.

---

## General Principles Applied Across the Dashboard
- **Consistent color language:** profit margin always uses the same green-to-red diverging scale wherever it appears; category colors are reused consistently across views.
- **Readable titles:** every sheet title states the business question, not just the field names (e.g., "Profit Margin by Region" rather than "Sheet 3").
- **Useful filters over decorative ones:** filters are limited to dimensions leadership actually uses to slice decisions (Region, Category, Segment, Date, Ship Mode) rather than every available field.
- **Zero-based, proportional axes throughout**, avoiding any chart that would visually exaggerate small differences.
