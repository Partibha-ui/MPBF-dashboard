# MPBF-dashboard
import streamlit as st
import pandas as pd
import plotly.graph_objects as go
from datetime import datetime

# Page config
st.set_page_config(page_title="MPBF Office D-1 Dashboard", layout="wide", initial_sidebar_state="expanded")

# Custom styling
st.markdown("""
<style>
    .metric-card { padding: 20px; border-radius: 10px; text-align: center; }
    .status-good { background-color: #d4edda; }
    .status-warning { background-color: #fff3cd; }
    .status-critical { background-color: #f8d7da; }
</style>
""", unsafe_allow_html=True)

# ==================== TITLE ====================
st.title("📊 MPBF Office D-1 Performance Dashboard")
st.markdown("**Real-time Performance Monitoring** | Period: June 1-14, 2026")

# ==================== DATA ====================
data = {
    'Date': ['1-Jun', '2-Jun', '3-Jun', '4-Jun', '5-Jun', '6-Jun', '7-Jun', 
             '8-Jun', '9-Jun', '10-Jun', '11-Jun', '12-Jun', '13-Jun', '14-Jun'],
    'Compliance': [100.0]*14,
    'Vehicles_Reported': [33, 37, 32, 33, 34, 37, 15, 40, 36, 35, 32, 42, 31, 17],
    'Boxes_Received': [7174, 9852, 7708, 7146, 8286, 9248, 8621, 8213, 
                       9499, 8309, 10363, 8841, 5769, 5769],
    'IB_Achievement': [105, 144, 113, 104, 121, 135, 126, 103, 119, 104, 72, 130, 111, 72],
    'Cross_Dock': [2147, 2943, 2079, 2213, 4887, 3339, 2918, 2591, 3488, 1413, 1458, 4243, 3184, 2889],
    'Floor_Pendency': [6426, 7429, 4973, 5445, 4334, 6334, 6087, 6209, 
                       6346, 7886, 5008, 7887, 7227, 5200],
    'OB_Pendency_24_48': [29, 11, 62, 133, 53, 30, 151, 35, 3, 9, 66, 28, 92, 270],
    'OB_Pendency_above_48': [10, 10, 14, 13, 10, 12, 11, 12, 4, 4, 3, 3, 3, 6],
    'Boxes_Dispatched': [5225, 8410, 9575, 6942, 8542, 8901, 7904, 8313, 
                         8956, 7041, 8966, 8091, 9184, 7956],
    'Breach_24_48hrs': [2, 5, 6, 7, 6, 5, 0, 16, 13, 0, 7, 1, 1, 0],
    'Breach_above_48hrs': [0, 2, 5, 4, 5, 7, 0, 5, 16, 3, 2, 2, 2, 0],
}

df = pd.DataFrame(data)

# ==================== SIDEBAR ====================
st.sidebar.title("🔧 Dashboard Controls")
st.sidebar.markdown("---")

date_filter = st.sidebar.selectbox(
    "Select Time Period",
    options=['Last 7 Days', 'Last 14 Days'],
    index=1
)

if date_filter == 'Last 7 Days':
    display_df = df.tail(7)
else:
    display_df = df

