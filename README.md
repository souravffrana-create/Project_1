import streamlit as st
import requests

# Page config
st.set_page_config(
    page_title="MailGuard",
    page_icon="🛡️",
    layout="centered"
)

# Custom CSS for dark theme
st.markdown("""
    <style>
        .stApp {
            background-color: #0f0f0f;
            color: #ffffff;
        }
        .title {
            text-align: center;
            font-size: 3rem;
            font-weight: 800;
            color: #00e5ff;
            margin-bottom: 0.2rem;
        }
        .subtitle {
            text-align: center;
            font-size: 1rem;
            color: #888888;
            margin-bottom: 2rem;
        }
        .spam-box {
            background-color: #3d0000;
            border: 2px solid #ff4444;
            border-radius: 12px;
            padding: 20px;
            text-align: center;
            font-size: 1.5rem;
            font-weight: 700;
            color: #ff4444;
        }
        .ham-box {
            background-color: #003d1a;
            border: 2px solid #00e676;
            border-radius: 12px;
            padding: 20px;
            text-align: center;
            font-size: 1.5rem;
            font-weight: 700;
            color: #00e676;
        }
        .stTextArea textarea {
            background-color: #1a1a1a !important;
            color: #ffffff !important;
            border: 1px solid #333333 !important;
            border-radius: 8px !important;
        }
        .stButton > button {
            background-color: #00e5ff;
            color: #000000;
            font-weight: 700;
            font-size: 1rem;
            border-radius: 8px;
            border: none;
            width: 100%;
            padding: 0.6rem;
        }
        .stButton > button:hover {
            background-color: #00b8d4;
            color: #000000;
        }
    </style>
""", unsafe_allow_html=True)

# Header
st.markdown('<div class="title">🛡️ MailGuard</div>', unsafe_allow_html=True)
st.markdown('<div class="subtitle">AI-powered Email Spam Classifier</div>', unsafe_allow_html=True)
st.markdown("---")

# Input
email_text = st.text_area(
    "📧 Paste your email content below:",
    height=200,
    placeholder="Enter email text here..."
)

char_count = len(email_text)
st.caption(f"Characters: {char_count}")

# Classify button
if st.button("🔍 Classify Email"):
    if not email_text.strip():
        st.warning("Please enter some email content first.")
    else:
        with st.spinner("Analyzing email..."):
            try:
                response = requests.post(
                    "http://127.0.0.1:8000/predict",
                    json={"Text": email_text}
                )

                if response.status_code == 200:
                    data = response.json()
                    result = data["Response"]
                    message = result["Message"]
                    prediction = result["Prediction"]

                    st.markdown("### Result:")

                    if message == "Spam":
                        st.markdown(
                            f'<div class="spam-box">🚨 SPAM &nbsp;|&nbsp; Prediction: {prediction}</div>',
                            unsafe_allow_html=True
                        )
                    else:
                        st.markdown(
                            f'<div class="ham-box">✅ HAM (Not Spam) &nbsp;|&nbsp; Prediction: {prediction}</div>',
                            unsafe_allow_html=True
                        )
                else:
                    st.error(f"API Error: {response.status_code}")

            except requests.exceptions.ConnectionError:
                st.error("❌ Cannot connect to FastAPI server. Make sure it's running on http://127.0.0.1:8000")
            except Exception as e:
                st.error(f"Something went wrong: {str(e)}")

# Footer
st.markdown("---")
st.markdown(
    '<p style="text-align:center; color:#555; font-size:0.8rem;">Powered by LinearSVC + TF-IDF | 98.5% Accuracy</p>',
    unsafe_allow_html=True
)
# Project_1
my first project
<br>
hii everyone
