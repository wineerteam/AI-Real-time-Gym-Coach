# 🚀 AI Gym Coach Deployment Guide (Public Hosting)

This guide takes you through the step-by-step process of hosting your project publicly for free on **Streamlit Community Cloud**.

---

## 🔒 Step 1: Push Code to GitHub

Streamlit Cloud pulls your code directly from a GitHub repository. Follow these commands to upload your workspace:

1. **Initialize Git**:
   Open your command line in the project root directory (`e:\ai-gym-coach-main\ai-gym-coach-main`) and run:
   ```bash
   git init
   ```
2. **Add Files & Commit**:
   Our updated `.gitignore` file will automatically prevent your virtual environment (`.venv/`) and your private API key (`.env`) from being uploaded.
   ```bash
   git add .
   git commit -m "Configure AI Gym Coach for deployment"
   ```
3. **Push to GitHub**:
   * Go to [GitHub](https://github.com/) and create a new repository (name it something like `ai-gym-coach`).
   * Run the commands provided by GitHub to link and push your code:
     ```bash
     git remote add origin https://github.com/your-username/ai-gym-coach.git
     git branch -M main
     git push -u origin main
     ```

---

## 🌐 Step 2: Deploy to Streamlit Community Cloud (Free)

Streamlit Community Cloud allows you to host Python Streamlit apps directly from GitHub for free.

1. **Sign Up / Log In**:
   Visit [share.streamlit.io](https://share.streamlit.io/) and log in using your GitHub account.
2. **Create a New App**:
   * Click the **New app** button on your dashboard.
   * Under **Repository**, select your newly created repository (e.g., `your-username/ai-gym-coach`).
   * Under **Branch**, select `main`.
   * Under **Main file path**, type:
     ```text
     Main App/main.py
     ```
3. **Configure API Secrets (Crucial for AI voice feedback)**:
   * Before clicking deploy, click on the **Advanced settings...** button.
   * In the **Secrets** text box, paste your Groq API key in the following TOML format:
     ```toml
     GROQ_API_KEY = "your_actual_groq_api_key_here"
     ```
   * Click **Save**.
4. **Deploy**:
   * Click **Deploy!**
   * Streamlit will compile your dependencies from `requirements.txt` and launch the app in 1-2 minutes.

---

## 📹 Note on WebRTC and Camera Permissions

* **HTTPS Requirement**: Modern web browsers (Chrome, Safari, Edge, Firefox) only allow webapps to access your webcam if the website is served over **HTTPS** (secured connection) or if it is running on **localhost**.
* **Streamlit Cloud Advantage**: Streamlit Community Cloud automatically provisions an SSL certificate and serves your application over `https://your-app.streamlit.app`.
* Therefore, your camera and WebRTC real-time pose estimation stream will work out-of-the-box for anyone visiting your live public link!
