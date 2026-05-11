# Campaign-Finance-Loans

Project Overview
This project transforms raw campaign finance data into a strategic decision-making tool. By leveraging SQL (BigQuery) for data engineering and Power BI for visualization, I identified critical funding risks, tracked debt velocity, and performed a compliance audit on over a decade of financial records.

----------------------------------------------------------------------------------------------

1. Concentration Risk (Dependency Analysis)

Business Question: Is the campaign’s funding diverse, or is it overly dependent on a few individuals?

The Insight: I identified a high concentration risk where 49.06% of total funding originates from just three lenders. Specifically, Jennifer Virden alone provides over 20% of the total capital.

Actionable Advice: The campaign needs to diversify its fundraising strategy to avoid a liquidity crisis if a top-tier donor stops contributing.

Technical Achievement: Used Nested Window Functions (SUM OVER) to calculate individual contributions against the grand total in a single pass.

SQL:

SELECT 
  Lender,
  SUM(Loan_Amount) as lender_total,
  SUM(SUM(Loan_Amount)) OVER() as grand_total,
  ROUND((SUM(Loan_Amount)/ SUM(SUM(Loan_Amount)) OVER()) * 100,2) as percentage_of_total
FROM `fluted-box-488519-p3.sql_practice.Cleaned_Campaign_Finance_Loans`
Group By Lender
ORDER BY lender_total DESC
LIMIT 3;

----------------------------------------------------------------------------------------------

2. Cash Flow Velocity

Business Question: How does our debt accumulate relative to the election cycle?

The Insight: Borrowing is not linear. Debt "explodes" in the final months of an election year, jumping from manageable levels to millions of dollars in a matter of weeks.

Actionable Advice: To avoid high-interest emergency loans, the campaign should establish a cash reserve in "off-years" to buffer against predictable October spikes.

Technical Achievement: Implemented a Running Total using window functions with an ORDER BY clause to visualize the "staircase" effect of debt accumulation.

SQL:

SELECT
  Loan_Date,
  -- 1. Get the total for just this day
  SUM(Loan_Amount) as daily_total,
  -- 2. Add today's total to the sum of all previous days
  SUM(SUM(Loan_Amount)) OVER(ORDER BY Loan_Date) as running_total_debt
FROM `fluted-box-488519-p3.sql_practice.Cleaned_Campaign_Finance_Loans`
GROUP BY Loan_Date
ORDER BY Loan_Date;

----------------------------------------------------------------------------------------------

3. Compliance Data Health Audit

Business Question: What percentage of our records are "audit-ready" versus "at-risk"?

The Insight: My audit flagged 196 records missing legally required occupation data. High counts of "Not Provided" are red flags for regulatory inquiries.

Actionable Advice: Generated a targeted "Cleanup List" for the compliance team to contact specific lenders and update records before the next filing deadline.

Technical Achievement: Utilized COUNTIF with conditional string matching to isolate data gaps.

SQL:

SELECT
  COUNTIF(Lender_Reported_Occupation <> 'Not Provided') as count_of_occupations,
  COUNTIF(Lender_Reported_Occupation LIKE 'Not Provided') as count_of_no_occupation
FROM `fluted-box-488519-p3.sql_practice.Cleaned_Campaign_Finance_Loans`

----------------------------------------------------------------------------------------------

4. Operational Accuracy (The Correction Ratio)

Business Question: Are our data entry and filing processes improving over time?

The Insight: Analyzing the "Correction" field revealed that the error rate peaked between 2014-2016. Recent years show a significant decrease in "Modify" and "X" (cancellation) flags.

Actionable Advice: This trend confirms that recent staff training and data-entry protocols are working, reducing the risk of costly administrative audits.

Technical Achievement: Applied SAFE_DIVIDE and EXTRACT(YEAR) to generate a Year-over-Year (YoY) error rate percentage.

SQL:

SELECT
  COUNTIF(Correction LIKE 'New') as New_flag,
  COUNTIF(Correction LIKE 'Modify') as Modify_flag,
  COUNTIF(Correction LIKE 'X') as X_flag,
  EXTRACT(YEAR FROM Loan_Date) as loan_year,
  -- Calculate the % of records that needed correcting
  ROUND(
    SAFE_DIVIDE(COUNTIF(Correction = 'Modify'), COUNT(*)) * 100, 2
  ) AS error_rate_percentage
FROM `fluted-box-488519-p3.sql_practice.Cleaned_Campaign_Finance_Loans`
GROUP BY EXTRACT(YEAR FROM Loan_Date)
ORDER BY EXTRACT(YEAR FROM Loan_Date);

