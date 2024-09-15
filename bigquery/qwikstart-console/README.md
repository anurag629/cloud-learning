# BigQuery: Qwik Start - Console

## Overview

Google BigQuery is a fully-managed, serverless data warehouse that enables fast SQL queries using the processing power of Google's infrastructure. This lab introduces you to BigQuery and guides you through querying public datasets, creating your own datasets and tables, loading data, and running queries.

## Objectives

By the end of this lab, you will be able to:

- Query a public dataset using BigQuery.
- Create a new dataset in your BigQuery project.
- Load data into a new table within your dataset.
- Run queries against your custom table.

### Task 1: Access the BigQuery Console

1. **Open the Google Cloud Console**:
   - Navigate to [https://console.cloud.google.com/](https://console.cloud.google.com/).
   - Log in using your Google account credentials.

2. **Open BigQuery**:
   - In the Cloud Console, click on the **Navigation Menu** (☰) in the top-left corner.
   - Scroll down to the **Big Data** section and select **BigQuery**.
   - If prompted with a welcome message or tour, you can close it or proceed as desired.

### Task 2: Query a Public Dataset

1. **Compose a New Query**:
   - In the BigQuery console, click on the **Compose New Query** button or the **+** icon.

2. **Write the Query**:
   - Paste the following SQL query into the query editor:
     ```sql
     SELECT
       weight_pounds,
       state,
       year,
       gestation_weeks
     FROM
       `bigquery-public-data.samples.natality`
     ORDER BY
       weight_pounds DESC
     LIMIT 10;
     ```
     - This query retrieves the top 10 records of babies with the highest birth weights from the public natality dataset.

3. **Run the Query**:
   - Click on the **Run** button.
   - Wait for the query to execute and observe the results displayed in the bottom pane.
   - Review the data to familiarize yourself with the dataset.

### Task 3: Create a New Dataset

1. **Initiate Dataset Creation**:
   - In the **Explorer** panel on the left, find your project ID.
   - Hover over your project ID, click on the **Options** icon (⋮), and select **Create dataset**.

2. **Configure the Dataset**:
   - **Dataset ID**: Enter `babynames`.
   - **Data Location**: Leave it as the default (typically `US`).
   - **Default Table Expiration**: Leave blank unless you want tables to expire after a certain number of days.
   - **Encryption**: Use the default settings.
   - Click on **Create dataset**.

3. **Verify Dataset Creation**:
   - In the **Explorer** panel, you should now see the `babynames` dataset under your project.

### Task 4: Load Data into a New Table

1. **Start Table Creation**:
   - Click on the `babynames` dataset to select it.
   - Click on the **Create Table** button.

2. **Specify the Source Data**:
   - **Create table from**: Select **Google Cloud Storage**.
   - **Select file from GCS bucket**: Enter `gs://spls/gsp072/baby-names/yob2014.txt`.
     - This is a publicly accessible file containing baby names data from 2014.
   - **File format**: Choose **CSV**.

3. **Set the Destination Table**:
   - **Dataset name**: Should be `babynames`.
   - **Table name**: Enter `names_2014`.

4. **Define the Schema**:
   - Under **Schema**, uncheck **Auto detect**.
   - Click on **Edit as text** and enter the following schema:
     ```
     name:STRING, gender:STRING, count:INTEGER
     ```
     - This defines the data types for each column in the CSV file.

5. **Create the Table**:
   - Leave other settings at their defaults.
   - Click on **Create table**.
   - Wait for the data to be loaded into BigQuery.

6. **Verify Table Creation**:
   - Once the table is created, it will appear under the `babynames` dataset in the **Explorer** panel.

### Task 5: Preview the Table Data

1. **View Table Data**:
   - Click on the `names_2014` table.
   - Select the **Preview** tab to view a sample of the data.
   - Verify that the data matches the expected format and content.

### Task 6: Query Your Custom Dataset

1. **Compose a New Query**:
   - Click on **Compose New Query**.

2. **Write the Query**:
   - Enter the following SQL query, replacing `your-project-id` with your actual project ID:
     ```sql
     SELECT
       name,
       count
     FROM
       `your-project-id.babynames.names_2014`
     WHERE
       gender = 'M'
     ORDER BY
       count DESC
     LIMIT 5;
     ```
     - This query retrieves the top 5 most popular male baby names from 2014.

3. **Run the Query**:
   - Click on the **Run** button.
   - Review the results to see the names and their counts.

4. **Explore Further**:
   - Modify the query to retrieve female baby names by changing `gender = 'M'` to `gender = 'F'`.
   - Experiment with different `LIMIT` values or add conditions to the `WHERE` clause.

### Task 7: Understand BigQuery Pricing (Optional)

- **Note**: BigQuery charges based on the amount of data processed by your queries.
- **Estimate Query Cost**:
  - Before running a query, BigQuery displays an estimate of the data it will process.
  - Keep an eye on this estimate to manage costs, especially with large datasets.
- **Best Practices**:
  - Use `LIMIT` clauses to reduce data scanned.
  - Select only the necessary columns instead of using `SELECT *`.

## Cleanup (Optional)

If you're using a personal Google Cloud account and want to avoid incurring charges:

- Delete the dataset:
  - In the BigQuery console, hover over the `babynames` dataset.
  - Click on the **Options** icon (⋮) and select **Delete dataset**.
  - Confirm the deletion.

## Conclusion

In this lab, you:

- Queried a public dataset using BigQuery.
- Created your own dataset and loaded data into it.
- Ran SQL queries against your custom table.
- Learned how BigQuery can efficiently handle large datasets.

BigQuery is a powerful tool for data analysis, enabling you to process large amounts of data quickly and cost-effectively. By leveraging BigQuery's capabilities, you can gain valuable insights from your data with minimal infrastructure overhead.
