This document provides a comprehensive overview of a Power BI project focusing on customer churn analysis. The project leverages a combination of SQL Server for ETL processes, Power BI for data transformation and visualization, and Python for machine learning-based churn prediction.

### Project Goal

The primary objective is to **develop a robust Power BI dashboard to understand and mitigate customer churn** within a telecom company (as an example). The insights gained will help:

*   **Analyze customer data across various dimensions:** demographic, geographic, payment and account information, and services used.
*   **Identify key churn drivers and inform targeted marketing campaigns.**
*   **Develop a predictive model to identify potential churners.**

### Key Metrics

The project focuses on tracking and visualizing the following metrics:

*   **Total Customers:** The total number of customers in the dataset.
*   **Total Churn & Churn Rate:** The number and percentage of customers who have churned.
*   **New Joiners:** The number of new customers who have joined.

### Technology Stack

*   **Microsoft SQL Server:** Utilized for ETL (Extract, Transform, Load) processes, ensuring data integrity and efficient handling of recurring data loads.
*   **SQL Server Management Studio (SSMS):** Provides a GUI for interacting with SQL Server and executing queries.
*   **Power BI Desktop:** Used for data transformation, creating insightful visualizations, and building an interactive dashboard.
*   **Python:** Leverages the scikit-learn library, specifically the Random Forest algorithm, for machine learning-based churn prediction.
*   **Jupyter Notebook:** An interactive environment for developing and running Python code for the machine learning model.

### Project Steps

#### STEP 1: ETL Process in SQL Server

**Objective:** Load and prepare the raw customer data for analysis.

**Actions Taken:**

1.  **Database Creation:** Create a dedicated database named "db\_Churn" in SQL Server using SSMS. `CREATE DATABASE db_Churn`
2.  **Import Data into Staging Table:**
    *   Import the CSV data file into a staging table "stg\_Churn" using the SQL Server Import Wizard (Right click on db_churn >> Task >> Import >> Flat file >> Browse CSV file).
    *   Handle data type inconsistencies and potential import errors during this step (add customerId as primary key, allow nulls for all remaining columns, an dchange the datatype where it say Bit to Varchar(50)).
