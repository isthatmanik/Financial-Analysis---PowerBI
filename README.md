# Financial Analysis using Power BI & DAX

This project focuses on analyzing credit card usage and financial metrics for a banking institution using **Power BI** and **DAX queries**. The goal is to provide key insights into customer behavior, credit utilization, and financial performance by applying advanced DAX functions.

## Project Overview

The analysis covers various aspects of financial data, including:

- Running totals of credit card transactions.
- 4-week moving averages of credit limits for each client.
- Month-over-month and week-over-week growth in transaction amounts.
- Customer Acquisition Cost (CAC) as a ratio of transaction amounts.
- Yearly average of client utilization ratios.
- Top 5 clients by total transaction amount.
- Credit risk scores based on multiple financial metrics.
- Churn indicators for inactive clients.

## Problem Statements and DAX Queries

1. **Running Total of Credit Card Transactions**

    **DAX Query:**
    ```DAX
    Running Total = CALCULATE([Total Transaction Amount]),
    FILTER(ALL('Credit Card'),'Credit Card'[Week_Start_Date] <= MAX('Credit Card'[Week_Start_Date])))
    ```

2. **4-Week Moving Average of the Credit Limit for Each Client**

    **DAX Query:**
    ```DAX
    moving_average_4_weeks =
    var weeks4 = DATESINPERIOD('calendar'[Date], MAX('calendar'[Date]), -28, DAY)
    var total_amount = CALCULATE([Total Transaction Amount],weeks4)
    var num_of_weeks = CALCULATE(DISTINCTCOUNT('calendar'[week_number]),weeks4)
    RETURN DIVIDE(total_amount, num_of_weeks, 0)
    ```

3. **Month-on-Month (MoM%) and Week-on-Week (WoW%) Growth on Transaction Amount**

    **DAX Query:**
    ```DAX
    mom%growth =
    var prev_month = CALCULATE([Total Transaction Amount],DATEADD('calendar'[Date], -1,MONTH))
    return DIVIDE([Total Transaction Amount]-prev_month,prev_month,0)

    wow%growth =
    var prev_week = CALCULATE([Total Transaction Amount], DATEADD('calendar'[Date],-7,DAY))
    return DIVIDE([Total Transaction Amount]-prev_week, prev_week,0)
    ```

4. **Customer Acquisition Cost (CAC) as a Ratio of Transaction Amount**

    **DAX Query:**
    ```DAX
    ratio_cac_transaction_amount = DIVIDE(
    SUM('Credit Card'[Customer_Acq_Cost]),
    [Total Transaction Amount],0)
    ```

5. **Yearly Average of Avg_Utilization_Ratio for All Clients**

    **DAX Query:**
    ```DAX
    avg_utilization_ratio =
    AVERAGE('Credit Card'[Avg_Utilization_Ratio])
    ```

6. **Percentage of Interest Earned Compared to Total Revolving Balance for Each Client**

    **DAX Query:**
    ```DAX
    interest_earned_by_revol_balance = DIVIDE(
    SUM('Credit Card'[Interest_Earned]),
    SUM('Credit Card'[Total_Revolving_Bal]),0)
    ```

7. **Top 5 Clients by Total Transaction Amount**

    **DAX Query:**
    ```DAX
    top_5_clients =
    TOPN(5,SUMMARIZE('Credit Card', 'Credit Card'[Client_Num],"total_amount", [Total Transaction Amount]),[total_amount],DESC)
    ```

8. **Clients Whose Avg_Utilization_Ratio Exceeds 80%**

    **DAX Query:**
    ```DAX
    client_Avg_Utilization_Ratio_exceeds_80% = 
    IF([Avg_Utilization_Ratio] > 0.8, TRUE, FALSE)
    ```

9. **Customer Churn Indicator (Clients with No Transactions in Last 6 Months)**

    **DAX Query:**
    ```DAX
    churn_status = 
    var last_6_months = CALCULATE([Total Transaction Amount], DATESINPERIOD('calendar'[Date],MAX('calendar'[Date]), -6, MONTH))
    RETURN(IF(ISBLANK(last_6_months), "Churned", "Not-Churned"))
    ```

10. **Delinquency Rate (Percentage of Clients with Delinquent Accounts)**

    **DAX Query:**
    ```DAX
    percentage_with_delinquent_acc =
    var greater_zero = CALCULATE(COUNTROWS('Credit Card'),'Credit Card'[Delinquent_Acc] > 0)
    var total_rows = COUNTROWS('Credit Card')
    RETURN DIVIDE(greater_zero, total_rows,0)
    ```

