<img width="1901" height="912" alt="Screenshot 2025-12-04 151836" src="https://github.com/user-attachments/assets/4ffd0e6c-a542-442d-bd28-9cb3015f0b06" />
<img width="1906" height="897" alt="Screenshot 2025-12-04 151822" src="https://github.com/user-attachments/assets/375b5c95-56c6-42d4-a04d-7471ed56fe51" />
import streamlit as st
import pandas as pd
import numpy as np
import random
from datetime import datetime

# 1. Page Configuration
st.set_page_config(
    page_title="HooLiv Market Intel",
    page_icon="🏙️",
    layout="wide"
)

# 2. Sidebar & Title
st.title("HooLiv × Market Intelligence (Simulation Engine)")
st.caption(f"Last update: {datetime.now().strftime('%b %d, %H:%M')} | Status: Running Simulation")

# Sidebar Inputs
st.sidebar.header("Simulation Parameters")
city = st.sidebar.text_input("Target City", "Indore").strip().title()
budget = st.sidebar.slider("Max Budget (₹)", 5000, 30000, 15000)

# Professional explanation of the simulation
st.sidebar.info(
    """
    **ℹ️ Data Mode: Synthetic Simulation**
    
    This dashboard is running in **Simulation Mode** to demonstrate the analytics pipeline. 
    
    It generates realistic market scenarios mathematically to bypass API latency and costs.
    """
)

# 3. The "Safe" Data Generator (Simulation)
def get_safe_data(city_name):
    # Seed random so the data stays consistent for the same city
    # This simulates a "stable" database
    random.seed(len(city_name)) 
    
    locations = [
        f"{city_name} Central", f"North {city_name}", f"{city_name} IT Park", 
        f"Old {city_name}", f"{city_name} University Area", "Market Road"
    ]
    
    types = ["Single Room", "Double Sharing", "Triple Sharing", "1 BHK Flat"]
    amenities_list = ["Wifi, AC", "Wifi, Food", "No Food", "Full Furnished", "Standard"]
    
    data = []
    # Generate 50 realistic listings
    for i in range(50):
        loc = random.choice(locations)
        typ = random.choice(types)
        
        # Logic: Mumbai/Bangalore are more expensive than Indore
        base_price = 4000 if city_name in ["Indore", "Bhopal"] else 8000
        price = base_price + random.randint(1000, 15000)
        
        entry = {
            "Property Name": f"HooLiv Stays - {random.choice(['Alpha', 'Beta', 'Gamma', 'Delta'])} {i}",
            "Type": typ,
            "Location": loc,
            "Price (₹)": price,
            "Amenities": random.choice(amenities_list),
            "Rating": round(random.uniform(3.5, 5.0), 1),
            "Availability": random.choice(["Immediate", "1 Week", "1 Month"])
        }
        data.append(entry)
    
    return pd.DataFrame(data)

# 4. Load Data
with st.spinner(f"Simulating market data for {city}..."):
    df = get_safe_data(city)

# 5. Filter by Budget
df_filtered = df[df["Price (₹)"] <= budget]

# 6. KPI Metrics
col1, col2, col3, col4 = st.columns(4)
with col1:
    st.metric("Properties Scanned", f"{len(df_filtered)}")
with col2:
    st.metric("Avg Market Rent", f"₹{int(df['Price (₹)'].mean())}")
with col3:
    st.metric("Region", city)
with col4:
    st.metric("Data Source", "Synthetic Engine", delta="Active", delta_color="normal")

# 7. Safe Visualizations (No crashing charts)
st.markdown("---")

c1, c2 = st.columns([2, 1])

with c1:
    st.subheader(f"Rent Overview in {city}")
    # This TABLE with bars replaces the chart that was crashing Python 3.14
    # It looks like a chart but is actually a safe table
    st.dataframe(
        df_filtered[["Location", "Price (₹)", "Type"]]
        .sort_values("Price (₹)")
        .style.bar(subset=["Price (₹)"], color='#00CC96'),
        use_container_width=True
    )

with c2:
    st.subheader("Room Types")
    type_counts = df_filtered["Type"].value_counts()
    st.write(type_counts)

# 8. Detailed Data Table
st.subheader("Top Recommended Listings")
st.dataframe(
    df_filtered.sort_values(by="Rating", ascending=False).style.background_gradient(cmap="Greens", subset=["Rating"]),
    use_container_width=True
)

# 9. Success Message
st.success(f"Simulation Complete: Market intelligence generated for {city}!")

# --- DOWNLOAD BUTTON ---
csv = df_filtered.to_csv(index=False).encode('utf-8')
st.download_button(
    label="Download Simulation Report (CSV)",
    data=csv,
    file_name=f'HooLiv_Simulation_{city}.csv',
    mime='text/csv',

)
