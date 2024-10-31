# Data Cleaning Script for Pionex CSV Files

![RobotClean](RobotClean.png)

Here’s a comprehensive, step-by-step guide for using the Python code for data cleaning. This guide explains each part of the code, covering setup, data loading, processing steps, and saving the results. You can follow these steps to clean and organize your dataset on Google Colab.

# Features of this code
Here’s a list of features provided by this Python code for data cleaning:

- Drive Mounting and Data Loading: Connects Google Drive to access and load a CSV dataset directly into a pandas DataFrame.

- Garbage Bin for Unwanted Data: Initializes a separate DataFrame, garbage_bin, to collect rows and columns that do not meet specified criteria, preserving the original DataFrame for further cleaning.

- Column Selection and Dropping: Moves specified columns (BrandCode and Lang) to the garbage bin and removes them from the main DataFrame for a cleaner dataset.

- Date and Time Extraction: Converts a RegistrationDate column to a datetime format, then separates it into distinct Registration Date and Registration Time columns.

- Name Standardization: Formats First Name and Last Name columns to title case, ensuring consistent capitalization.

- Splitting Name Components: Splits complex names into separate parts, handling first and last name entries more effectively, and fills any missing components with empty strings.

- Missing Value Handling: Removes rows with missing values in essential columns (e.g., First Name and Last Name), moving them to the garbage bin.

