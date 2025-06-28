import streamlit as st
import pandas as pd
import gspread
from google.auth import default

# Page setup
st.set_page_config(page_title="CRM Enquiry Dashboard", layout="wide")
st.title("📋 CRM - Customer Enquiry Portal")

# Google Sheets auth and load
creds, _ = default()
gc = gspread.authorize(creds)

# Load data from Sheet2
sheet_url = "https://docs.google.com/spreadsheets/d/1cjdbjcvqWqV_wasrQw341gpIjUzMVGXR4DZAt1dg2wM"
worksheet = gc.open_by_url(sheet_url).worksheet("Sheet2")
data = worksheet.get_all_values()

# DataFrame
headers = data[0]
rows = data[1:]
df = pd.DataFrame(rows, columns=headers)
df.columns = [col.strip() for col in df.columns]

# Search functionality
search_query = st.text_input("🔍 Search by Name, Email, or Phone")
if search_query:
    df = df[df.apply(lambda row: search_query.lower() in row.astype(str).str.lower().to_string(), axis=1)]

st.markdown(f"### Showing {len(df)} Enquiries")

# Display each enquiry
for idx, row in df.iterrows():
    with st.container():
        cols = st.columns([1, 5])

        # Image preview (Column I)
        img_url = row.get("I", "")
        if "http" in img_url:
            cols[0].image(img_url, width=100)
        else:
            cols[0].markdown("🖼️ *No Image*")

        # Display details
        with cols[1]:
            for col_name, value in row.items():
                if col_name != "I":
                    st.markdown(f"**{col_name}**: {value}")
        st.divider()