3.  **Data Exploration and Validation:**
    *   Write SQL queries to analyze the distribution of key variables (e.g., gender, contract type, customer status).
    
        ```sql
        -- Count of distinct values and their percentage distribution for Gender column 
        SELECT Gender, Count(Gender) as TotalCount,
        Count(Gender) * 1.0 / (Select Count(*) from stg_Churn)  as Percentage
        from stg_Churn
        Group by Gender
        
        -- Expected Output:
        --  Gender	TotalCount	Percentage
        --  Male	2370	0.369273917108
        --  Female	4048	0.630726082891

        -- Count of distinct values and their percentage distribution for Contract column
        SELECT Contract, Count(Contract) as TotalCount,
        Count(Contract) * 1.0 / (Select Count(*) from stg_Churn)  as Percentage
        from stg_Churn
        Group by Contract

        -- Expected Output:
        --  Contract	TotalCount	Percentage
        --  Month-to-Month	3286	0.511997507011
        --  One Year	1413	0.220162044250
        --  Two Year	1719	0.267840448737

        -- different customer statuses and their revenue contribution.
        SELECT Customer_Status, Count(Customer_Status) as TotalCount, Sum(Total_Revenue) as TotalRev,
        Sum(Total_Revenue) / (Select sum(Total_Revenue) from stg_Churn) * 100  as RevPercentage
        from stg_Churn
        Group by Customer_Status

        -- Expected Output:
        --  Customer_Status	TotalCount	TotalRev	RevPercentage
        --  Joined	411      49281.5598697662  	  0.253097281975677
        --  Churned	1732	3411960.5796299         17.5229426827105
        --  Stayed	4275	16010148.2622757    	  82.2239600353138

        -- States with the highest customer representation
        SELECT State, Count(State) as TotalCount,
        Count(State) * 1.0 / (Select Count(*) from stg_Churn)  as Percentage
        from stg_Churn
        Group by State
        Order by Percentage desc

        -- Expected Output:
        --  State	        TotalCount	    Percentage
        --  Uttar Pradesh	  629	        0.098005609224
        --  Tamil Nadu	      600	        0.093487067622
        --  ...
        ```
    *   Identify and quantify null values in each column to guide data cleaning efforts.
        ```sql
        -- Checking for null values in each column
        SELECT 
            SUM(CASE WHEN Customer_ID IS NULL THEN 1 ELSE 0 END) AS Customer_ID_Null_Count,
            SUM(CASE WHEN Gender IS NULL THEN 1 ELSE 0 END) AS Gender_Null_Count,
            SUM(CASE WHEN Age IS NULL THEN 1 ELSE 0 END) AS Age_Null_Count,
            SUM(CASE WHEN Married IS NULL THEN 1 ELSE 0 END) AS Married_Null_Count,
            SUM(CASE WHEN State IS NULL THEN 1 ELSE 0 END) AS State_Null_Count,
            SUM(CASE WHEN Number_of_Referrals IS NULL THEN 1 ELSE 0 END) AS Number_of_Referrals_Null_Count,
            SUM(CASE WHEN Tenure_in_Months IS NULL THEN 1 ELSE 0 END) AS Tenure_in_Months_Null_Count,
            SUM(CASE WHEN Value_Deal IS NULL THEN 1 ELSE 0 END) AS Value_Deal_Null_Count,
            SUM(CASE WHEN Phone_Service IS NULL THEN 1 ELSE 0 END) AS Phone_Service_Null_Count,
            SUM(CASE WHEN Multiple_Lines IS NULL THEN 1 ELSE 0 END) AS Multiple_Lines_Null_Count,
            SUM(CASE WHEN Internet_Service IS NULL THEN 1 ELSE 0 END) AS Internet_Service_Null_Count,
            SUM(CASE WHEN Internet_Type IS NULL THEN 1 ELSE 0 END) AS Internet_Type_Null_Count,
            SUM(CASE WHEN Online_Security IS NULL THEN 1 ELSE 0 END) AS Online_Security_Null_Count,
            SUM(CASE WHEN Online_Backup IS NULL THEN 1 ELSE 0 END) AS Online_Backup_Null_Count,
            SUM(CASE WHEN Device_Protection_Plan IS NULL THEN 1 ELSE 0 END) AS Device_Protection_Plan_Null_Count,
            SUM(CASE WHEN Premium_Support IS NULL THEN 1 ELSE 0 END) AS Premium_Support_Null_Count,
            SUM(CASE WHEN Streaming_TV IS NULL THEN 1 ELSE 0 END) AS Streaming_TV_Null_Count,
            SUM(CASE WHEN Streaming_Movies IS NULL THEN 1 ELSE 0 END) AS Streaming_Movies_Null_Count,
            SUM(CASE WHEN Streaming_Music IS NULL THEN 1 ELSE 0 END) AS Streaming_Music_Null_Count,
            SUM(CASE WHEN Unlimited_Data IS NULL THEN 1 ELSE 0 END) AS Unlimited_Data_Null_Count,
            SUM(CASE WHEN Contract IS NULL THEN 1 ELSE 0 END) AS Contract_Null_Count,
            SUM(CASE WHEN Paperless_Billing IS NULL THEN 1 ELSE 0 END) AS Paperless_Billing_Null_Count,
            SUM(CASE WHEN Payment_Method IS NULL THEN 1 ELSE 0 END) AS Payment_Method_Null_Count,
            SUM(CASE WHEN Monthly_Charge IS NULL THEN 1 ELSE 0 END) AS Monthly_Charge_Null_Count,
            SUM(CASE WHEN Total_Charges IS NULL THEN 1 ELSE 0 END) AS Total_Charges_Null_Count,
            SUM(CASE WHEN Total_Refunds IS NULL THEN 1 ELSE 0 END) AS Total_Refunds_Null_Count,
            SUM(CASE WHEN Total_Extra_Data_Charges IS NULL THEN 1 ELSE 0 END) AS Total_Extra_Data_Charges_Null_Count,
            SUM(CASE WHEN Total_Long_Distance_Charges IS NULL THEN 1 ELSE 0 END) AS Total_Long_Distance_Charges_Null_Count,
            SUM(CASE WHEN Total_Revenue IS NULL THEN 1 ELSE 0 END) AS Total_Revenue_Null_Count,
            SUM(CASE WHEN Customer_Status IS NULL THEN 1 ELSE 0 END) AS Customer_Status_Null_Count,
            SUM(CASE WHEN Churn_Category IS NULL THEN 1 ELSE 0 END) AS Churn_Category_Null_Count,
            SUM(CASE WHEN Churn_Reason IS NULL THEN 1 ELSE 0 END) AS Churn_Reason_Null_Count
        FROM stg_Churn;

        -- Expected Output (columns with null values):
        --  Value_Deal_Null_Count  3548
        --  Multiple_Lines_Null_Count  622
        --  Internet_Type_Null_Count  1390
        --  Online_Security_Null_Count  1390
        --  Online_Backup_Null_Count  1390
        --  Device_Protection_Plan_Null_Count 1390
        --  Premium_Support_Null_Count  1390
        --  Streaming_TV_Null_Count 1390
        --  Streaming_Movies_Null_Count 1390
        --  Streaming_Music_Null_Count  1390
        --  Unlimited_Data_Null_Count 1390
        --  Churn_Category_Null_Count 4686
        --  Churn_Reason_Null_Count 4686
          
        ```
