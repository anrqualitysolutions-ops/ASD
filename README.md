import streamlit as st
import pandas as pd
import re
import os

st.set_page_config(page_title="PDF Data Extractor", layout="wide")

st.title("🧪 Lab Data to Sheet Converter")
st.write("Upload your PDF or enter the file paths to generate an organized sheet.")

# Input: Since the data is in the file path, we allow path input
path_input = st.text_area("Enter File Paths (one per line):", 
    placeholder=r"D:\Harihar Pharmachem Pvt. Ltd\2026\Jan-26\Data\050126_RCMH_B-156_CRS_05.dat")

if st.button("Generate Sheet"):
    if path_input:
        lines = path_input.split('\n')
        data_list = []

        for path in lines:
            if not path.strip(): continue
            
            try:
                # 1. Extract Party Name (between D:\ and the next \)
                party_match = re.search(r'[A-Z]:\\([^\\]+)', path)
                party_name = party_match.group(1) if party_match else "Unknown"

                # 2. Extract Filename from the end of the path
                filename = os.path.basename(path)
                
                # 3. Parse filename: 050126_RCMH_B-156_CRS_05.dat
                # Splitting by underscores
                parts = filename.split('_')
                
                if len(parts) >= 3:
                    date_raw = parts[0]
                    # Format date from 050126 to 05-01-26
                    formatted_date = f"{date_raw[:2]}-{date_raw[2:4]}-{date_raw[4:]}"
                    
                    sample_id = parts[1] + "_" + parts[2] # RCMH_B-156
                    test_parameter = parts[3] # CRS
                    
                    data_list.append({
                        "Date": formatted_date,
                        "Party Name": party_name,
                        "Sample ID": sample_id,
                        "Test Parameter": test_parameter,
                        "Full Path": path
                    })
            except Exception as e:
                st.error(f"Error processing path: {path} - {e}")

        df = pd.DataFrame(data_list)
        st.dataframe(df)

        # Download Button
        csv = df.to_csv(index=False).encode('utf-8')
        st.download_button("Download Excel/CSV Sheet", csv, "Analyzed_Data.csv", "text/csv")
    else:
        st.warning("Please enter at least one file path.")
