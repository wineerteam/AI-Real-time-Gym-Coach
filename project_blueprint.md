# 🏋️‍♂️ AI Real-time GYM Coach - Blueprint & Roadmap

This document outlines **how to use** the application, provides a **color-coded architecture block diagram**, lists **current features**, and highlights the **future scope & roadmap** for production-level development.

---

## 📖 How to Use the Application (Step-by-Step)

Follow these steps to operate the app locally:

1. **User Onboarding**:
   * Open the app. You will see the **Login Wall**.
   * Enter a unique username (e.g., `princekhunt`) and click **Start Session**.
2. **Configure Workout Plan**:
   * Go to the **Sidebar** (`Apna AI Coach`).
   * Select your exercise (e.g., `Squats`, `Push-ups`, `Lunges`).
   * Enter the number of **Sets** and **Reps per Set**.
   * Click the **Start Workout** button.
3. **Camera & AI Coaching**:
   * Grant webcam access when prompted.
   * Click **Start** on the WebRTC camera widget to start streaming.
   * Reposition your body so you are fully visible in the camera frame.
   * Start performing the selected exercise. The AI coach will count your reps and speak live postural corrections (e.g., *"Keep your back straight!"*).
4. **Workout Summary**:
   * Once you finish your target sets, click **End Workout**.
   * Scroll down to view the **Workout History** table summarizing your exercise, total reps, sets completed, duration (sec), and date.

---

## 🎨 System Flow & Block Diagram

Here is the system architecture. Nodes are color-coded based on their functional roles:
* 🟢 **Green (Input/Client Side)**: Captures user interaction and webcam video.
* 🔵 **Blue (Core Posture Engine)**: Calculates joints, coordinates, and angles.
* 🟣 **Purple (Persistence)**: Saves history to the database.
* 🟡 **Yellow (AI Brain)**: Generates coaching cues.
* 🔴 **Orange (Output)**: Speaks and displays updates.

```mermaid
graph TD
    %% Define Styles
    classDef inputNode fill:#e1f5fe,stroke:#03a9f4,stroke-width:2px,color:#000;
    classDef engineNode fill:#e8f5e9,stroke:#4caf50,stroke-width:2px,color:#000;
    classDef dbNode fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px,color:#000;
    classDef aiNode fill:#fffde7,stroke:#fbc02d,stroke-width:2px,color:#000;
    classDef outputNode fill:#fbe9e7,stroke:#ff5722,stroke-width:2px,color:#000;

    %% Nodes
    A[Webcam Video Stream]:::inputNode
    B[streamlit-webrtc Iframe]:::inputNode
    C[OpenCV Frame Parser]:::engineNode
    D[MediaPipe Pose Landmarker]:::engineNode
    E[Exercise Joint Calculators]:::engineNode
    F[(SQLite DB - data.db)]:::dbNode
    G[Groq Llama 3.3 API]:::aiNode
    H[gTTS Speech Converter]:::outputNode
    I[Hidden HTML5 Autoplay Player]:::outputNode
    J[Streamlit Stats Dashboard]:::outputNode

    %% Flow
    A --> B
    B -->|Live Frames| C
    C -->|RGB Matrix| D
    D -->|33 Skeleton Joints| E
    E -->|Reps & Sets completed| F
    E -->|Postural Errors / Events| G
    G -->|Text Cue| H
    H -->|MP3 Bytes| I
    E -->|Stats Sync| J

    %% Apply Classes
```

---

## 🌟 Current Features

| Feature | Description | Business Value |
| :--- | :--- | :--- |
| **Real-time Skeleton Tracking** | Detects 33 human joint coordinates dynamically. | Users receive visual confirmation of tracking status. |
| **Multi-exercise Support** | Tracks Squats, Push-ups, Biceps Curls, Shoulder Presses, and Lunges. | Provides a versatile routine for full-body workouts. |
| **State-Machine Counter** | Counts repetitions accurately by detecting state changes (e.g., going UP vs DOWN). | Users don't have to keep count, letting them focus on form. |
| **Postural Error Correction** | Identifies knee bends, hip sags, elbow drifts, and torso swings. | Reduces the risk of workout-related injuries. |
| **AI Voice Coach** | Speaks corrective advice aloud in real-time. | Emulates a real personal trainer's presence. |
| **Workout Logs** | Logs complete stats locally on a SQLite table. | Helps users track history and incremental progress. |

---

## 🚀 Future Scope & Enhancements (Roadmap)

To elevate this project to a production-grade startup application, the following features can be added:

### 1. Advanced Authentication & Cloud Sync
* **What**: Replace SQLite with a cloud-managed service (like **Supabase** or **Firebase**), adding email/Google login.
* **Why**: Saves users' progress across multiple devices (mobile, laptop) instead of locking data to a single local file.

### 2. Custom AI Workout Planner
* **What**: Integrate a profiling questionnaire (age, weight, goal: bulk/cut) and use Llama 3.3 to auto-generate weekly sets, reps, and exercise plans.
* **Why**: Transforms the app from a simple pose tracker into a comprehensive digital wellness planner.

### 3. Gamification, Streaks & Leaderboards
* **What**: Introduce a points system (XP per rep), virtual badges, and a leaderboard where friends can view each other's weekly workout metrics.
* **Why**: Drastically improves user retention and user engagement through friendly competition.

### 4. Client-side WebAssembly Pose Processing
* **What**: Port the MediaPipe model to run directly in the user's browser using JavaScript / WebAssembly instead of passing raw video frames to Python.
* **Why**: Reduces server hosting costs to almost $0 since frame processing happens on the client's device, enabling the app to scale to millions of users smoothly.

### 5. Wearables & Heart Rate Sync
* **What**: Integrate APIs for Apple Watch, Fitbit, or Garmin to capture live heart rate and calorie burn data alongside posture metrics.
* **Why**: Provides deep biological insights, warning users if their heart rate goes too high during intensive sets.
