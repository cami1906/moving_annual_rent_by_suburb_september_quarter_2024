# Integrated Workflow: Cleaning and Processing Excel Data

This document outlines a complete workflow for cleaning and processing the Excel file data. The process is split into two major parts:

1. **VBA Processing in Excel:**  
   - **Purpose:** Clean and reformat the Excel worksheets.
   - **File Flow:**  
     - **Original file:** `moving_annual_rent_by_suburb_september_quarter_2024.xlsx`  
     - **Copy for VBA cleaning:** `moving_annual_rent_by_suburb_september_quarter_2024_vba.xlsx`  
     - After cleaning, save a new copy as: `moving_annual_rent_by_suburb_september_quarter_2024_copy.xlsx`

2. **Python Processing:**  
   - **Purpose:** Read the cleaned data, reshape it, convert data types, and export the results.
   - **Input file:** `moving_annual_rent_by_suburb_september_quarter_2024_copy.xlsx`
