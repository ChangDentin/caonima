# Quick Start: Parametric Character Creation

This is the **recommended approach** - using game-like character creation with AI-driven parameter adjustment.

## 🎯 What We're Building

```
User uploads photo → AI analyzes face → Extracts measurements → 
Maps to character parameters → Generates 3D character → 
User adjusts sliders → Export final model
```

## ⚡ 15-Minute Proof of Concept

Let's build a minimal working version to test the concept!

### Step 1: Install Dlib (Face Analysis)

```bash
# Create project directory
mkdir character-creator
cd character-creator

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install dlib opencv-python numpy pillow

# Download facial landmark model (68 points)
wget http://dlib.net/files/shape_predictor_68_face_landmarks.dat.bz2
bunzip2 shape_predictor_68_face_landmarks.dat.bz2
```

### Step 2: Test Face Detection

Create `test_face_detection.py`:

```python
import dlib
import cv2
import numpy as np
from PIL import Image
import matplotlib.pyplot as plt

# Load models
detector = dlib.get_frontal_face_detector()
predictor = dlib.shape_predictor("shape_predictor_68_face_landmarks.dat")

def detect_landmarks(image_path):
    # Load image
    img = cv2.imread(image_path)
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    
    # Detect faces
    faces = detector(gray)
    
    if len(faces) == 0:
        print("No face detected!")
        return None
    
    # Get landmarks for first face
    landmarks = predictor(gray, faces[0])
    
    # Convert to numpy array
    points = []
    for i in range(68):
        x = landmarks.part(i).x
        y = landmarks.part(i).y
        points.append((x, y))
        # Draw point on image
        cv2.circle(img, (x, y), 2, (0, 255, 0), -1)
    
    # Display result
    plt.imshow(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
    plt.title("Detected Landmarks")
    plt.show()
    
    return np.array(points)

def calculate_measurements(landmarks):
    """Calculate facial measurements from landmarks"""
    measurements = {}
    
    # Face width (distance between jaw points 0 and 16)
    face_width = np.linalg.norm(landmarks[0] - landmarks[16])
    
    # Inter-ocular distance (between inner eye corners)
    left_eye_inner = landmarks[39]
    right_eye_inner = landmarks[42]
    eye_distance = np.linalg.norm(left_eye_inner - right_eye_inner)
    
    # Nose width (distance between nose wings)
    nose_left = landmarks[31]
    nose_right = landmarks[35]
    nose_width = np.linalg.norm(nose_left - nose_right)
    
    # Mouth width (distance between mouth corners)
    mouth_left = landmarks[48]
    mouth_right = landmarks[54]
    mouth_width = np.linalg.norm(mouth_left - mouth_right)
    
    # Face height (top of head to chin)
    face_top = landmarks[27]  # nose bridge
    face_bottom = landmarks[8]  # chin
    face_height = np.linalg.norm(face_top - face_bottom)
    
    # Calculate ratios (normalized measurements)
    measurements['face_width'] = face_width
    measurements['face_height'] = face_height
    measurements['eye_distance_ratio'] = eye_distance / face_width
    measurements['nose_width_ratio'] = nose_width / face_width
    measurements['mouth_width_ratio'] = mouth_width / face_width
    measurements['face_aspect_ratio'] = face_height / face_width
    
    # Eye size (approximation)
    left_eye_width = np.linalg.norm(landmarks[36] - landmarks[39])
    measurements['eye_size_ratio'] = left_eye_width / face_width
    
    # Jaw width (distance between jaw corners)
    jaw_left = landmarks[3]
    jaw_right = landmarks[13]
    jaw_width = np.linalg.norm(jaw_left - jaw_right)
    measurements['jaw_width_ratio'] = jaw_width / face_width
    
    return measurements

def map_to_character_parameters(measurements):
    """Map measurements to character creation parameters"""
    # These would be MakeHuman parameters
    # Values typically range from -1.0 to 1.0
    
    params = {}
    
    # Map eye distance (typical range: 0.15-0.20)
    # Convert to parameter scale
    eye_dist = measurements['eye_distance_ratio']
    params['eyes/distance'] = (eye_dist - 0.175) / 0.025  # normalize around 0.175
    
    # Map nose width (typical range: 0.08-0.12)
    nose_w = measurements['nose_width_ratio']
    params['nose/width'] = (nose_w - 0.10) / 0.02
    
    # Map mouth width (typical range: 0.12-0.18)
    mouth_w = measurements['mouth_width_ratio']
    params['mouth/width'] = (mouth_w - 0.15) / 0.03
    
    # Map face aspect ratio (typical: 1.2-1.5)
    face_aspect = measurements['face_aspect_ratio']
    params['face/length'] = (face_aspect - 1.35) / 0.15
    
    # Map jaw width (typical: 0.45-0.55)
    jaw_w = measurements['jaw_width_ratio']
    params['face/jaw-width'] = (jaw_w - 0.50) / 0.05
    
    # Clamp values to [-1, 1]
    for key in params:
        params[key] = max(-1.0, min(1.0, params[key]))
    
    return params

# Test with your photo
if __name__ == "__main__":
    image_path = "test_photo.jpg"  # Replace with your photo
    
    print("Detecting facial landmarks...")
    landmarks = detect_landmarks(image_path)
    
    if landmarks is not None:
        print("\nCalculating measurements...")
        measurements = calculate_measurements(landmarks)
        
        print("\nFacial Measurements:")
        for key, value in measurements.items():
            print(f"  {key}: {value:.4f}")
        
        print("\nCharacter Parameters:")
        params = map_to_character_parameters(measurements)
        for key, value in params.items():
            print(f"  {key}: {value:.4f}")
```

