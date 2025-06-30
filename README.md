# week7_assignment

# Azure Data Factory Pipeline for Daily Truncate and Load of CSV Files

This project implements an automated Azure Data Factory pipeline that loads data from CSV files stored in Azure Data Lake Gen2 into Azure SQL Database tables. It is designed to perform a truncate-load operation on a daily basis.

## Project Objective

There are three types of files stored in the same container in the data lake. The pipeline reads each file type, processes it as per its naming pattern, applies necessary transformations, and loads the data into its respective SQL table. The process ensures any previously loaded data is removed before new data is loaded.

## File Types Handled

1. Files starting with CUST_MSTR followed by a date in the format YYYYMMDD. Example: CUST_MSTR_20191112.csv
2. Files starting with master_child_export followed by a date in the format YYYYMMDD. Example: master_child_export-20191112.csv
3. A fixed file named H_ECOM_ORDER.csv

## Behavior for Each File Type

- For CUST_MSTR files:
  - Extract the date from the file name.
  - Add a new column named LoadDate in the format YYYY-MM-DD.
  - Load the data into the SQL table named CUST_MSTR.
  - Truncate the table before loading.

- For master_child_export files:
  - Extract the date from the file name.
  - Add two new columns: LoadDate in the format YYYY-MM-DD and DateKey in the format YYYYMMDD.
  - Load the data into the SQL table named master_child.
  - Truncate the table before loading.

- For H_ECOM_ORDER.csv:
  - Load the file as-is into the SQL table named H_ECOM_Orders.
  - Truncate the table before loading.

## Pipeline Design Steps

1. Use the Get Metadata activity to list all CSV files in the data lake container path. This collects all file names in the folder.
2. Use a ForEach activity to loop over the list of file names.
3. Inside the ForEach loop, use If Condition activities to check the type of file based on the filename prefix:
   - Starts with CUST_MSTR
   - Starts with master_child_export
   - Equals H_ECOM_ORDER.csv
4. For each file type, do the following:
   - Extract the date from the file name using the substring function.
   - Call a Script activity to truncate the respective target table in the database.
   - Use a Data Flow (for CUST_MSTR and master_child_export) to add the new date columns from the file name.
   - Use a Copy activity (for H_ECOM_ORDER) to load the file as-is.
5. Parameterize the folder path and file name in the DelimitedText datasets so that the same dataset can be reused dynamically.
6. Use a Schedule Trigger to run the pipeline once per day, automating the daily truncate-and-load process.

## Components in the Repository

- ARMTemplates folder contains exported pipeline definitions in JSON format. These can be imported into any ADF instance.
- Scripts folder contains sample SQL scripts used to truncate tables.
- README.md explains the implementation and purpose of the pipeline.

## Deployment Instructions

1. Open Azure Data Factory Studio.
2. Navigate to the Manage section and select ARM template.
3. Use the Import option to upload the pipeline JSON files from the ARMTemplates folder.
4. Reconfigure Linked Services as required for your storage and SQL environments.
5. Create the daily trigger to run the pipeline automatically.

## Requirements

- Azure Data Factory instance
- Azure Data Lake Storage Gen2 account with flat folder structure
- Azure SQL Database with tables CUST_MSTR, master_child, and H_ECOM_Orders
- Permissions to truncate tables and load data via ADF

## Output

After running the pipeline, each of the SQL tables will contain the latest data from the respective files in the data lake. The tables are truncated before new data is inserted to ensure data freshness every day.