4.  **Data Cleaning and Production Table:**
    *   Remove null values by replacing them with appropriate values (e.g., 'None', 'No', 'Others').
    *   Insert the cleaned data into a production table "prod\_Churn".
        ```sql
        SELECT 
            Customer_ID,
            Gender,
            Age,
            Married,
            State,
            Number_of_Referrals,
            Tenure_in_Months,
            ISNULL(Value_Deal, 'None') AS Value_Deal,
            Phone_Service,
            ISNULL(Multiple_Lines, 'No') As Multiple_Lines,
            Internet_Service,
            ISNULL(Internet_Type, 'None') AS Internet_Type,
            ISNULL(Online_Security, 'No') AS Online_Security,
            ISNULL(Online_Backup, 'No') AS Online_Backup,
            ISNULL(Device_Protection_Plan, 'No') AS Device_Protection_Plan,
            ISNULL(Premium_Support, 'No') AS Premium_Support,
            ISNULL(Streaming_TV, 'No') AS Streaming_TV,
            ISNULL(Streaming_Movies, 'No') AS Streaming_Movies,
            ISNULL(Streaming_Music, 'No') AS Streaming_Music,
            ISNULL(Unlimited_Data, 'No') AS Unlimited_Data,
            Contract,
            Paperless_Billing,
            Payment_Method,
            Monthly_Charge,
            Total_Charges,
            Total_Refunds,
            Total_Extra_Data_Charges,
            Total_Long_Distance_Charges,
            Total_Revenue,
            Customer_Status,
            ISNULL(Churn_Category, 'Others') AS Churn_Category,
            ISNULL(Churn_Reason , 'Others') AS Churn_Reason

        INTO [db_Churn].[dbo].[prod_Churn]
        FROM [db_Churn].[dbo].[stg_Churn];
        ```