### Step 3: Run the Test

```bash
# Add a test photo to your directory (name it test_photo.jpg)
python test_face_detection.py
```

**Expected Output:**
- Image with 68 green dots showing detected landmarks
- Printed measurements (ratios)
- Printed character parameters (-1 to 1 range)

---

## 🎮 Step 4: Test MakeHuman

### Option A: Desktop App (Easy)

```bash
# Download and install MakeHuman
# Mac: http://www.makehumancommunity.org/content/downloads.html
# Or via Homebrew: brew install --cask makehuman

# Launch MakeHuman and explore the interface
# Try adjusting sliders under "Modelling" tab
# Note down parameter names you want to control
```

### Option B: Python API (For Integration)

MakeHuman can be controlled via Python, but it's easier to start with the desktop app to understand the parameters.

For programmatic access, you can:
1. Use MakeHuman's Python API (requires running MakeHuman as module)
2. Or use Blender + MB-Lab addon (more flexible)

---

## 🚀 Step 5: Build Simple Web Interface

Create `app.py` (FastAPI backend):

```python
from fastapi import FastAPI, UploadFile, File
from fastapi.middleware.cors import CORSMiddleware
import dlib
import cv2
import numpy as np
import io
from PIL import Image

app = FastAPI()

# Enable CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# Load models (do this once at startup)
detector = dlib.get_frontal_face_detector()
predictor = dlib.shape_predictor("shape_predictor_68_face_landmarks.dat")

@app.post("/api/analyze")
async def analyze_face(file: UploadFile = File(...)):
    """Analyze uploaded photo and return character parameters"""
    
    # Read uploaded image
    contents = await file.read()
    nparr = np.frombuffer(contents, np.uint8)
    img = cv2.imdecode(nparr, cv2.IMREAD_COLOR)
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    
    # Detect face
    faces = detector(gray)
    if len(faces) == 0:
        return {"error": "No face detected"}
    
    # Get landmarks
    landmarks = predictor(gray, faces[0])
    points = np.array([[landmarks.part(i).x, landmarks.part(i).y] for i in range(68)])
    
    # Calculate measurements
    measurements = calculate_measurements(points)
    
    # Map to parameters
    parameters = map_to_character_parameters(measurements)
    
    return {
        "success": True,
        "landmarks": points.tolist(),
        "measurements": measurements,
        "parameters": parameters
    }

def calculate_measurements(landmarks):
    # Same as before
    face_width = np.linalg.norm(landmarks[0] - landmarks[16])
    eye_distance = np.linalg.norm(landmarks[39] - landmarks[42])
    nose_width = np.linalg.norm(landmarks[31] - landmarks[35])
    mouth_width = np.linalg.norm(landmarks[48] - landmarks[54])
    face_height = np.linalg.norm(landmarks[27] - landmarks[8])
    
    return {
        'eye_distance_ratio': eye_distance / face_width,
        'nose_width_ratio': nose_width / face_width,
        'mouth_width_ratio': mouth_width / face_width,
        'face_aspect_ratio': face_height / face_width,
    }

def map_to_character_parameters(measurements):
    # Same as before
    params = {
        'eyes_distance': (measurements['eye_distance_ratio'] - 0.175) / 0.025,
        'nose_width': (measurements['nose_width_ratio'] - 0.10) / 0.02,
        'mouth_width': (measurements['mouth_width_ratio'] - 0.15) / 0.03,
        'face_length': (measurements['face_aspect_ratio'] - 1.35) / 0.15,
    }
    
    # Clamp to [-1, 1]
    for key in params:
        params[key] = max(-1.0, min(1.0, params[key]))
    
    return params

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

Run the backend:
```bash
pip install fastapi uvicorn python-multipart
python app.py
```

---

## 🎨 Step 6: Create Frontend

Create `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Character Creator</title>
    <style>
        body { font-family: Arial; max-width: 1200px; margin: 0 auto; padding: 20px; }
        .container { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        .upload-area { border: 2px dashed #ccc; padding: 40px; text-align: center; }
        .preview { max-width: 100%; }
        .parameters { background: #f5f5f5; padding: 20px; }
        .slider-container { margin: 10px 0; }
        .slider { width: 100%; }
        button { background: #4CAF50; color: white; padding: 10px 20px; border: none; cursor: pointer; }
        button:hover { background: #45a049; }
    </style>
</head>
<body>
    <h1>AI Character Creator</h1>
    
    <div class="container">
        <div>
            <h2>1. Upload Photo</h2>
            <div class="upload-area">
                <input type="file" id="fileInput" accept="image/*">
                <button onclick="analyzePhoto()">Analyze Face</button>
            </div>
            <img id="preview" class="preview" style="display:none;">
        </div>
        
        <div>
            <h2>2. Adjust Parameters</h2>
            <div id="parameters" class="parameters">
                <p>Upload a photo to see parameters...</p>
            </div>
            <button onclick="generateCharacter()" style="margin-top: 20px;">Generate Character</button>
        </div>
    </div>
    
    <div id="result" style="margin-top: 40px;"></div>

    <script>
        let currentParameters = {};
        
        document.getElementById('fileInput').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    document.getElementById('preview').src = e.target.result;
                    document.getElementById('preview').style.display = 'block';
                };
                reader.readAsDataURL(file);
            }
        });
        
        async function analyzePhoto() {
            const fileInput = document.getElementById('fileInput');
            const file = fileInput.files[0];
            
            if (!file) {
                alert('Please select a photo first');
                return;
            }
            
            const formData = new FormData();
            formData.append('file', file);
            
            try {
                const response = await fetch('http://localhost:8000/api/analyze', {
                    method: 'POST',
                    body: formData
                });
                
                const data = await response.json();
                
                if (data.error) {
                    alert(data.error);
                    return;
                }
                
                currentParameters = data.parameters;
                displayParameters(data.parameters);
                
            } catch (error) {
                console.error('Error:', error);
                alert('Failed to analyze photo');
            }
        }
        
        function displayParameters(params) {
            const container = document.getElementById('parameters');
            container.innerHTML = '<h3>Character Parameters:</h3>';
            
            for (const [key, value] of Object.entries(params)) {
                const div = document.createElement('div');
                div.className = 'slider-container';
                div.innerHTML = `
                    <label>${key}: <span id="${key}-value">${value.toFixed(2)}</span></label><br>
                    <input type="range" class="slider" id="${key}" 
                           min="-1" max="1" step="0.01" value="${value}"
                           oninput="updateParameter('${key}', this.value)">
                `;
                container.appendChild(div);
            }
        }
        
        function updateParameter(key, value) {
            currentParameters[key] = parseFloat(value);
            document.getElementById(key + '-value').textContent = parseFloat(value).toFixed(2);
        }
        
        function generateCharacter() {
            document.getElementById('result').innerHTML = `
                <h3>Character Parameters:</h3>
                <pre>${JSON.stringify(currentParameters, null, 2)}</pre>
                <p><em>In a real implementation, these parameters would be sent to MakeHuman 
                to generate the actual 3D model.</em></p>
            `;
        }
    </script>
</body>
</html>
```

Open `index.html` in your browser!

---

## 🎯 What You've Built

A working proof-of-concept that:
1. ✅ Uploads photos
2. ✅ Detects 68 facial landmarks
3. ✅ Calculates facial measurements
4. ✅ Maps to character parameters
5. ✅ Displays interactive sliders
6. ✅ Allows manual adjustment

**Next steps to make it complete:**
- Integrate MakeHuman to actually generate 3D models
- Add Three.js viewer to display the model
- Add more parameters (currently just 4-5)
- Improve parameter mapping accuracy
- Add export functionality

---

## 🔧 Troubleshooting

### "No face detected"
- Use clear, well-lit, front-facing photos
- Face should be clearly visible, not too far away
- Try different photos

### Dlib installation fails
```bash
# Mac with M1/M2:
brew install cmake
pip install dlib

# Ubuntu/Debian:
sudo apt-get install cmake libopenblas-dev liblapack-dev
pip install dlib
```

### Model file not found
```bash
# Download again
wget http://dlib.net/files/shape_predictor_68_face_landmarks.dat.bz2
bunzip2 shape_predictor_68_face_landmarks.dat.bz2
```

---

## 📈 Next Steps

1. **Improve Parameter Mapping**
   - Test with many different faces
   - Tune the normalization constants
   - Add more parameters (cheekbones, forehead, etc.)

2. **Integrate MakeHuman**
   - Use MakeHuman Python API
   - Or use Blender + MB-Lab programmatically
   - Generate actual 3D models

3. **Add 3D Viewer**
   - Integrate Three.js
   - Load GLB models
   - Add camera controls

4. **Train ML Model (Optional)**
   - Generate dataset of MakeHuman characters
   - Train neural network for better mapping
   - Replace hand-crafted formulas

5. **Polish UI**
   - Better design
   - Real-time preview
   - Save/load presets

---

## 🎉 You're Ready!

You now have a working prototype! The hard parts (face detection, parameter extraction) are done. 

The remaining work is:
- Connecting to MakeHuman
- Adding 3D visualization
- Polishing the UI

See `REVISED_PLAN_PARAMETRIC.md` for the complete implementation roadmap!
