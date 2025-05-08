# -import streamlit as st
import pandas as pd
import os
from datetime import date
import plotly.express as px

# 資料檔案路徑
DATA_FILE = "data.csv"

# 初始化 CSV
if not os.path.exists(DATA_FILE):
    df_init = pd.DataFrame(columns=["日期", "類別", "金額", "備註"])
    df_init.to_csv(DATA_FILE, index=False)

# 載入資料
df = pd.read_csv(DATA_FILE)

st.title("📒 每日花費記帳與消費分析")

# 輸入區
st.header("新增一筆花費")
with st.form("expense_form"):
    date_input = st.date_input("日期", value=date.today())
    category = st.selectbox("類別", ["飲食", "交通", "娛樂", "生活", "其他"])
    amount = st.number_input("金額", min_value=0.0, step=1.0)
    note = st.text_input("備註")
    submitted = st.form_submit_button("新增")
    if submitted:
        new_data = pd.DataFrame([[date_input, category, amount, note]], columns=df.columns)
        new_data.to_csv(DATA_FILE, mode='a', header=False, index=False)
        st.success("花費已新增 ✅")
        st.experimental_rerun()

# 顯示紀錄
st.header("📊 消費紀錄與分析")

if not df.empty:
    df["日期"] = pd.to_datetime(df["日期"])
    df = df.sort_values(by="日期", ascending=False)
    
    st.subheader("💵 最近花費紀錄")
    st.dataframe(df.tail(10))

    # 繪製圓餅圖
    st.subheader("📈 分類支出比例")
    fig1 = px.pie(df, names="類別", values="金額", title="支出分布")
    st.plotly_chart(fig1)

    # 趨勢圖
    st.subheader("📅 每日花費趨勢")
    daily = df.groupby("日期")["金額"].sum().reset_index()
    fig2 = px.line(daily, x="日期", y="金額", markers=True, title="每日支出趨勢")
    st.plotly_chart(fig2)
else:
    st.info("尚未有任何記錄。請新增一筆花費！")
