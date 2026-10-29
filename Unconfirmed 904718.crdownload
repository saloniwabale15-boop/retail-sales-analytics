import streamlit as st
import pandas as pd
import plotly.express as px

st.set_page_config(
    page_title="Retail Sales Analytics",
    page_icon="📊",
    layout="wide"
)

st.title("Retail Sales Analytics & Forecasting Dashboard")
st.write("Analyze revenue, products, stores, and customer behavior.")

# Load dataset
df = pd.read_csv("Retail_Sales_Data_Unlox_(1).csv")
df.columns = df.columns.str.strip()

df["Date"] = pd.to_datetime(df["Date"], errors="coerce")

for col in ["Revenue", "Total_Sales", "Units_Sold",
            "Discount_Percentage"]:
    df[col] = pd.to_numeric(df[col], errors="coerce")

df = df.dropna(subset=["Date", "Revenue"])

# Sidebar filters
st.sidebar.header("Dashboard Filters")

min_date = df["Date"].min().date()
max_date = df["Date"].max().date()

date_range = st.sidebar.date_input(
    "Select date range",
    value=(min_date, max_date),
    min_value=min_date,
    max_value=max_date
)

if isinstance(date_range, (tuple, list)) and len(date_range) == 2:
    start_date, end_date = date_range
    filtered = df[
        df["Date"].dt.date.between(start_date, end_date)
    ]
else:
    filtered = df.copy()

locations = sorted(filtered["Store_Location"].dropna().unique())
selected_locations = st.sidebar.multiselect(
    "Store locations",
    locations,
    default=locations
)

filtered = filtered[
    filtered["Store_Location"].isin(selected_locations)
]

categories = sorted(filtered["Product_Category"].dropna().unique())
selected_categories = st.sidebar.multiselect(
    "Product categories",
    categories,
    default=categories
)

filtered = filtered[
    filtered["Product_Category"].isin(selected_categories)
]

# KPI cards
revenue = filtered["Revenue"].sum()
units = filtered["Units_Sold"].sum()
stores = filtered["Store_ID"].nunique()
avg_revenue = filtered["Revenue"].mean()

c1, c2, c3, c4 = st.columns(4)

c1.metric("Total Revenue", f"{revenue:,.2f}")
c2.metric("Units Sold", f"{units:,.0f}")
c3.metric("Stores", f"{stores:,}")
c4.metric("Average Revenue / Record", f"{avg_revenue:,.2f}")

# Monthly revenue
st.subheader("Monthly Revenue Trend")

monthly = (
    filtered.set_index("Date")["Revenue"]
    .resample("MS").sum()
    .reset_index()
)

fig_monthly = px.line(
    monthly, x="Date", y="Revenue",
    markers=True, title="Revenue by Month"
)
st.plotly_chart(fig_monthly, use_container_width=True)

# Category analysis
left, right = st.columns(2)

with left:
    st.subheader("Revenue by Product Category")
    category_sales = (
        filtered.groupby("Product_Category", as_index=False)
        ["Revenue"].sum()
        .sort_values("Revenue", ascending=False)
    )
    fig_cat = px.bar(
        category_sales, x="Product_Category", y="Revenue"
    )
    st.plotly_chart(fig_cat, use_container_width=True)

with right:
    st.subheader("Top 10 Stores")
    top_stores = (
        filtered.groupby("Store_ID", as_index=False)
        ["Revenue"].sum()
        .nlargest(10, "Revenue")
    )
    fig_store = px.bar(
        top_stores, x="Store_ID", y="Revenue"
    )
    st.plotly_chart(fig_store, use_container_width=True)

# Promotion analysis
st.subheader("Promotion Performance")

promo = (
    filtered.groupby("Promotion_Applied", as_index=False)
    .agg(
        Revenue=("Revenue", "sum"),
        Average_Revenue=("Revenue", "mean"),
        Records=("Revenue", "count")
    )
)

st.dataframe(promo, use_container_width=True)

# Raw data
with st.expander("View filtered sales data"):
    st.dataframe(filtered, use_container_width=True)
