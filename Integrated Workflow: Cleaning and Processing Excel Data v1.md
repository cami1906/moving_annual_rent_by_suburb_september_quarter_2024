# Integrated Workflow: Cleaning and Processing Excel Data

## 1. Cleaning Excel Worksheets using VBA

### 1.1 Overview
The VBA script performs the following actions:
### - **a.** Adjust worksheet names for improved readability in Jupyter Notebook and SQL.
### - **b.** Remove unnecessary header rows and fields.
### - **c.** Concatenate non-empty values from Column A with the corresponding value in Column B into a new Column C.
### - **d.** Delete the original Columns A and B.

### 1.2 VBA Script

### Open the file `moving_annual_rent_by_suburb_september_quarter_2024_vba.xlsx` in Excel and run the following VBA macro:

```vba
```vba
Sub DeleteRowsWithGroupTotalAndCleanHeaders()
    Dim wb As Workbook
    Dim ws As Worksheet
    Dim wsNames As Variant
    Dim i As Long, j As Long
    Dim lastRow As Long
    Dim isMerged As Boolean
    Dim currentArea As String
    Dim originalString As String, processedString As String

    ' Define the worksheet names
    wsNames = Array("1_bedroom_flat", "2_bedroom_flat", "3_bedroom_flat", _
                    "2_bedroom_house", "3_bedroom_house", "4_bedroom_house", "all_properties")

    ' Set the workbook
    On Error Resume Next
    Set wb = Workbooks("moving_annual_rent_by_suburb_september_quarter_2024_vba.xlsx")
    On Error GoTo 0

    If wb Is Nothing Then
        MsgBox "Workbook not found. Please ensure the file name is correct and the workbook is open.", vbExclamation
        Exit Sub
    End If

    ' Loop through each worksheet
    For i = LBound(wsNames) To UBound(wsNames)
        On Error Resume Next
        Set ws = wb.Sheets(wsNames(i))
        On Error GoTo 0

        If Not ws Is Nothing Then
            ' Check if cells A1 and A2 are merged
            If Not IsNull(ws.Range("A1:A2").MergeCells) Then
                isMerged = ws.Range("A1:A2").MergeCells
            Else
                isMerged = False ' Assume not merged if MergeCells is Null
            End If

            ' If merged, unmerge them
            If isMerged Then
                ws.Range("A1:A2").UnMerge
            End If

            ' Clear the contents of cell A2
            ws.Range("A2").ClearContents

            ' Find the last row in column B
            lastRow = ws.Cells(ws.Rows.Count, "B").End(xlUp).Row

            ' Loop through rows in reverse and delete rows containing "Group Total"
            For j = lastRow To 1 Step -1
                If InStr(1, ws.Cells(j, 2).Value, "Group Total", vbTextCompare) > 0 Then
                    ws.Rows(j).Delete
                End If
            Next j

            ' Delete row 1 (after all other operations are complete)
            ws.Rows(1).Delete

            ' Re-merge cells A1 and A2 if they were originally merged
            If isMerged Then
                ws.Range("A1:A2").Merge
            End If

            ' Insert a new column to the right of column B
            ws.Columns("C").Insert Shift:=xlToRight

            ' Add the header "Area" in cell C2
            ws.Cells(2, 3).Value = "Area"

            ' Find the last row in column A after deletions
            lastRow = ws.Cells(ws.Rows.Count, "A").End(xlUp).Row

            ' Initialize currentArea as empty
            currentArea = ""
            ' Loop through each row starting from row 3 (row 2 contains headers)
            For j = 3 To lastRow
                ' If Column A has a non-empty value, update currentArea
                If Len(Trim(ws.Cells(j, 1).Value)) > 0 Then
                    originalString = Trim(ws.Cells(j, 1).Value)
                    processedString = Replace(originalString, " ", "_") & "_"  ' Replace all spaces with '_' and add an extra '_'
                    currentArea = processedString
                End If

                ' Concatenate currentArea with the value in Column B and write to Column C
                ws.Cells(j, 3).Value = currentArea & " " & ws.Cells(j, 2).Value
            Next j

            ' Delete columns A and B now that concatenation is complete.
            ws.Columns("A:B").Delete
        Else
            MsgBox "Worksheet '" & wsNames(i) & "' not found.", vbExclamation
        End If
    Next i

    MsgBox "Operations completed: Rows deleted, new column added, concatenation performed, and columns A and B deleted.", vbInformation
