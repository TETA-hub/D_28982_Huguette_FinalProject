# Business Intelligence (BI) Requirements  
Customer Loyalty Reward System

## 1. Introduction
The Business Intelligence (BI) component provides analytical insights to support strategic, operational, and compliance-related decision-making in the Customer Loyalty Reward System.

## 2. BI Objectives
- Monitor customer loyalty and engagement
- Analyze sales and purchasing behavior
- Support data-driven business decisions
- Ensure compliance with business rules
- Monitor system usage and performance

## 3. Stakeholders
| Stakeholder | BI Needs |
|------------|----------|
| Business Owner | Revenue trends, customer value |
| Marketing Manager | Loyalty performance, top customers |
| Operations Manager | Transaction activity |
| Auditor / Admin | Denied operations, audit logs |
| IT Administrator | System performance metrics |

## 4. Key Performance Indicators (KPIs)

### Customer & Loyalty KPIs
- Active customers
- Total loyalty points earned
- Total loyalty points redeemed
- Top loyal customers

### Sales KPIs
- Total sales revenue
- Average purchase value
- Monthly sales trends

### Compliance & System KPIs
- Number of denied operations
- Operations by type (INSERT, UPDATE, DELETE)
- Audit log entries per table

## 5. Decision Support
- Identify high-value and loyal customers
- Evaluate loyalty program effectiveness
- Monitor transaction trends
- Detect unauthorized or restricted operations
- Support compliance and auditing

## 6. Reporting Frequency
| Report | Frequency |
|------|-----------|
| Executive Reports | Monthly |
| Sales & Loyalty Reports | Weekly |
| Audit Reports | Daily |
| Performance Reports | Weekly / On-demand |

## 7. Dashboard Mockups

### Executive Summary Dashboard
- KPI cards: sales, active customers, loyalty points
- Monthly sales trend chart
- Top customers bar chart  
**Purpose:** Strategic decision-making

### Audit & Compliance Dashboard
- KPI cards: denied operations, violations
- Operation type distribution
- Recent audit logs  
**Purpose:** Compliance monitoring

### Performance Dashboard
- Daily transaction volumes
- Most accessed tables
- Peak usage periods  
**Purpose:** System performance monitoring

## 8. Sample BI Queries (Oracle SQL)

### Top Loyal Customers
```sql
SELECT customer_id,
       SUM(points_earned - points_redeemed) AS total_points
FROM rewards
GROUP BY customer_id
ORDER BY total_points DESC
FETCH FIRST 10 ROWS ONLY;


Sample BI Queries

Monthly Sales Trend
SELECT TO_CHAR(purchase_date,'YYYY-MM') AS month,
       SUM(total_amount) AS total_sales
FROM purchases
GROUP BY TO_CHAR(purchase_date,'YYYY-MM')
ORDER BY month;

Audit Summary
SELECT table_name, operation_type, COUNT(*) AS operation_count
FROM transactions_log
GROUP BY table_name, operation_type;

9. Conclusion

The BI component enhances the system by transforming transactional data into actionable insights, supporting informed decision-making, performance monitoring, and compliance enforcement
