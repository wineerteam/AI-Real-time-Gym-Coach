# 🧠 Deep-Dive Tech Stack Handbook (Frontend & Backend)

This handbook provides an in-depth breakdown of every single frontend and backend component in the **AI Real-time GYM Coach** project. Each section explains the concept, code syntax, parameters, alternatives, and a conceptual analogy in Hinglish.

---

## 🎨 FRONTEND TECH STACK

Frontend handles the user interface (UI), inputs (sets, reps, exercises), stats visualization (metrics, history tables), and direct media playback (webcam stream and autoplay audio cues).

### 1. Streamlit Layout & Controls
* **What**: stream-based Python widgets used to render HTML structures like forms, sidebars, select boxes, and numbers.
* **Why**: Saves us from writing HTML/CSS boilerplates. Streamlit widgets auto-refresh the UI dynamically whenever a user updates a value.
* **Syntax/Code Example** (from `main.py`):
  ```python
  # Sidebar container and inputs
  with st.sidebar:
      st.title("🏋️‍♂️ Apna AI Coach")
      plan_exercise = st.selectbox(
          label="Exercise", 
          options=["Squats", "Push-ups", "Lunges"], 
          key="plan_exercise"
      )
      plan_sets = st.number_input(
          label="Sets", 
          min_value=0, 
          max_value=50, 
          key="plan_sets", 
          step=1
      )
      start_session_button = st.button("Start Workout", width="stretch")
  ```
  * *Parameters:*
    * `options`: Dropdown selection array.
    * `key`: Unique session state identifier to retain values.
    * `step`: Increments/decrements by this step amount.
* **Alternatives**: React `<select>` & `<input type="number">`, Vue.js input bindings.
* **Analogy (Hinglish)**: Streamlit widgets bilkul ek **"Remote Control"** ki tarah hain. Buttons ko click karne ya channel change karne (inputs badalne) par screen (UI) instantly response deti hai bina aapko TV ke internal components ko manually wire kiye.

---

### 2. Streamlit Session State & Metrics Visualization
* **What**: Client-specific dictionary variable (`st.session_state`) that persists variables across periodic app refreshes, paired with `st.metric` UI cards.
* **Why**: Streamlit reruns the whole script top-to-bottom on any user action. To prevent variable resets (like total reps going back to 0), we store states in `session_state`.
* **Syntax/Code Example** (from `session_defaults.py` & `main.py`):
  ```python
  # Initializing state
  if "reps" not in st.session_state:
      st.session_state["reps"] = 0

  # Displaying metrics card in main panel
  st.metric(
      label="Current Set Reps", 
      value=f"{st.session_state.current_set_reps} / {st.session_state.reps_per_set}"
  )
  ```
  * *Parameters:*
    * `label`: Metric category title.
    * `value`: Current count string to display in big bold font.
* **Alternatives**: Redux state (React), Pinia (Vue), Context API.
* **Analogy (Hinglish)**: `st.session_state` ek **"Gym Blackboard"** jaisa hai jahan trainer chalk se reps aur sets likhta hai. Har rep ke baad purane count ko mita kar naya count update ho jata hai taaki status delete na ho.

---

### 3. Custom CSS & Font Injection (UI Customization)
* **What**: Injecting raw `<style>` blocks into Streamlit's virtual DOM to overwrite default fonts (like AdobeClean.otf) and styling variables.
* **Why**: Streamlit UI looks default and plain. We inject raw CSS and Base64 font assets so that all buttons, sidebars, and custom layouts match a dark premium gym theme.
* **Syntax/Code Example** (from `style_loader.py`):
  ```python
  def load_css(file_path):
      if os.path.exists(file_path):
          with open(file_path) as f:
              st.markdown(f"<style>{f.read()}</style>", unsafe_allow_html=True)
  ```
  * *Parameters:*
    * `unsafe_allow_html=True`: Tells Streamlit to parse raw HTML tags (like `<style>` and `@font-face`) instead of treating them as plain strings.
* **Alternatives**: Webpack CSS Loaders, Tailwind CSS directives.
* **Analogy (Hinglish)**: CSS Injection bilkul ek **"Car Wrap/Sticker"** ki tarah hai. Car ka engine aur base structure (Streamlit) wahi rehta hai, par hum custom wrap lagakar uska paint, font, aur style look change kar derm hain.

---

### 4. Custom Iframe DOM Injector (WebRTC Buttons Styling Patch)
* **What**: Dynamic Javascript execution that searches for embedded Streamlit-webrtc iframes and injects custom fonts/styles into the iframe's isolated document object.
* **Why**: WebRTC renders its start/stop buttons inside an isolated child `<iframe>`. Custom styles from the parent page do not apply inside iframes due to browser security boundaries. This JS patch injects CSS directly into the iframe's `<head>`.
* **Syntax/Code Example** (from `style_loader.py`):
  ```python
  components.html(f"""
      <script>
      (function patchWebRTCStyles() {{
          function injectIntoIframe(iframe) {{
              const doc = iframe.contentDocument || iframe.contentWindow.document;
              if (doc.head.querySelector('#webrtc-custom-styles')) return;
              const style = doc.createElement('style');
              style.textContent = `.MuiButton-root {{ border-radius: 0 !important; }}`;
              doc.head.appendChild(style);
          }}
          // search and patch iframe
          const iframes = window.parent.document.querySelectorAll('iframe');
          iframes.forEach(iframe => {{ ... }});
      }})();
      </script>
  """, height=0)
  ```