End Sub
```

### 1.3 Instructions for VBA Part
    1.3.1. Insert the above VBA code into a new module via the VBA editor.
    1.3.2. Run the macro DeleteRowsWithGroupTotalAndCleanHeaders.
    1.3.3. Save the cleaned file as moving_annual_rent_by_suburb_september_quarter_2024_copy.xlsx for subsequent Python processing.
    1.3.4. Open moving_annual_rent_by_suburb_september_quarter_2024_vba.xlsx in Excel.

## 2. Processing and Exporting Data with Python
### 2.1 Overview
### The Python script performs the following tasks:
 
### - Reads the cleaned Excel file (with two header rows).
### - Processes a single sheet (e.g., 1_bedroom_flat) as an example, then processes all sheets.
### - Converts the data from wide to long format.
### - Formats data types and values.
### - Exports the results as a multi-tab Excel file.
### - Optionally exports each sheet into separate Excel files and compresses them into a ZIP file.


### 2.2 Reading the Excel File
```python
import pandas as pd

# Read all worksheets with two header rows
all_sheets = pd.read_excel(
    'moving_annual_rent_by_suburb_september_quarter_2024_copy.xlsx',
    sheet_name=None,
    header=[0, 1]
)
print("Sheets found:", list(all_sheets.keys()))
```

### 2.3 Processing a Single Sheet (Example: '1_bedroom_flat')
```python
sheet_name = '1_bedroom_flat'
df = all_sheets[sheet_name]
print(f"\nProcessing sheet: {sheet_name}")
print("Original columns:", df.columns.tolist())

# Create a 'wide' DataFrame by setting the 'Area' column as the index
df_wide_data = df.set_index(('Unnamed: 0_level_0', 'Area'))

# Stack the top-level columns to get a 'long' DataFrame
df_long_data = df_wide_data.stack(level=0)

# Rename the new index levels: 'Area' and 'Quarter'
df_long_data.index.names = ['Area', 'Quarter']

# Reset the index so that 'Area' and 'Quarter' become columns
df_reformatted = df_long_data.reset_index()
```

### 2.4 Converting Columns to Desired Formats
```python
# Format 'Area' as string
df_reformatted['Area'] = df_reformatted['Area'].astype(str)

# Convert 'Quarter' to datetime, then format as abbreviated month and year
df_reformatted['Quarter'] = pd.to_datetime(df_reformatted['Quarter'], errors='coerce')
df_reformatted['Quarter'] = df_reformatted['Quarter'].dt.strftime('%b %Y')

# Convert 'Count' to an integer (nullable Int64)
df_reformatted['Count'] = pd.to_numeric(df_reformatted['Count'], errors='coerce').astype('Int64')

# Convert 'Median' to numeric, then format as currency with no decimals
df_reformatted['Median'] = pd.to_numeric(df_reformatted['Median'], errors='coerce')
df_reformatted['Median'] = df_reformatted['Median'].apply(
    lambda x: "${:,.0f}".format(x) if pd.notnull(x) else ""
)
```

### 2.5 Inspecting the Processed Data
```python
# Return the first 5 rows for inspection
print(df_reformatted.head())

# Display DataFrame information
df_reformatted.info()

# List unique values in the 'Area' column
print(df_reformatted['Area'].unique().tolist())
```

### Processing All Sheets
```python
# Dictionary to store the processed DataFrames from each sheet
processed_sheets = {}

