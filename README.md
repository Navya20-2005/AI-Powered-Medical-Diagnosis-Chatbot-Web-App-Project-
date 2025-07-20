# AI-Powered-Medical-Diagnosis-Chatbot-Web-App-Project-
import streamlit as st
import google.generativeai as genai
from PIL import Image
import io
import time

# Configure Gemini API (Replace 'YOUR_GEMINI_API_KEY' with your actual key)
genai.configure(api_key="AIzaSyCm7-EAoO2x_0rv3a4ss7BplHwWx-MUHUo")

# Custom CSS for styling
st.markdown(
    """
    <style>
    .stApp {
        background: linear-gradient(135deg, #f5f7fa, #c3cfe2);
        color: #2c3e50;
    }
    .stButton>button {
        background-color: #4CAF50;
        color: white;
        border-radius: 8px;
        padding: 10px 20px;
        border: none;
        font-size: 16px;
        transition: background-color 0.3s ease;
    }
    .stButton>button:hover {
        background-color: #45a049;
    }
    .stSidebar {
        background-color: #2c3e50;
        color: white;
    }
    .stMarkdown h1 {
        color: #4CAF50;
    }
    .stMarkdown h2 {
        color: #2c3e50;
    }
    /* Login/Signup Page Background */
    .login-background {
        background-image: url('https://images.unsplash.com/photo-1532938911079-1b06ac7ceec7');
        background-size: cover;
        background-position: center;
        padding: 2rem;
        border-radius: 10px;
        color: white;
    }
    .login-form {
        background: rgba(0, 0, 0, 0.7);
        padding: 2rem;
        border-radius: 10px;
    }
    </style>
    """,
    unsafe_allow_html=True,
)

# Simulated user database (username: password)
if "USER_DATABASE" not in st.session_state:
    st.session_state.USER_DATABASE = {
        "user1": "password1",
        "user2": "password2",
        "admin": "admin123",
    }


# Function to get available models
def get_available_models():
    try:
        models = genai.list_models()
        return [model.name for model in models]
    except Exception as e:
        return [f"Error fetching models: {e}"]


# Function to generate chatbot response
def chatbot_response(query):
    try:
        model_name = "models/gemini-1.5-pro-latest"  # Updated to a valid model
        model = genai.GenerativeModel(model_name)
        response = model.generate_content(query)
        return response.text if response else "I'm sorry, I couldn't generate a response."
    except Exception as e:
        return f"Error: {e}. Available models: {get_available_models()}"


# Function to analyze medical image
def analyze_medical_image(image_data, disease_type):
    """Analyze a medical image based on the selected disease type"""
    try:
        model_name = "models/gemini-1.5-flash-latest"  # Updated to a supported model
        model = genai.GenerativeModel(model_name)

        image = Image.open(io.BytesIO(image_data))
        prompt = f"Analyze this medical image for {disease_type} abnormalities."
        response = model.generate_content([prompt, image])

        return response.text if response else "No analysis result available."
    except Exception as e:
        return f"Error analyzing image: {e}. Available models: {get_available_models()}"


# Initialize session state for page navigation and authentication
if "page" not in st.session_state:
    st.session_state.page = "Login/Signup"  # Default to Login/Signup page
if "logged_in" not in st.session_state:
    st.session_state.logged_in = False

# Sidebar for navigation (only show if logged in)
if st.session_state.logged_in:
    st.sidebar.title("🚀 Navigation")
    if st.sidebar.button("🤖 Home"):
        st.session_state.page = "Home"
    if st.sidebar.button("💬 Chat with AI"):
        st.session_state.page = "Chat with AI"
    if st.sidebar.button("🖼 Image Analysis"):
        st.session_state.page = "Image Analysis"
    if st.sidebar.button("⚙ Settings"):
        st.session_state.page = "Settings"
    if st.sidebar.button("🔐 Logout"):
        st.session_state.logged_in = False
        st.session_state.page = "Login/Signup"
        st.rerun()

# Main content based on the selected page
if st.session_state.page == "Login/Signup":
    st.title("🔐 Login / Signup")
    st.markdown(
        """
        <div class="login-background">
            <div class="login-form">
                <h2 style="color: white;">Welcome Back!</h2>
                <p style="color: white;">Please log in or sign up to continue.</p>
            </div>
        </div>
        """,
        unsafe_allow_html=True,
    )


    # Function to authenticate user
    def authenticate(username, password):
        # Trim whitespace from inputs
        username = username.strip()
        password = password.strip()

        # Debugging: Print inputs and database
        print(f"Username entered: '{username}'")
        print(f"Password entered: '{password}'")
        print(f"Database: {st.session_state.USER_DATABASE}")

        # Check if username exists and password matches
        if username in st.session_state.USER_DATABASE:
            if st.session_state.USER_DATABASE[username] == password:
                return True  # Authentication successful
            else:
                print("Password does not match.")  # Debugging
        else:
            print("Username not found.")  # Debugging
        return False  # Authentication failed


    # Function to add new users
    def add_user(username, password):
        # Trim whitespace from inputs
        username = username.strip()
        password = password.strip()

        if username in st.session_state.USER_DATABASE:
            return False  # User already exists
        st.session_state.USER_DATABASE[username] = password
        return True  # Signup successful


    # Authentication UI
    st.title("🔐 Login / Signup")
    with st.form("auth_form"):
        username = st.text_input("Username")
        password = st.text_input("Password", type="password")
        auth_action = st.radio("Choose Action", ["Login", "Signup"])
        submitted = st.form_submit_button("Submit")

        if submitted:
            if auth_action == "Login":
                if authenticate(username, password):
                    st.session_state.logged_in = True
                    st.session_state.page = "Home"  # Navigate to Home Page
                    st.success(f"Welcome, {username}!")
                    st.rerun()  # Rerun the app to reflect the changes
                else:
                    st.error("Invalid username or password.")
            elif auth_action == "Signup":
                if username and password:
                    if add_user(username, password):
                        st.success("Signup successful! Please log in.")
                    else:
                        st.error("Username already exists.")
                else:
                    st.error("Please enter a valid username and password.")

