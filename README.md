# E-commerce Sales & Operations Dashboard

A Power BI report built on a public Brazilian marketplace dataset with about 100k orders placed between 2016 and 2018. The data is anonymized; the default view covers Jan/2017 to Aug/2018.

## Files

- **Ecommerce.pbix**: Power BI report with the semantic model
- **Ecommerce.pdf**: export of the four report pages

## Report pages

1. **Executive Overview**: GMV, orders, average ticket, customers, review score and on-time delivery. Monthly GMV trend, 2018 vs 2017 by month, GMV by category and by customer state, and a year-over-year comparison.
2. **Logistics**: delivered and late orders, actual vs promised delivery time, on-time rate by month and by state, and average review score by delivery status.
3. **Customers & Satisfaction**: unique and repeat customers, review distribution, monthly score trend, payment method mix, and average ticket by region.
4. **Products & Sellers**: performance tables by category and by seller (GMV, orders, ticket, freight share, score, on-time rate).

## Semantic model

- Star schema with an order header table linked to items, payments and reviews, plus date, customer, product and seller dimensions. All relationships are single-direction.
- 32 documented DAX measures in display folders: sales, customers, logistics, satisfaction, payments, and time intelligence (prior year and YoY variance).
- Business rules: GMV = product revenue + freight. Canceled and unavailable orders are excluded from sales.

## Interactivity

- Four slicers (year, customer state, seller state, category) are synced across all pages.
- Each slicer lists only the values that have data under the other filters. This is driven by a DAX measure used as a visual filter.
- Custom dark theme with a palette checked for contrast and color blindness.

## Validation

All measures were checked against an independent Python (pandas) calculation: 176 automated DAX tests, all passing.
