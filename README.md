[create_table.sql](https://github.com/user-attachments/files/24706293/create_table.sql)

import mysql.connector

def get_connection():
[db_connection.py](https://github.com/user-attachments/files/24706294/db_connection.py)[client_page.py](https://github.com/user-attachments/files/24706301/client_page.py)
[client.sql](https://github.com/user-attachments/files/24706300/client.sql)
[app.py](https://github.com/user-attachments/files/24706298/app.py)
[register.py](https://github.com/user-attachments/files/24706297/register.py)
[query_form.py](https://github.com/user-attachments/files/24706296/query_form.py)
[login.py](https://github.com/user-attachments/files/24706295/login.py)
return mysql.connector.connect(
        host="localhost",
        user="root",
        password="Hema@2528",
        database="client_query_db"
        
        )
import streamlit as st
from db_connection import get_connection

def show_login():
    st.subheader("Login")

    username = st.text_input("Username")
    password = st.text_input("Password", type="password")

    if st.button("Login"):
        conn = get_connection()
        cursor = conn.cursor(dictionary=True)

        cursor.execute(
            "SELECT * FROM users WHERE username=%s AND password=%s",
            (username, password)
        )
        user = cursor.fetchone()
        conn.close()

        if user:
            st.session_state["logged_in"] = True
            st.session_state["username"] = user["username"]
            st.session_state["role"] = user["role"]
            st.success("Login successful")
            st.rerun()
        else:
            st.error("Invalid credentials")
import streamlit as st
from db_connection import get_connection
from datetime import datetime

def show_query_form():
    st.subheader("Submit Client Query")

    email = st.text_input("Email ID")
    mobile = st.text_input("Mobile Number")
    heading = st.text_input("Query Heading")
    description = st.text_area("Query Description")

    if st.button("Submit Query"):
        conn = get_connection()
        cursor = conn.cursor()

        cursor.execute(
            """
            INSERT INTO queries
            (mail_id, mobile_number, query_heading, query_description, status, created_time)
            VALUES (%s, %s, %s, %s, %s, %s)
            """,
            (email, mobile, heading, description, "Open", datetime.now())
        )

        conn.commit()
        conn.close()
        st.success("Query submitted successfully")
import streamlit as st
from db_connection import get_connection

def show_register():
    st.subheader("Register")

    username = st.text_input("Username")
    password = st.text_input("Password", type="password")
    role = st.selectbox("Role", ["Client", "Support"])

    if st.button("Register"):
        if username == "" or password == "":
            st.error("All fields required")
            return

        conn = get_connection()
        cursor = conn.cursor()

        try:
            cursor.execute(
                "INSERT INTO users (username, password, role) VALUES (%s, %s, %s)",
                (username, password, role)
            )
            conn.commit()
            st.success("Registration successful. Please login.")
        except:
            st.error("Username already exists")

        conn.close()
import streamlit as st
import base64

import register
import login
import query_form
import admin_page


# ---------------- BACKGROUND IMAGE ----------------
def set_bg(image_file):
    with open(image_file, "rb") as img:
        encoded = base64.b64encode(img.read()).decode()

    st.markdown(
        f"""
        <style>
        .stApp {{
            background-image: url("data:image/jpg;base64,{encoded}");
            background-size: cover;
            background-position: center;
        }}
        </style>
        """,
        unsafe_allow_html=True
    )


# ---------------- PAGE SETUP ----------------
st.set_page_config(page_title="Client Query Management", layout="wide")

set_bg("images/bg.jpg")

st.markdown(
    "<h1 style='text-align:center;'>Client Query Management System</h1>",
    unsafe_allow_html=True
)


# ---------------- SESSION DEFAULTS ----------------
if "logged_in" not in st.session_state:
    st.session_state["logged_in"] = False

if "role" not in st.session_state:
    st.session_state["role"] = None


# ---------------- NOT LOGGED IN ----------------
if not st.session_state["logged_in"]:

    choice = st.radio(
        "Select Option",
        ["Register", "Login"],
        horizontal=True
    )

    if choice == "Register":
        register.show_register()

    elif choice == "Login":
        login.show_login()


# ---------------- LOGGED IN ----------------
else:
    st.success(f"Logged in as {st.session_state['role']}")

    if st.session_state["role"] == "Support":
        admin_page.show_admin_page()
    else:
        query_form.show_query_form()

    if st.button("Logout"):
        st.session_state.clear()
        st.rerun()

CREATE DATABASE IF NOT EXISTS client_query_db;
SHOW DATABASES;
USE client_query_db;

CREATE TABLE IF NOT EXISTS queries(
query_id INT AUTO_INCREMENT PRIMARY KEY,
    mail_id VARCHAR(100),
    mobile_number VARCHAR(15),
    query_heading VARCHAR(200),
    query_description TEXT,
    status VARCHAR(20),
    created_time DATETIME,
    query_closed_time DATETIME
);


CREATE TABLE IF NOT EXISTS users(
	user_id INT AUTO_INCREMENT PRIMARY KEY,
	username VARCHAR(50) UNIQUE NOT NULL,
	password VARCHAR(50) NOT NULL,
	role VARCHAR(20) NOT NULL
);
USE client_query_db;
SHOW TABLES;

USE client_query_db;
SELECT * from queries;
import streamlit as st
import pandas as pd
from db_connection import get_connection
from datetime import datetime

def show_admin_page():
    st.subheader("Admin Dashboard")

    conn = get_connection()

    df = pd.read_sql("SELECT * FROM queries", conn)

    st.dataframe(df)

    st.markdown("### Update Query Status")

    query_id = st.number_input("Query ID", step=1)

    new_status = st.selectbox(
        "Status",
        ["Open", "In Progress", "Closed"]
    )

    if st.button("Update Status"):
        cursor = conn.cursor()

        if new_status == "Closed":
            cursor.execute(
                """
                UPDATE queries
                SET status=%s, query_closed_time=%s
                WHERE query_id=%s
                """,
                (new_status, datetime.now(), query_id)
            )
        else:
            cursor.execute(
                """
                UPDATE queries
                SET status=%s
                WHERE query_id=%s
                """,
                (new_status, query_id)
            )

        conn.commit()
        st.success("Query status updated")

    conn.close()