5.  **View Creation for Power BI:**
    *   Create views "vw\_ChurnData" and "vw\_JoinData" to isolate data relevant for historical churn analysis and churn prediction, respectively.

        ```sql
        -- customers who have either churned or stayed with the company. For analyzing historical churn patterns
        Create View vw_ChurnData as select * from prod_Churn where Customer_Status In ('Churned', 'Stayed')
        -- customers who have recently joined the company. For later use in predicting future churn 
        Create View vw_JoinData as select * from prod_Churn where Customer_Status = 'Joined'
        ```

**Reasoning:**

*   SQL Server provides a robust and scalable environment for managing and processing data.
*   Staging tables ensure that raw data is preserved before any transformations.
*   Data exploration helps understand the characteristics of the data and identify potential issues.
*   Data cleaning is crucial for ensuring the accuracy and reliability of the analysis.
*   Views provide focused datasets for specific analytical tasks, simplifying data access in Power BI.



#### STEP 2: Power BI Data Transformation

**Objective:** Refine and enhance the data in Power BI for effective visualization and analysis.

**Actions Taken:**

1.  **Add Calculated Columns:**
    *   Create a "Churn Status" column, assigning 1 for churned customers and 0 otherwise. `= if [Customer_Status] = "Churned" then 1 else 0`. After Creation change Churn Status data type to numbers
    *   Create a "Monthly Charge Range" column, grouping monthly charges into buckets (e.g., "&lt; 20", "20-50", "50-100", "&gt; 100"). `Monthly Charge Range = if [Monthly_Charge] < 20 then "< 20" else if [Monthly_Charge] < 50 then "20-50" else if [Monthly_Charge] < 100 then "50-100" else "> 100"`
2.  **Create Mapping Tables:**
    *   Create a "mapping\_AgeGrp" table to categorize customer ages into groups (e.g., "&lt; 20", "20 - 35"). 
        - reference the prod_Churn table so that the source of data is the prod_Churn and we don't have to make unnecessary requests to sql server
        - Keep only Age column and remove duplicates
        - Add custom column Age Group `= if [Age] < 20 then "< 20" else if [Age] < 36 then "20 - 35" else if [Age] < 51 then "36 - 50" else "> 50"`
    
    *   Add a sorting column to ensure proper ordering in visualizations.
        - AgeGrpSorting `= if [Age Group] = "< 20" then 1 else if [Age Group] = "20 - 35" then 2 else if [Age Group] = "36 - 50" then 3 else 4`
        - Change data type of AgeGrpSorting to whole number
    *   Create a "mapping\_TenureGrp" table for tenure-based grouping.
        - reference the prod_Churn table for the table data
        - Keep only Tenure_in_Months and remove duplicates
        - Add custom column Tenure Group `= if [Tenure_in_Months] < 6 then "< 6 Months" else if [Tenure_in_Months] < 12 then "6-12 Months" else if [Tenure_in_Months] < 18 then "12-18 Months" else if [Tenure_in_Months] < 24 then "18-24 Months" else ">= 24 Months"`
    *   Add a sorting column for visualization ordering.
        - TenureGrpSorting `= if [Tenure Group] = "< 6 Months" then 1 else if [Tenure Group] =  "6-12 Months" then 2 else if [Tenure Group] = "12-18 Months" then 3 else if [Tenure Group] = "18-24 Months " then 4 else 5`
        - change datatype to number
3.  **Create Service Table:**
    *   Unpivot service-related columns (e.g., Phone\_Service, Multiple\_Lines) to consolidate service usage information.
    *   Rename columns for clarity.
        - Attribute --> Services
        - Value     --> Status

**Reasoning:**

*   Calculated columns provide derived metrics for analysis and visualization.
*   Mapping tables simplify visualizations and allow for grouped analysis.
*   Sorting columns ensure that visualizations display categorical data in the desired order.
*   Unpivoting service columns transforms data into a more suitable format for analysis and visualization.

#### STEP 3: Power BI Measure Creation

**Objective:** Calculate and define key metrics for the dashboard.

**Actions Taken:**