* **Alternatives**: Direct inline style configurations, compiling custom CSS into the WebRTC react package.
* **Analogy (Hinglish)**: Iframe styling bypass ek **"Passport Office Counter Window"** ki tarah hai. Glass window ke uss paar (iframe) aap directly haath nahi daal sakte. Lekin aap ek paper slip window ke niche se push karke desk worker (iframe document) ko custom rule de sakte hain.

---

### 5. Hidden Autoplay Audio Pipeline
* **What**: Hidden HTML5 `<audio>` player widget triggered dynamically through python bytes streams.
* **Why**: Standard web browsers block autoplay audio to prevent spam. By combining `st.audio`'s native autoplay feature with a CSS snippet that hides the control widget, we achieve a silent voice cue mechanism.
* **Syntax/Code Example** (from `voice_pipeline.py`):
  ```python
  def autoplay_audio(audio_bytes):
      # CSS to hide the audio player bar from the layout
      st.markdown(
          "<style>[data-testid='stAudio'] {display: none;}</style>", 
          unsafe_allow_html=True
      )
      # Stream raw bytes as mp3 and trigger autoplay
      st.audio(audio_bytes, format="audio/mp3", autoplay=True)
  ```
* **Alternatives**: Javascript Audio Context API (`new Audio()`), Web Speech synthesis API.
* **Analogy (Hinglish)**: Yeh **"Invisible Background Music Speaker"** jaisa hai. Speaker background mein chalta hai par physical speaker kisi closet (display: none) mein chhupa hai taaki aesthetic clean rahe.

---

## ⚙️ BACKEND TECH STACK

Backend handles thread locking, computer vision (landmarks extraction), exercise logic calculations (angles and state machines), databases management, LLM generation, and speech rendering.

### 1. SQLite3 Database Persistence (Local History)
* **What**: Relational SQL engine that runs locally in-memory or reads/writes directly to a single `data.db` file.
* **Why**: Saves completed sets, times, and rep stats. By using `check_same_thread=False` and caching, it prevents database lock issues when multiple Streamlit threads write data.
* **Syntax/Code Example** (from `exercise_repository.py`):
  ```python
  import sqlite3
  import streamlit as st

  @st.cache_resource
  def _get_connection():
      conn = sqlite3.connect("data.db", check_same_thread=False)
      conn.row_factory = sqlite3.Row  # Returns rows as dictionary-like objects
      return conn

  def add_exercise(user_id, exercise_name, reps, sets, time):
      conn = _get_connection()
      with conn:  # Autocommits transaction
          conn.execute(
              "INSERT INTO exercises (user_id, exercise_name, sets, reps, time) VALUES (?, ?, ?, ?, ?)",
              (user_id, exercise_name, sets, reps, time)
          )
  ```
* **Alternatives**: PostgreSQL (using SQLAlchemy), MongoDB (NoSQL).
* **Analogy (Hinglish)**: SQLite ek **"Self-contained Ledger (Khata Register)"** ki tarah hai jo ek single notebook (local file) mein save hota hai. Hamein door kisi block building ya accountant office (database server) tak safar nahi karna padta.

---

### 2. MediaPipe Pose Landmarker Engine (Vision Node)
* **What**: High-speed ML detector that returns normalized coordinates (X, Y, Z, Visibility) for 33 keypoints on a human body.
* **Why**: Helps calculate relative angles between joint segments.
* **Syntax/Code Example** (from `exercise_video_processor.py`):
  ```python
  from mediapipe.tasks import python
  from mediapipe.tasks.python import vision

  model_path = "ml_models/pose_landmarker_full.task"
  base_option = python.BaseOptions(model_asset_path=model_path)
  
  options = vision.PoseLandmarkerOptions(
      base_options=base_option,
      running_mode=vision.RunningMode.VIDEO,  # Optimized for video frame sequence
      min_pose_detection_confidence=0.7,
      min_tracking_confidence=0.7
  )
  self._landmarker = vision.PoseLandmarker.create_from_options(options)

  # Processing inside recv() frame thread
  result = self._landmarker.detect_for_video(mp_image, timestamp_ms)
  ```
  * *Parameters:*
    * `running_mode`: Video mode uses spatial temporal filters across consecutive frames to reduce point jittering.
    * `min_tracking_confidence`: Accuracy threshold before dropping keypoints.
* **Alternatives**: YOLOv8-pose model detection, OpenPose engine.
* **Analogy (Hinglish)**: MediaPipe ek **"Puppeteer (Kathputli Chalane Wala)"** jaisa hai jo hamare haath-paanv ko invisible strings (33 joints) se track karta hai aur hamari body movements ko numbers mein decode karta hai.

---