- Number and Special Character Validation: Identifies and removes rows where names contain numbers or specified special characters (@, %, $, #, _), moving these to the garbage bin.

- Short Name Filtering: Removes rows where either First Name or Last Name is a single character, as these may indicate incomplete data entries.

- Date Consistency Check: Ensures rows contain valid values in both Registration Date and Registration Time columns, moving rows with missing values to the garbage bin.

- Phone Number Validation: Verifies phone numbers to ensure they contain only numeric characters, moving rows with non-numeric phone numbers to the garbage bin.

- Column Reordering: Reorders columns to place specific columns (like First Name and Last Name) at the beginning of the DataFrame for easier reading and analysis.

- Final Data Saving: Saves the cleaned DataFrame to a new CSV file, with a separate file for garbage_bin data, providing a clear distinction between processed and discarded data.

These features help streamline the process of loading, cleaning, and organizing large datasets efficiently.

## Step 1: Setup
To begin, make sure you have your dataset uploaded to Google Drive and Google Colab connected to it.

### Import Necessary Libraries
### 1) First, import libraries that are essential for data manipulation and processing.

```python
from google.colab import drive
import pandas as pd
import numpy as np
import csv
import re  # for regular expressions
```
- pandas: For handling data frames.
- numpy: For numerical operations.
- csv: To handle CSV files.
- re: For regular expressions, useful for pattern matching in text data.

### Mount Google Drive
To access files stored on Google Drive, mount it using: 

```python
drive.mount('/content/drive')
```
### Define the File Path
Specify the path of the file in your Google Drive. Update file_path with your actual file path.

```python
file_path = 'your_file_path'
```
## Step 2: Load the Data
Read the CSV file into a DataFrame, df, and print the first few rows to verify the data was loaded correctly.

```python
df = pd.read_csv(file_path)
print(df.head(20))  # Display the first 20 rows
```

## Step 3: Create a Garbage Bin for Discarded Data
This will hold rows that do not meet certain cleaning criteria.
```python
garbage_bin = pd.DataFrame()  # Initialize empty DataFrame
```

## Step 4: Move Unwanted Columns to Garbage Bin
Columns such as BrandCode and Lang are moved to garbage_bin for removal from the main dataset.

```python
garbage_bin = df[['BrandCode', 'Lang']]
df = df.drop(['BrandCode', 'Lang'], axis=1)
```
## Step 5: Format the Registration Date
Convert RegistrationDate column into separate Registration Date and Registration Time columns for better readability.

```python
df['RegistrationDate'] = pd.to_datetime(df['RegistrationDate'], errors='coerce')
df['Registration Date'] = df['RegistrationDate'].dt.date
df['Registration Time'] = df['RegistrationDate'].dt.time
df = df.drop('RegistrationDate', axis=1)  # Drop the original column if unneeded
```
## Step 6: Standardize Name Formats
Standardize names in First Name and Last Name columns using title case, splitting names if needed.
```python
df['First Name'] = df['First Name'].str.title()
df['Last Name'] = df['Last Name'].str.title()
df[['First Name_new', 'Last Name_new']] = df['First Name'].str.split(' ', n=1, expand=True)
df['Last Name_new'] = df['Last Name_new'].fillna('')  # Fill missing parts with an empty string
df['Last Name_new'] = df['Last Name'] + ' ' + df['Last Name_new']
df = df.drop(['First Name', 'Last Name'], axis=1)  # Drop original columns
df = df.rename(columns={'First Name_new': 'First Name', 'Last Name_new': 'Last Name'})
```
## Step 7: Handle Missing Names
Remove rows with missing values in First Name or Last Name.

```python
missing_values_rows = df[df['First Name'].isna() | df['Last Name'].isna()]
garbage_bin = pd.concat([garbage_bin, missing_values_rows], ignore_index=True)
df = df.dropna(subset=['First Name', 'Last Name'])
```
## Step 8: Remove Names with Numbers or Special Characters
Define functions to identify and remove rows where names contain numbers or special characters.

```python
def contains_numbers(s):
    return any(char.isdigit() for char in s)

rows_with_numbers = df[df['First Name'].apply(contains_numbers) | df['Last Name'].apply(contains_numbers)]
garbage_bin = pd.concat([garbage_bin, rows_with_numbers], ignore_index=True)
df = df[~df['First Name'].apply(contains_numbers) & ~df['Last Name'].apply(contains_numbers)]

special_chars = ['@', '%', '$', '#', '_']

def contains_special_chars(text):
    return any(char in str(text) for char in special_chars)

special_char_rows = df[df['First Name'].apply(contains_special_chars) | df['Last Name'].apply(contains_special_chars)]
garbage_bin = pd.concat([garbage_bin, special_char_rows], ignore_index=True)
df = df[~df['First Name'].apply(contains_special_chars) & ~df['Last Name'].apply(contains_special_chars)]
```

## Step 9: Remove Entries with Missing Registration Information
Rows with missing Registration Date or Registration Time are moved to garbage_bin.

```python
missing_registration_rows = df[df['Registration Date'].isna() | df['Registration Time'].isna()]
garbage_bin = pd.concat([garbage_bin, missing_registration_rows], ignore_index=True)
df = df.dropna(subset=['Registration Date', 'Registration Time'])
```

## Step 10: Filter Phone Numbers
Filter rows with numeric-only phone numbers to ensure all phone numbers are valid.

```python
def is_numeric(s):
    try:
        float(s)  # Check if it can be converted to a number
        return True
    except ValueError:
        return False

numeric_phone_rows = df[df['Phone'].apply(is_numeric)]
non_numeric_phone_rows = df[~df['Phone'].apply(is_numeric)]
garbage_bin = pd.concat([garbage_bin, non_numeric_phone_rows], ignore_index=True)
df = numeric_phone_rows
```

## Step 11: Save the Cleaned Data
Finally, save the cleaned data and the garbage data separately for review and analysis.

```python
output_file_path = 'your_file_path'
df.to_csv(output_file_path, index=False)

garbage_output_file_path = 'your_file_path'
garbage_bin.to_csv(garbage_output_file_path, index=False)
```

### Final Notes
Review Each Step: Double-check each stage to ensure all unwanted data has been moved to garbage_bin.
Backup Data: Keep a copy of the original data before performing irreversible operations like dropping rows or columns.
Document Adjustments: Any changes to column names or filtering criteria should be documented to keep track of modifications made.