11. **Credit Risk Score (Based on Avg_Utilization_Ratio, Delinquent Accounts, and Total Revolving Balance)**

    **DAX Query:**

    **Step 1:** Normalize the revolving balance as it is in thousands in the data
    ```DAX
    normalized_revol_balance =
    Divide('Credit Card'[Total_Revolving_balance]-MIN('Credit Card'[Total_Revolving_balance]),
    MAX('Credit Card'[Total_Revolving_balance])-MIN('Credit Card'[Total_Revolving_balance]),0)
    ```

    **Step 2:** Weight the average utilization ratio, delinquent accounts, and normalized revolving balance.
    ```DAX
    credit_risk_score =
    0.5*[avg_utilization_ratio]+
    0.3*'Credit Card'[normalised_Revol_balance]+
    0.2*'Credit Card'[Delinquent_Acc]
    ```

13. **Income vs Credit Limit Correlation(Created using Quick Measure)**

    **DAX Query:**
    ```DAX
    Income and Credit_Limit correlation for Client_Num = 
      VAR __CORRELATION_TABLE = VALUES('Customers'[Client_Num])
      VAR __COUNT =
      	COUNTX(
      		KEEPFILTERS(__CORRELATION_TABLE),
      		CALCULATE(SUM('Customers'[Income]) * SUM('Credit Card'[Credit_Limit]))
      	)
      VAR __SUM_X =
      	SUMX(
      		KEEPFILTERS(__CORRELATION_TABLE),
      		CALCULATE(SUM('Customers'[Income]))
      	)
      VAR __SUM_Y =
      	SUMX(
      		KEEPFILTERS(__CORRELATION_TABLE),
      		CALCULATE(SUM('Credit Card'[Credit_Limit]))
      	)
      VAR __SUM_XY =
      	SUMX(
      		KEEPFILTERS(__CORRELATION_TABLE),
      		CALCULATE(SUM('Customers'[Income]) * SUM('Credit Card'[Credit_Limit]) * 1.)
      	)
      VAR __SUM_X2 =
      	SUMX(
      		KEEPFILTERS(__CORRELATION_TABLE),
      		CALCULATE(SUM('Customers'[Income]) ^ 2)
      	)
      VAR __SUM_Y2 =
      	SUMX(
      		KEEPFILTERS(__CORRELATION_TABLE),
      		CALCULATE(SUM('Credit Card'[Credit_Limit]) ^ 2)
      	)
      RETURN
      	DIVIDE(
      		__COUNT * __SUM_XY - __SUM_X * __SUM_Y * 1.,
      		SQRT(
      			(__COUNT * __SUM_X2 - __SUM_X ^ 2)
      				* (__COUNT * __SUM_Y2 - __SUM_Y ^ 2)
      		)
      	)
    ```

14. **Average Customer Satisfaction Score by Credit Card Category**

    **DAX Query:**
    ```DAX
    avg_satisfaction_score =
    SUMMARIZE('Credit Card','Credit Card'[Card_Category], "avg_satisfaction_score", AVERAGE(customers[Cust_Satisfaction_Score]))
    ```

15. **Loan Approval vs Credit Limit (Analyzing How Credit Limit Affects Loan Approval)**

    **DAX Query:**
    ```DAX
    loan_approval_yes = CALCULATE(AVERAGE('Credit Card'[Credit_Limit]), Customers[Personal_loan] = "Yes")

    loan_approval_no = CALCULATE(AVERAGE('Credit Card'[Credit_Limit]), Customers[Personal_loan] = "No")
    ```

16. **High Risk Clients Flag (Clients with Revolving Balance > 90% of Credit Limit and High Avg_Utilization_Ratio)**

    **DAX Query:**
    ```DAX
    flag_clients =
    IF('Credit Card'[normalized_revol_balance] > 0.9 && [avg_utilization_ratio] > 0.8, "Flagged", "Not Flagged")
    ```

## Tools Used

- **Power BI**: Data visualization and reporting.
- **DAX**: Data Analysis Expressions for financial metrics calculation.

## Contact

Feel free to reach out for any queries or collaboration:

- **Email**: manik.gupta0210@gmail.com
- **LinkedIn**: [linkedin.com/in/isthatmanik](http://linkedin.com/in/isthatmanik)