1.  **Create Measures:**
    *   **Total Customers:** Count the distinct number of customer IDs.
    *   **New Joiners:** Calculate the count of customer IDs where customer status is "Joined."
    *   **Total Churn:** Sum the "Churn Status" column to get the total number of churned customers.
    *   **Churn Rate:** Divide "Total Churn" by "Total Customers."

**Reasoning:**

*   Measures provide a way to calculate and display key metrics based on the underlying data.

**Code Snippets:**

*   **Total Customers Measure:**

```
Total Customers = COUNT(prod_Churn[Customer_ID])
```

*   **New Joiners Measure:**

```
New Joiners = CALCULATE(COUNT(prod_Churn[Customer_ID]), prod_Churn[Customer_Status] = “Joined”) 
```

*   **Churn Rate Measure:**

```
Total Churn = SUM(prod_Churn[Churn Status]) 
Churn Rate = [Total Churn] / [Total Customers]
```

#### STEP 4: Power BI Visualization - Summary Page

**Objective:** Design an informative and interactive dashboard summarizing key churn insights.

**Actions Taken:**

1.  **Summary Page Design:**
    *   Create a "Summary" page with various visualizations covering demographic, account information, geographic distribution, churn distribution, and services used.
    *   Utilize cards, donut charts, bar charts, line and stacked column charts, and tables for data visualization.
        - Top Card
            - Total Customers
            - New Joiners
            - Total Churn
            - Churn Rate%

        - Demographic
            - Gender – Churn Rate (donut chart)
            - Age Group – Total Customer & Churn Rate (line and stacked column chart)

        - Account Info
            - Payment Method – Churn Rate (Clustered bar chart)
            - Contract – Churn Rate (Clustered bar chart)
            - Tenure Group – Total Customer & Churn Rate (line and stacked column chart)

        - Geographic
            - Top 5 State – Churn Rate (Clustered bar chart and Filtertop 5 States)

        - Churn Distribution
            - Churn Category – Total Churn (Clustered bar chart)
            - Churn Reason Page (Tooltip) : Churn Reason – Total Churn (Table)

        - Service Used
            - Internet Type – Churn Rate (Clustered bar chart)
            - prod_Service >> Services – Status – % RT Sum of Churn Status (Matrix)
        
    *   Implement tooltips to provide detailed insights for specific data points.
    *   Use slicers for interactive filtering based on monthly charge range and marital status.

**Reasoning:**

*   A well-structured summary page provides a clear overview of churn patterns and key drivers.
*   Diverse visualization types cater to different data representation needs and enhance user understanding.
*   Tooltips provide a mechanism to reveal more detailed information without cluttering the main visualization.
*   Slicers empower users to interact with the dashboard and explore data from different perspectives.

#### STEP 5: Predict Customer Churn using Random Forest

**Objective:** Develop a machine learning model to predict the likelihood of churn for new customers.

**Actions Taken:**

1.  **Data Preparation:**
    *   Export "vw\_ChurnData" and "vw\_JoinData" from SQL Server to an Excel file.
2.  **Python Environment Setup:**
    *   Install necessary Python libraries, including pandas, numpy, scikit-learn, and joblib. `pip install -r requirements.txt`
3.  **Data Preprocessing:**
    *   Load data into a pandas DataFrame.
    *   Drop irrelevant columns (e.g., Customer\_ID, Churn\_Category).
    *   Encode categorical variables into numerical values using LabelEncoder.
    *   Manually encode the target variable "Customer\_Status" for consistent interpretation.
    *   Split data into training (80%) and testing (20%) sets.
4.  **Model Training:**
    *   Initialize a Random Forest Classifier with 1000 decision trees.
    *   Train the model on the training data.
5.  **Model Evaluation:**
    *   Make predictions on the test data.
    *   Analyze model performance using a confusion matrix, classification report, and feature importance visualization.
