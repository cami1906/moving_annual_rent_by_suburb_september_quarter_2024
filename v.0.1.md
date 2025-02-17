## 1. Reading the Excel File
### First, load all sheets from the Excel file using two header rows:
```python
import pandas as pd
```

# Read all worksheets with two header rows
```python
all_sheets = pd.read_excel(
    'moving_annual_rent_by_suburb_september_quarter_2024_copy.xlsx',
    sheet_name=None,
    header=[0, 1]
)
print("Sheets found:", list(all_sheets.keys()))
```

### To inspect a specific sheet (e.g., '1_bedroom_flat'):
```python
all_sheets['1_bedroom_flat'].head()
```
## 2. Processing a Single Sheet
### Let's process the '1_bedroom_flat' sheet step by step.

## a. Select the Sheet
### Select the '1_bedroom_flat' sheet
```python
sheet_name = '1_bedroom_flat'
df = all_sheets[sheet_name]
print(f"\nProcessing sheet: {sheet_name}")
print("Original columns:", df.columns.tolist())
```

## b. Reshape the Data
### 1. Create a 'wide' DataFrame by setting the 'Area' column as the index
```python
df_wide_data = df.set_index(('Unnamed: 0_level_0', 'Area'))
```

### 2. Stack the top-level columns (quarter/date columns) to get a 'long' DataFrame
```python
df_long_data = df_wide_data.stack(level=0)
```

### 3. Rename the new index levels: 'Area' and 'Quarter'
```python
df_long_data.index.names = ['Area', 'Quarter']
```

### 4. Reset the index so that 'Area' and 'Quarter' become columns (reformatted DataFrame)
```python
df_reformatted = df_long_data.reset_index()
```

## c. Convert Columns to Desired Formats
### Format 'Area' as string
```python
df_reformatted['Area'] = df_reformatted['Area'].astype(str)
```

### Convert 'Quarter' to datetime, then format as abbreviated month and year
```pythondf_reformatted['Quarter'] = pd.to_datetime(df_reformatted['Quarter'], errors='coerce')
df_reformatted['Quarter'] = df_reformatted['Quarter'].dt.strftime('%b %Y')
```

### Convert 'Count' to an integer (nullable Int64)
```pythondf_reformatted['Count'] = pd.to_numeric(df_reformatted['Count'], errors='coerce').astype('Int64')
```

### Convert 'Median' to numeric, then format as currency with no decimals
```python
df_reformatted['Median'] = pd.to_numeric(df_reformatted['Median'], errors='coerce')
df_reformatted['Median'] = df_reformatted['Median'].apply(
    lambda x: "${:,.0f}".format(x) if pd.notnull(x) else ""
)
```

## d. Inspect the Processed Data
### Return the first 5 rows for inspection
```python
df_reformatted.head()
```

### Display DataFrame information
```pythondf_reformatted.info()
```

### List unique values in the 'Area' column
```python
print(df_reformatted['Area'].unique().tolist())
```

### Display DataFrame first 5 rows
```python
df_reformatted.head()
```

## 3. Processing All Sheets
### The following code processes each sheet and stores the reformatted DataFrames in a dictionary.
```python
# Dictionary to store the processed DataFrames from each sheet
processed_sheets = {}

### Iterate over each sheet in the Excel file
for sheet_name, df in all_sheets.items():
    print(f"\nProcessing sheet: {sheet_name}")
    print("Original columns:", df.columns.tolist())
    
    ### 1. Create a 'wide' DataFrame by setting the 'Area' column as the index
    df_wide_data = df.set_index(('Unnamed: 0_level_0', 'Area'))
    
    ### 2. Stack the top-level columns (quarter/date columns) to get a 'long' DataFrame
    df_long_data = df_wide_data.stack(level=0)
    
    ### 3. Rename the new index levels: 'Area' and 'Quarter'
    df_long_data.index.names = ['Area', 'Quarter']
    
    ### 4. Reset the index so that 'Area' and 'Quarter' become columns
    df_reformatted = df_long_data.reset_index()
    
    ### 5. Convert columns to desired types/formats
    df_reformatted['Area'] = df_reformatted['Area'].astype(str)
    df_reformatted['Quarter'] = pd.to_datetime(df_reformatted['Quarter'], errors='coerce')
    df_reformatted['Quarter'] = df_reformatted['Quarter'].dt.strftime('%b %Y')
    df_reformatted['Count'] = pd.to_numeric(df_reformatted['Count'], errors='coerce').astype('Int64')
    df_reformatted['Median'] = pd.to_numeric(df_reformatted['Median'], errors='coerce')
    df_reformatted['Median'] = df_reformatted['Median'].apply(
        lambda x: "${:,.0f}".format(x) if pd.notnull(x) else ""
    )
    
    ### 6. Optionally, print the first few rows for inspection
    print(df_reformatted.head())
    print(f"Unique 'Area' values for sheet '{sheet_name}':", df_reformatted['Area'].unique().tolist())

    ### 7. Store the processed DataFrame in the dictionary
    processed_sheets[sheet_name] = df_reformatted
```
    
## 4. Correct unexpected output
### Clean the 'Area' column: Replace occurrences of "Median.1" with "Inner_Melbourne_Albert "
```python
df_reformatted['Area'] = df_reformatted['Area'].str.replace(r'^Median\.1\s*', 'Inner_Melbourne_Albert ', regex=True)
if 'all_properties' in processed_sheets:
    print("First few rows of 'all_properties' sheet:")
    print(processed_sheets['all_properties'].head())
else:
    print("Sheet 'all_properties' not found in processed_sheets.")
```

## 5. Exporting the Processed Sheets
## a. Write All Processed Sheets to a Single Multi-Tab Excel File
```python
output_filename = 'moving_annual_rent_by_suburb_september_quarter_2024_reformatted.xlsx'
with pd.ExcelWriter(output_filename) as writer:
    for sheet_name, df_reformatted in processed_sheets.items():
        df_reformatted.to_excel(writer, sheet_name=sheet_name, index=False)
print(f"\nAll processed sheets have been written to {output_filename}")
```

## b. Write Each Processed Sheet to a Separate Excel File and Compress into a ZIP file
```python
import os
import shutil
```

### 1. Create a temporary output directory to hold individual Excel files
```python
temp_output_dir = 'moving_annual_rent_by_suburb_september_quarter_2024_temp'
if not os.path.exists(temp_output_dir):
    os.makedirs(temp_output_dir)
```

### 2. Write each processed DataFrame to its own Excel file in the temporary directory
```python
for sheet_name, df_reformatted in processed_sheets.items():
    ### Construct a file name (sanitise sheet_name if needed)
    output_file = os.path.join(temp_output_dir, f"{sheet_name}.xlsx")
    df_reformatted.to_excel(output_file, index=False)
    print(f"Sheet '{sheet_name}' written to {output_file}")
```

### 3. Define the base name for the zip file (without the .zip extension)
```python
zip_base_name = 'moving_annual_rent_by_suburb_september_quarter_2024'
zip_filename = f"{zip_base_name}.zip"
```

### 4. Create a zip file from the temporary output directory
```pythonshutil.make_archive(zip_base_name, 'zip', temp_output_dir)
print(f"\nAll processed sheets have been zipped into {zip_filename}")
```