st.sidebar.markdown("---")
st.sidebar.markdown("### 📊 Data Info")
st.sidebar.info(f"Records: {len(display_df)} days\nLast Updated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
st.sidebar.markdown("Auto-refresh: Every 60 seconds")

# ==================== KEY METRICS ====================
st.subheader("📈 Key Performance Indicators (KPIs)")

today = display_df.iloc[-1]
period_avg = display_df.mean()

col1, col2, col3, col4, col5, col6 = st.columns(6)

with col1:
    comp = today['Compliance']
    st.metric(
        label="Compliance",
        value=f"{comp:.1f}%",
        delta=f"{comp - period_avg['Compliance']:.1f}%",
        delta_color="off"
    )

with col2:
    ib = today['IB_Achievement']
    delta_color = "normal" if ib >= 100 else "inverse"
    st.metric(
        label="IB Achievement",
        value=f"{ib:.0f}%",
        delta=f"{ib - period_avg['IB_Achievement']:.0f}%",
        delta_color=delta_color
    )

with col3:
    boxes = today['Boxes_Received']
    st.metric(
        label="Boxes Received",
        value=f"{int(boxes):,}",
        delta=f"{int(boxes - period_avg['Boxes_Received']):,}"
    )

with col4:
    pend = today['Floor_Pendency']
    if pend <= 5000:
        color = "normal"
    elif pend <= 7000:
        color = "off"
    else:
        color = "inverse"
    st.metric(
        label="Floor Pendency",
        value=f"{int(pend):,}",
        delta=f"{int(pend - period_avg['Floor_Pendency']):,}",
        delta_color=color
    )

with col5:
    breach = today['Breach_above_48hrs']
    st.metric(
        label="Breach >48hrs",
        value=int(breach),
        delta=int(breach - period_avg['Breach_above_48hrs']),
        delta_color="inverse"
    )

with col6:
    dispatch = today['Boxes_Dispatched']
    st.metric(
        label="Boxes Dispatched",
        value=f"{int(dispatch):,}",
        delta=f"{int(dispatch - period_avg['Boxes_Dispatched']):,}"
    )

st.markdown("---")

# ==================== CHARTS ====================
st.subheader("📊 Performance Trends & Analysis")

# Row 1: Inbound vs Outbound
col1, col2 = st.columns(2)

with col1:
    fig1 = go.Figure()
    fig1.add_trace(go.Scatter(
        x=display_df['Date'], y=display_df['Boxes_Received'],
        mode='lines+markers', name='Boxes Received',
        line=dict(color='#1f77b4', width=3),
        marker=dict(size=8)
    ))
    fig1.add_hline(y=display_df['Boxes_Received'].mean(), line_dash="dash", 
                   line_color="gray", annotation_text=f"Avg: {display_df['Boxes_Received'].mean():.0f}")
    fig1.update_layout(
        title="<b>Inbound Boxes Trend</b>",
        xaxis_title="Date",
        yaxis_title="Number of Boxes",
        hovermode='x unified',
        height=400,
        template='plotly_white'
    )
    st.plotly_chart(fig1, use_container_width=True)

with col2:
    fig2 = go.Figure()
    fig2.add_trace(go.Scatter(
        x=display_df['Date'], y=display_df['Boxes_Dispatched'],
        mode='lines+markers', name='Boxes Dispatched',
        line=dict(color='#2ca02c', width=3),
        marker=dict(size=8)
    ))
    fig2.add_hline(y=display_df['Boxes_Dispatched'].mean(), line_dash="dash",
                   line_color="gray", annotation_text=f"Avg: {display_df['Boxes_Dispatched'].mean():.0f}")
    fig2.update_layout(
        title="<b>Outbound Boxes Trend</b>",
        xaxis_title="Date",
        yaxis_title="Number of Boxes",
        hovermode='x unified',
        height=400,
        template='plotly_white'
    )
    st.plotly_chart(fig2, use_container_width=True)

# Row 2: IB Achievement & Floor Pendency
col1, col2 = st.columns(2)

with col1:
    fig3 = go.Figure()
    colors = ['#28a745' if x >= 100 else '#ffc107' if x >= 90 else '#dc3545' 
              for x in display_df['IB_Achievement']]
    fig3.add_trace(go.Bar(
        x=display_df['Date'], y=display_df['IB_Achievement'],
        marker_color=colors, name='IB Achievement %'
    ))
    fig3.add_hline(y=100, line_dash="dash", line_color="blue", 
                   annotation_text="Target: 100%")
    fig3.update_layout(
        title="<b>IB Achievement vs Target</b>",
        xaxis_title="Date",
        yaxis_title="Achievement %",
        height=400,
        template='plotly_white'
    )
    st.plotly_chart(fig3, use_container_width=True)

with col2:
    fig4 = go.Figure()
    fig4.add_trace(go.Scatter(
        x=display_df['Date'], y=display_df['Floor_Pendency'],
        mode='lines+markers', name='Floor Pendency',
        line=dict(color='#d62728', width=3),
        fill='tozeroy'
    ))
    fig4.add_hline(y=5000, line_dash="dash", line_color="green",
                   annotation_text="Ideal: 5000")
    fig4.add_hline(y=7000, line_dash="dash", line_color="red",
                   annotation_text="Alert: 7000")
    fig4.update_layout(
        title="<b>Total Floor Pendency Trend</b>",
        xaxis_title="Date",
        yaxis_title="Number of Boxes",
        height=400,
        template='plotly_white'
    )
    st.plotly_chart(fig4, use_container_width=True)

# Row 3: SLA Breaches
col1, col2, col3 = st.columns(3)

with col1:
    fig5 = go.Figure()
    fig5.add_trace(go.Bar(
        x=display_df['Date'], y=display_df['Breach_24_48hrs'],
        marker_color='#ffc107', name='24-48 hrs'
    ))
    fig5.update_layout(
        title="<b>Dockets Breach 24-48 hrs</b>",
        xaxis_title="Date",
        yaxis_title="Count",
        height=400,
        template='plotly_white'
    )
    st.plotly_chart(fig5, use_container_width=True)

with col2:
    fig6 = go.Figure()
    fig6.add_trace(go.Bar(
        x=display_df['Date'], y=display_df['Breach_above_48hrs'],
        marker_color='#dc3545', name='>48 hrs'
    ))
    fig6.update_layout(
        title="<b>Dockets Breach >48 hrs ⚠️</b>",
        xaxis_title="Date",
        yaxis_title="Count",
        height=400,
        template='plotly_white'
    )
    st.plotly_chart(fig6, use_container_width=True)

with col3:
    fig7 = go.Figure()
    fig7.add_trace(go.Scatter(
        x=display_df['Date'], y=display_df['OB_Pendency_above_48'],
        mode='lines+markers', name='OB Pendency >48hrs',
        line=dict(color='#dc3545', width=3)
    ))
    fig7.update_layout(
        title="<b>OB Pendency >48 hrs</b>",
        xaxis_title="Date",
        yaxis_title="Count",
        height=400,
        template='plotly_white'
    )
    st.plotly_chart(fig7, use_container_width=True)

st.markdown("---")

# ==================== ALERTS ====================
st.subheader("⚠️ Alerts & Recommendations")

alert_count = 0

if today['Floor_Pendency'] > 7000:
    st.error(f"🔴 **CRITICAL**: Floor pendency ({int(today['Floor_Pendency'])}) exceeds 7000 threshold!")
    alert_count += 1
elif today['Floor_Pendency'] > 6500:
    st.warning(f"🟡 **WARNING**: Floor pendency ({int(today['Floor_Pendency'])}) approaching limit (7000)")
    alert_count += 1

if today['Breach_above_48hrs'] > 5:
    st.error(f"🔴 **CRITICAL**: {int(today['Breach_above_48hrs'])} dockets breached >48 hours SLA!")
    alert_count += 1
elif today['Breach_above_48hrs'] > 0:
    st.warning(f"🟡 **WARNING**: {int(today['Breach_above_48hrs'])} dockets in SLA breach (>48 hrs)")
    alert_count += 1

if today['IB_Achievement'] < 90:
    st.warning(f"🟡 **WARNING**: IB Achievement {today['IB_Achievement']:.0f}% is below target (100%)")
    alert_count += 1

if today['Breach_24_48hrs'] > 100:
    st.warning(f"🟡 **WARNING**: {int(today['Breach_24_48hrs'])} dockets approaching 48-hour breach")
    alert_count += 1

if alert_count == 0:
    st.success("✅ **All metrics within acceptable ranges** - No critical alerts")

st.markdown("---")

# ==================== DATA TABLE ====================
st.subheader("📋 Daily Performance Summary")

table_df = display_df[[
    'Date', 'Compliance', 'Boxes_Received', 'IB_Achievement', 
    'Floor_Pendency', 'Boxes_Dispatched', 'Breach_above_48hrs'
]].copy()

table_df.columns = ['Date', 'Compliance %', 'Boxes In', 'IB Achieve %', 
                     'Floor Pendency', 'Boxes Out', 'Breaches >48hrs']

# Format for display
table_df['Boxes In'] = table_df['Boxes In'].apply(lambda x: f"{int(x):,}")
table_df['Floor Pendency'] = table_df['Floor Pendency'].apply(lambda x: f"{int(x):,}")
table_df['Boxes Out'] = table_df['Boxes Out'].apply(lambda x: f"{int(x):,}")
table_df['Compliance %'] = table_df['Compliance %'].apply(lambda x: f"{x:.1f}%")
table_df['IB Achieve %'] = table_df['IB Achieve %'].apply(lambda x: f"{int(x)}%")

st.dataframe(table_df, use_container_width=True, hide_index=True)

# ==================== SUMMARY STATS ====================
st.markdown("---")
st.subheader("📊 Period Summary")

col1, col2, col3, col4 = st.columns(4)

with col1:
    st.metric("Period Avg - Boxes Received", f"{int(display_df['Boxes_Received'].mean()):,}")
with col2:
    st.metric("Period Avg - Floor Pendency", f"{int(display_df['Floor_Pendency'].mean()):,}")
with col3:
    st.metric("Total Breaches >48hrs", f"{int(display_df['Breach_above_48hrs'].sum())}")
with col4:
    st.metric("Avg IB Achievement", f"{int(display_df['IB_Achievement'].mean())}%")

# ==================== FOOTER ====================
st.markdown("---")
st.markdown(f"""
<div style='text-align: center; color: gray; font-size: 12px;'>
📊 <b>MPBF Office D-1 Performance Dashboard</b> | 
Last Updated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')} | 
Data Source: MPBF Performance Report Card
</div>
""", unsafe_allow_html=True)