# Only show the following pages if logged in
if st.session_state.logged_in:
    if st.session_state.page == "Home":
        st.title("🤖 Welcome to the AI-Powered Medical Diagnosis Chatbot!")
        st.markdown("""
        <div style="text-align: center;">
            <h2>🌟 Your Health, Our Priority 🌟</h2>
            <p>Use the sidebar to navigate to different sections.</p>
            <p>✨ <strong>Features:</strong></p>
            <ul style="list-style-type: none; padding: 0;">
                <li>💬 Chat with an AI Medical Assistant</li>
                <li>🖼 Analyze Medical Images</li>
                <li>⚙ Customize Settings</li>
            </ul>
        </div>
        """, unsafe_allow_html=True)

    elif st.session_state.page == "Chat with AI":
        st.title("💬 Chat with the AI Medical Assistant")
        user_query = st.text_input("Enter your medical question:")
        if user_query:
            with st.spinner("🤖 Thinking..."):
                response = chatbot_response(user_query)
            st.write("AI Response:", response)

    elif st.session_state.page == "Image Analysis":
        st.title("🖼 Upload Medical Image for Analysis")

        # Disease selection dropdown
        disease_type = st.selectbox(
            "Select the type of disease or body part to analyze:",
            ["Eye", "Bone", "Brain", "Chest", "Skin"]
        )

        uploaded_file = st.file_uploader("Upload a medical image (X-ray, CT scan, MRI, eye image)",
                                         type=["jpg", "png", "jpeg"])

        if uploaded_file:
            st.image(uploaded_file, caption="Uploaded Image", use_container_width=True)

            # Convert uploaded file to bytes
            image_bytes = uploaded_file.getvalue()

            # Show a progress bar during analysis
            with st.spinner("🔍 Analyzing image..."):
                progress_bar = st.progress(0)
                for percent_complete in range(100):
                    time.sleep(0.01)  # Simulate analysis time
                    progress_bar.progress(percent_complete + 1)

                # Analyze the image based on the selected disease type
                analysis_result = analyze_medical_image(image_bytes, disease_type)
                st.write("AI Analysis Result:", analysis_result)

                # Feedback Section
                st.subheader("📝 Feedback")
                feedback = st.radio(
                    "Was this analysis helpful?",
                    options=["👍 Yes", "👎 No"]
                )
                if feedback:
                    st.write("Thank you for your feedback!")

                # Download Report Section
                st.subheader("📥 Download Report")
                report_content = f"""
                Disease Type: {disease_type}
                Analysis Result:
                {analysis_result}
                """
                st.download_button(
                    label="Download Report as TXT",
                    data=report_content,
                    file_name=f"{disease_type}_analysis_report.txt",
                    mime="text/plain"
                )

    elif st.session_state.page == "Settings":
        st.title("⚙ Settings")
        st.markdown("### 🎨 Customize Your Experience")

        # Theme Selection
        st.subheader("🌙 Theme")
        theme = st.selectbox(
            "Choose a theme:",
            ["Light 🌞", "Dark 🌚", "Blue Ocean 🌊"]
        )
        st.write(f"Selected Theme: {theme}")

        # Notification Preferences
        st.subheader("🔔 Notifications")
        email_notifications = st.checkbox("📧 Enable Email Notifications")
        push_notifications = st.checkbox("📱 Enable Push Notifications")
        if email_notifications or push_notifications:
            st.success("Notification preferences saved!")

        # Language Preferences
        st.subheader("🌐 Language")
        language = st.selectbox(
            "Choose your preferred language:",
            ["English 🇺🇸", "Spanish 🇪🇸", "French 🇫🇷", "German 🇩🇪"]
        )
        st.write(f"Selected Language: {language}")

        # Data Privacy
        st.subheader("🔒 Data Privacy")
        st.write("Manage your data privacy settings.")
        if st.button("🗑 Delete My Data"):
            st.warning("Are you sure you want to delete all your data? This action cannot be undone.")
            if st.button("✅ Confirm Deletion"):
                st.error("All data has been deleted.")

        # Feedback and Support
        st.subheader("📝 Feedback & Support")
        feedback = st.text_area("Share your feedback or report an issue:")
        if st.button("📤 Submit Feedback"):
            st.success("Thank you for your feedback!")

# Display login status in the sidebar
if st.session_state.logged_in:
    st.sidebar.write("✅ Logged in as User")
else:
    st.sidebar.write("🔒 Not logged in")