### 3. Exercise Detector Logic (Vector Mathematics)
* **What**: Geometry class that computes 2D vectors and joint angles ($\theta$) to manage exercise state machines (UP vs DOWN stages).
* **Why**: Standardizes movement ranges (e.g., Squat depth status transitions from "down" to "up" only when knee angle crosses $\le 100^\circ$ and then $\ge 160^\circ$).
* **Syntax/Code Example** (from `base_exercise.py` & `squat.py`):
  ```python
  # Vector Dot Product & Cosine Theorem
  def calculate_angle(self, a, b, c):
      ax, ay = a[0] - b[0], a[1] - b[1]  # Vector BA
      cx, cy = c[0] - b[0], c[1] - b[1]  # Vector BC

      dot = ax * cx + ay * cy
      mag_a = math.sqrt(ax**2 + ay**2)
      mag_c = math.sqrt(cx**2 + cy**2)

      if mag_a * mag_c == 0:
          return 0.0

      cos_angle = dot / (mag_a * mag_c)
      return math.degrees(math.acos(max(-1.0, min(1.0, cos_angle))))
  ```
* **Alternatives**: Deep Learning posture classification classifiers, simple threshold coordinates matching.
* **Analogy (Hinglish)**: Detector logic ek **"Mechanical Joint Hinge (Kabza)"** ki tarah hai. Jaise darwaza band hone par angle $0^\circ$ aur poora khulne par $90^\circ$ hota hai, ye logic body joints ko real-time digital joints mein map karta hai.

---

### 4. OpenCV Visual Rendering & Overlays
* **What**: Real-time rendering module that mutates image pixel matrices to draw skeleton paths and visual labels.
* **Why**: Shows immediate feedback to the user on the camera stream screen.
* **Syntax/Code Example** (from `exercise_video_processor.py`):
  ```python
  # Draw a skeletal connector line
  cv2.line(
      img=image, 
      pt1=(int(p1.x * width), int(p1.y * height)), 
      pt2=(int(p2.x * width), int(p2.y * height)), 
      color=(0, 255, 0),  # Green color BGR
      thickness=8
  )
  # Write text status overlay
  cv2.putText(
      img=image,
      text=f"DEPTH: {metrics['depth_status']}",
      org=(20, height - 20),
      fontFace=cv2.FONT_HERSHEY_SIMPLEX,
      fontScale=1,
      color=(0, 255, 0),
      thickness=2
  )
  ```
* **Alternatives**: Canvas overlay (HTML5), PyGame rendering.
* **Analogy (Hinglish)**: OpenCV overlay ek **"Glass Pane (Kanch ka board)"** jaisa hai jo camera lens ke aage laga hai. Aap piche actual target video dekh rahe ho, par glass ke upar green ink aur markers se feedback draw kiya ja raha hai.

---

### 5. Groq Llama AI Client (LLM Generation)
* **What**: AI text client executing high-performance LLM reasoning based on context prompts.
* **Why**: Dynamically analyzes the user's movement errors and crafts human-like coaching advice.
* **Syntax/Code Example** (from `llm.py`):
  ```python
  from groq import Groq

  class LLMCoach:
      def __init__(self, groq_client):
          self.client = groq_client
          self.history = []

      def give_feedback(self, event, issue):
          prompt = f"Event: {event}"
          if issue:
              prompt += f" Form Issue: {issue}"

          messages = [
              {"role": "system", "content": self.system_prompt},
              *self.history[-10:],  # Chat context window
              {"role": "user", "content": prompt}
          ]
          
          response = self.client.chat.completions.create(
              model="llama-3.3-70b-versatile",
              messages=messages,
              temperature=0.4
          )
          return response.choices[0].message.content.strip()
  ```
* **Alternatives**: OpenAI client, Cohere API, HuggingFace Inference API.
* **Analogy (Hinglish)**: Groq LLM client ek **"Personal Trainer on Call"** ki tarah hai. Aap unhe phone karke batate ho "Event: Squat form issue: too high", aur hui instantly ek line mein call pe jawab deta hai: "Depth thodi badhao, hips niche karo!"

---

### 6. gTTS (Google Text-To-Speech API Wrapper)
* **What**: Client library wrapper sending text to Google Translate servers and returning MP3 audio streams.
* **Why**: Speaks the LLM feedback text aloud.
* **Syntax/Code Example** (from `tts.py`):
  ```python
  from io import BytesIO
  from gtts import gTTS

  class TextToSpeech:
      def speak(self, text, lang="en"):
          cleaned = (text or "").strip()
          if not cleaned:
              return None
          
          buffer = BytesIO()  # Keep MP3 bytes in RAM (No disk write delay)
          gTTS(text=cleaned, lang=lang).write_to_fp(buffer)
          buffer.seek(0)
          return buffer.read()
  ```
* **Alternatives**: ElevenLabs SDK, Offline `pyttsx3` engine.
* **Analogy (Hinglish)**: gTTS ek **"Voice Dubbing Artist"** hai jo screen script (LLM Text) ko padh kar turant audio file stream mein convert kar deta hai taaki use instantly sunaya ja sake.
