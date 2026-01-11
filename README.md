# ==============================
# BUSINESS INTELLIGENCE DASHBOARD
# Deep-Dive, Interactive, Automated
# ==============================

import streamlit as st
import pandas as pd
import plotly.express as px
import numpy as np

# ------------------------------
# PAGE CONFIGURATION
# ------------------------------
st.set_page_config(
    page_title="Executive Business Intelligence Dashboard",
    layout="wide"
)

# ------------------------------
# DATA GENERATION (AUTOMATED)
# Replace this with database / API later
# ------------------------------
@st.cache_data
def generate_data():
    np.random.seed(42)
    dates = pd.date_range("2023-01-01", periods=18, freq="M")
    regions = ["North", "South", "East", "West"]
    products = ["Product A", "Product B", "Product C"]

    data = []
    for d in dates:
        for r in regions:
            for p in products:
                revenue = np.random.randint(15000, 50000)
                cost = revenue * np.random.uniform(0.55, 0.8)
                units = np.random.randint(200, 800)

                data.append([
                    d, r, p,
                    revenue,
                    cost,
                    units
                ])

    df = pd.DataFrame(
        data,
        columns=["date", "region", "product", "revenue", "cost", "units_sold"]
    )
    return df

df = generate_data()

# ------------------------------
# FEATURE ENGINEERING
# ------------------------------
df["profit"] = df["revenue"] - df["cost"]
df["margin"] = df["profit"] / df["revenue"]
df["month"] = df["date"].dt.strftime("%Y-%m")

# ------------------------------
# SIDEBAR FILTERS (MULTI-FACET)
# ------------------------------
st.sidebar.title("🔎 Business Filters")

selected_regions = st.sidebar.multiselect(
    "Select Region(s)",
    options=df["region"].unique(),
    default=df["region"].unique()
)

selected_products = st.sidebar.multiselect(
    "Select Product(s)",
    options=df["product"].unique(),
    default=df["product"].unique()
)

filtered_df = df[
    (df["region"].isin(selected_regions)) &
    (df["product"].isin(selected_products))
]

# ------------------------------
# EXECUTIVE KPI SUMMARY
# ------------------------------
st.title("📊 Executive Business Performance Dashboard")

kpi1, kpi2, kpi3, kpi4 = st.columns(4)

kpi1.metric(
    "Total Revenue",
    f"${filtered_df['revenue'].sum():,.0f}"
)

kpi2.metric(
    "Total Profit",
    f"${filtered_df['profit'].sum():,.0f}"
)

kpi3.metric(
    "Average Margin",
    f"{filtered_df['margin'].mean():.1%}"
)

kpi4.metric(
    "Units Sold",
    f"{filtered_df['units_sold'].sum():,.0f}"
)

# ------------------------------
# TIME-BASED DEEP DIVE
# ------------------------------
st.subheader("📈 Revenue & Profit Trend Analysis")

trend_df = (
    filtered_df
    .groupby("month", as_index=False)
    .agg(
        revenue=("revenue", "sum"),
        profit=("profit", "sum")
    )
)

fig_trend = px.line(
    trend_df,
    x="month",
    y=["revenue", "profit"],
    markers=True,
    title="Revenue vs Profit Over Time"
)

st.plotly_chart(fig_trend, use_container_width=True)

# ------------------------------
# DIMENSIONAL BREAKDOWN
# ------------------------------
col1, col2 = st.columns(2)

# Profit by Region
region_df = (
    filtered_df
    .groupby("region", as_index=False)
    .agg(profit=("profit", "sum"))
)

fig_region = px.bar(
    region_df,
    x="region",
    y="profit",
    color="region",
    title="Profit Contribution by Region"
)

col1.plotly_chart(fig_region, use_container_width=True)

# Profit by Product
product_df = (
    filtered_df
    .groupby("product", as_index=False)
    .agg(profit=("profit", "sum"))
)

fig_product = px.bar(
    product_df,
    x="product",
    y="profit",
    color="product",
    title="Profit Contribution by Product"
)

col2.plotly_chart(fig_product, use_container_width=True)

# ------------------------------
# MARGIN & EFFICIENCY ANALYSIS
# ------------------------------
st.subheader("📉 Margin & Efficiency Analysis")

fig_margin = px.box(
    filtered_df,
    x="product",
    y="margin",
    color="product",
    title="Margin Distribution by Product"
)

st.plotly_chart(fig_margin, use_container_width=True)

# ------------------------------
# ROOT-CAUSE DATA EXPLORER
# ------------------------------
st.subheader("🔬 Detailed Transaction-Level View")

st.dataframe(
    filtered_df.sort_values("profit", ascending=False),
    use_container_width=True
)

# ------------------------------
# BUSINESS INSIGHTS (AUTO)
# ------------------------------
st.subheader("💡 Automated Business Insights")

best_region = region_df.sort_values("profit", ascending=False).iloc[0]["region"]
worst_product = product_df.sort_values("profit").iloc[0]["product"]

st.success(f"✔ **Top Performing Region:** {best_region}")
st.warning(f"⚠ **Lowest Performing Product:** {worst_product}")
st.info("ℹ Consider reallocating investments toward high-margin regions/products.")

# ------------------------------
# FOOTER
# ------------------------------
st.caption("Automated | Interactive | Scalable | Executive-Ready")