6.  **Prediction on New Data:**
    *   Load the "vw\_JoinData" into a DataFrame.
    *   Preprocess the data similarly to the training data.
    *   Make predictions using the trained Random Forest model.
    *   Save the predictions to a CSV file.

**Reasoning:**

*   Random Forest is a simple yet a powerful and widely used algorithm for classification tasks like churn prediction.
*   Data preprocessing ensures that the model receives data in a suitable format.
*   Splitting data into training and testing sets allows for unbiased model evaluation.
*   Model evaluation metrics (confusion matrix, classification report) provide insights into model performance.
*   Feature importance analysis helps identify the most influential variables in predicting churn.

**Code Snippets:**

*   **Data Preprocessing (Encoding):**

```python
# Encode categorical variables except the target variable
label_encoders = {}
for column in columns_to_encode:
    label_encoders[column] = LabelEncoder()
    data[column] = label_encoders[column].fit_transform(data[column])

# Manually encode the target variable 'Customer_Status'
data['Customer_Status'] = data['Customer_Status'].map({'Stayed': 0, 'Churned': 1})
```

*   **Model Training:**

```python
# Initialize the Random Forest Classifier
rf_model = RandomForestClassifier(n_estimators=1000, random_state=42)

# Train the model
rf_model.fit(X_train, y_train)
```

*   **Model Evaluation (Confusion Matrix):**

```python
print("Confusion Matrix:")
print(confusion_matrix(y_test, y_pred))
```

*   **Prediction on New Data:**

```python
# Make predictions
new_predictions = rf_model.predict(new_data)
```

#### STEP 6: Power BI Visualization - Churn Prediction Page

**Objective:** Visualize churn prediction results and build a churner profile.

**Actions Taken:**

1.  **Import Prediction Data:** Import the CSV file containing churn predictions into Power BI.
2.  **Create Visualizations:**
    -   create Measures
        - `Count Predicted Churner = COUNT(Predictions[Customer_ID]) + 0`

        - `Title Predicted Churners = "COUNT OF PREDICTED CHURNERS : " & COUNT(Predictions[Customer_ID])`
    *   Display predicted churn count using cards.
    - Right Side Grid
        - Customer ID
        - Monthly Charge
        - Total Revenue
        - Total Refunds
        - Number of Referrals
    *   Create visualizations to analyze predicted churners based on:
        *   Demographics (gender, age group, marital status) - Churn Count.
        *   Account information (payment method, contract type, tenure group) -  Churn Count.
        *   Geographic distribution state - Churn count.
3.  **Dashboard Navigation:** Add buttons to navigate between the Summary and Churn Prediction pages.

**Reasoning:**

*   Visualizing the predicted churners provides a clear understanding of their characteristics.
*   Analyzing churn predictions across various dimensions helps identify specific customer segments at risk of churning.
*   Dashboard navigation enhances user experience by enabling seamless switching between different analytical views.

### Reproducing the Project

To replicate this project, you'll need:

1.  **Access to the Raw Data:** Obtain the CSV file containing customer data.
2.  **Microsoft SQL Server:** Install and configure SQL Server. You can also use a cloud-based SQL database if preferred.
3.  **SQL Server Management Studio (SSMS):** Install SSMS to interact with the SQL Server database.
4.  **Power BI Desktop:** Install Power BI Desktop for data transformation, visualization, and dashboard creation.
5.  **Python and Jupyter Notebook:** Install Python and Jupyter Notebook. You can use Anaconda, which bundles these tools together, along with essential libraries.
6.  **Required Python Libraries:** Install pandas, numpy, matplotlib, seaborn, scikit-learn, and joblib.

### Conclusion

This Power BI churn analysis project effectively combines data processing, visualization, and predictive modeling techniques. It not only provides a comprehensive view of historical churn patterns but also empowers stakeholders with the ability to predict and potentially mitigate future churn. The interactive dashboard, combined with the machine learning-driven insights, serves as a valuable tool for data-driven decision-making aimed at improving customer retention and business performance.