for sheet_name, df in all_sheets.items():
    print(f"\nProcessing sheet: {sheet_name}")
    print("Original columns:", df.columns.tolist())
    
    # Create a 'wide' DataFrame by setting the 'Area' column as the index
    df_wide_data = df.set_index(('Unnamed: 0_level_0', 'Area'))
    
    # Stack the top-level columns to get a 'long' DataFrame
    df_long_data = df_wide_data.stack(level=0)
    
    # Rename the new index levels: 'Area' and 'Quarter'
    df_long_data.index.names = ['Area', 'Quarter']
    
    # Reset the index so that 'Area' and 'Quarter' become columns
    df_reformatted = df_long_data.reset_index()
    
    # Convert columns to desired types/formats
    df_reformatted['Area'] = df_reformatted['Area'].astype(str)
    df_reformatted['Quarter'] = pd.to_datetime(df_reformatted['Quarter'], errors='coerce')
    df_reformatted['Quarter'] = df_reformatted['Quarter'].dt.strftime('%b %Y')
    df_reformatted['Count'] = pd.to_numeric(df_reformatted['Count'], errors='coerce').astype('Int64')
    df_reformatted['Median'] = pd.to_numeric(df_reformatted['Median'], errors='coerce')
    df_reformatted['Median'] = df_reformatted['Median'].apply(
        lambda x: "${:,.0f}".format(x) if pd.notnull(x) else ""
    )
    
    print(df_reformatted.head())
    print(f"Unique 'Area' values for sheet '{sheet_name}':", df_reformatted['Area'].unique().tolist())

    processed_sheets[sheet_name] = df_reformatted
```

### 2.7 Correcting Unexpected Output in the 'Area' Column
```python
# Clean the 'Area' column: Replace occurrences of "Median.1" with "Inner_Melbourne_Albert "
df_reformatted['Area'] = df_reformatted['Area'].str.replace(r'^Median\.1\s*', 'Inner_Melbourne_Albert ', regex=True)

if 'all_properties' in processed_sheets:
    print("First few rows of 'all_properties' sheet:")
    print(processed_sheets['all_properties'].head())
else:
    print("Sheet 'all_properties' not found in processed_sheets.")
```

### 2.8 Exporting the Processed Sheets
####    2.8.a Writing All Processed Sheets to a Single Multi-Tab Excel File
```python
output_filename = 'moving_annual_rent_by_suburb_september_quarter_2024_reformatted.xlsx'
with pd.ExcelWriter(output_filename) as writer:
    for sheet_name, df_reformatted in processed_sheets.items():
        df_reformatted.to_excel(writer, sheet_name=sheet_name, index=False)
print(f"\nAll processed sheets have been written to {output_filename}")
```

####    2.8.b Writing Each Processed Sheet to Separate Excel Files and Compressing into a ZIP file
```python
import os
import shutil

# Create a temporary output directory to hold individual Excel files
temp_output_dir = 'moving_annual_rent_by_suburb_september_quarter_2024_temp'
if not os.path.exists(temp_output_dir):
    os.makedirs(temp_output_dir)

# Write each processed DataFrame to its own Excel file in the temporary directory
for sheet_name, df_reformatted in processed_sheets.items():
    output_file = os.path.join(temp_output_dir, f"{sheet_name}.xlsx")
    df_reformatted.to_excel(output_file, index=False)
    print(f"Sheet '{sheet_name}' written to {output_file}")

# Define the base name for the zip file
zip_base_name = 'moving_annual_rent_by_suburb_september_quarter_2024'
zip_filename = f"{zip_base_name}.zip"

# Create a zip file from the temporary output directory
shutil.make_archive(zip_base_name, 'zip', temp_output_dir)
print(f"\nAll processed sheets have been zipped into {zip_filename}")
```

## 3. Summary
### 3.1. VBA Stage:
###   - Start with the original file and create a VBA copy (moving_annual_rent_by_suburb_september_quarter_2024_vba.xlsx).
###   - Run the provided VBA script to clean up the worksheets.
###   - Save the cleaned version as moving_annual_rent_by_suburb_september_quarter_2024_copy.xlsx.

### 3.2. Python Stage:
###   -Use the cleaned file (moving_annual_rent_by_suburb_september_quarter_2024_copy.xlsx) to read, reshape, and format the data.
###   -Export the results into a single multi-tab Excel file or individual files (with optional ZIP compression).

### Follow these steps in order to ensure that your data is correctly cleaned and processed. Please let me know if there was an easier way to achieve the same results.  
