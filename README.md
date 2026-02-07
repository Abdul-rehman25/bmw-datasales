import streamlit as st
import pandas as pd
import numpy as np
import plotly.graph_objects as go
import plotly.express as px

# Page configuration
st.set_page_config(
    page_title="BMW Sales Dashboard",
    page_icon="🚗",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Custom CSS
st.markdown("""
    <style>
    .main {
        padding-top: 2rem;
    }
    .metric-card {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        padding: 20px;
        border-radius: 10px;
        color: white;
    }
    </style>
    """, unsafe_allow_html=True)

# Load dataset
df = pd.read_csv('BMW sales data (2010-2024) (1).csv')

# Title and description
st.markdown("# 🚗 BMW Sales Dashboard (2010-2024)")
st.markdown("---")

# Sidebar filters
st.sidebar.markdown("## 📊 Filters")

# Create columns for metrics
col1, col2, col3, col4 = st.columns(4)

with col1:
    st.metric(
        label="Total Sales Records",
        value=f"{len(df):,}",
        delta="Complete Dataset"
    )

with col2:
    if 'Sales' in df.columns:
        st.metric(
            label="Total Revenue",
            value=f"${df['Sales'].sum():,.0f}M",
            delta="All Time"
        )
    else:
        st.metric(label="Data Points", value=len(df))

with col3:
    if 'Year' in df.columns:
        st.metric(
            label="Years Covered",
            value=f"{df['Year'].min()}-{df['Year'].max()}",
            delta="Time Range"
        )

with col4:
    if 'Model' in df.columns or 'model' in df.columns.str.lower():
        col_name = 'Model' if 'Model' in df.columns else [c for c in df.columns if 'model' in c.lower()][0]
        st.metric(
            label="Unique Models",
            value=f"{df[col_name].nunique()}",
            delta="Variants"
        )

st.markdown("---")

# Detect column names (handle various naming conventions)
year_col = next((col for col in df.columns if 'year' in col.lower()), None)
model_col = next((col for col in df.columns if 'model' in col.lower()), None)
sales_col = next((col for col in df.columns if 'sales' in col.lower()), None)

# Main Dashboard
col1, col2 = st.columns(2)

# Chart 1: Sales Trend Over Years
if year_col and sales_col:
    with col1:
        st.subheader("📈 Sales Trend Over Time")
        yearly_sales = df.groupby(year_col)[sales_col].sum().reset_index()
        fig1 = px.line(yearly_sales, x=year_col, y=sales_col,
                       title="Annual Sales Trend",
                       markers=True, line_shape="spline")
        fig1.update_traces(line=dict(color='#667eea', width=3))
        fig1.update_layout(hovermode='x unified', height=400)
        st.plotly_chart(fig1, use_container_width=True)

# Chart 2: Top Models
if model_col and sales_col:
    with col2:
        st.subheader("🏆 Top Selling Models")
        top_models = df.groupby(model_col)[sales_col].sum().nlargest(10).reset_index()
        fig2 = px.bar(top_models, x=sales_col, y=model_col,
                      orientation='h',
                      title="Top 10 Models by Sales",
                      color=sales_col,
                      color_continuous_scale='Viridis')
        fig2.update_layout(height=400, showlegend=False)
        st.plotly_chart(fig2, use_container_width=True)

col3, col4 = st.columns(2)

# Chart 3: Sales Distribution by Model (Pie Chart)
if model_col and sales_col:
    with col3:
        st.subheader("🥧 Market Share by Model")
        model_sales = df.groupby(model_col)[sales_col].sum().nlargest(8)
        fig3 = go.Figure(data=[go.Pie(
            labels=model_sales.index,
            values=model_sales.values,
            hole=.3
        )])
        fig3.update_layout(height=400)
        st.plotly_chart(fig3, use_container_width=True)

# Chart 4: Year-over-Year Comparison (Box Plot)
if year_col and sales_col:
    with col4:
        st.subheader("📊 Sales Distribution by Year")
        fig4 = px.box(df, x=year_col, y=sales_col,
                      title="Sales Variability by Year",
                      color=year_col)
        fig4.update_layout(height=400, showlegend=False)
        st.plotly_chart(fig4, use_container_width=True)

# Additional Analytics Section
st.markdown("---")
st.subheader("📋 Detailed Data View")

# Data table with sorting/filtering
st.dataframe(
    df,
    use_container_width=True,
    height=400
)

# Summary Statistics
st.markdown("---")
st.subheader("📈 Summary Statistics")
col1, col2 = st.columns(2)

with col1:
    st.write("**Numeric Columns Summary:**")
    st.dataframe(df.describe(), use_container_width=True)

with col2:
    st.write("**Data Info:**")
    info_text = f"""
    - **Total Records:** {len(df)}
    - **Total Columns:** {len(df.columns)}
    - **Memory Usage:** {df.memory_usage(deep=True).sum() / 1024:.2f} KB
    - **Missing Values:** {df.isnull().sum().sum()}
    """
    st.markdown(info_text)
