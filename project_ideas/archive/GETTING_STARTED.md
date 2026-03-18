# Getting Started Guide

This guide will help you set up your development environment and start building the 3D character generation website.

## Prerequisites

### Required Software
- **Python 3.9+** with pip
- **Node.js 18+** with npm/yarn
- **Git**
- **Docker** (optional but recommended)
- **CUDA-capable GPU** (NVIDIA, for ML model inference)

### Required Accounts
- Cloud storage account (AWS S3, Google Cloud Storage, or similar)
- GPU compute provider (AWS, Google Cloud, or Lambda Labs)

## Step-by-Step Setup

### 1. Test TripoSR Locally (Most Important First Step)

Before building the full application, verify that the core AI model works:

```bash
# Clone TripoSR
git clone https://github.com/VAST-AI-Research/TripoSR.git
cd TripoSR

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
pip install -r requirements.txt

# Test with sample image
python run.py examples/chair.png --output-dir output/
```

**Expected result:** A 3D model file (.obj or .glb) in the output directory.

**Important:** If this doesn't work, troubleshoot before proceeding!

### 2. Set Up Project Structure

```bash
# Navigate to your project directory
cd /Users/changdentin.ai/caonima

# Create project structure
mkdir -p frontend backend ml_models docker docs

# Initialize git repository
git init
```

### 3. Set Up Frontend

```bash
cd frontend

# Create React + Vite project
npm create vite@latest . -- --template react

# Install dependencies
npm install
npm install three @react-three/fiber @react-three/drei
npm install axios react-dropzone
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# Start development server
npm run dev
```

### 4. Set Up Backend

```bash
cd ../backend

# Create virtual environment
python -m venv venv
source venv/bin/activate

# Create requirements.txt
cat > requirements.txt << EOF
fastapi==0.104.1
uvicorn[standard]==0.24.0
python-multipart==0.0.6
pillow==10.1.0
pydantic==2.5.0
python-dotenv==1.0.0
celery==5.3.4
redis==5.0.1
boto3==1.29.7
sqlalchemy==2.0.23
psycopg2-binary==2.9.9
torch==2.1.0
torchvision==0.16.0
EOF

# Install dependencies
pip install -r requirements.txt

# Create basic FastAPI app
mkdir -p app/api/routes
touch app/__init__.py
touch app/api/__init__.py
touch app/api/main.py
```

### 5. Create Basic FastAPI Server

```bash
# Create app/api/main.py
cat > app/api/main.py << 'EOF'
from fastapi import FastAPI, UploadFile, File
from fastapi.middleware.cors import CORSMiddleware
import uuid

app = FastAPI()

# Enable CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
async def root():
    return {"message": "3D Model Generator API"}

@app.post("/api/upload")
async def upload_image(file: UploadFile = File(...)):
    job_id = str(uuid.uuid4())
    return {
        "job_id": job_id,
        "status": "processing",
        "message": "Image uploaded successfully"
    }

@app.get("/api/status/{job_id}")
async def get_status(job_id: str):
    return {
        "job_id": job_id,
        "status": "completed",
        "progress": 100,
        "model_url": "https://example.com/model.glb"
    }
EOF

# Run the server
cd app/api
uvicorn main:app --reload --port 8000
```

### 6. Create Basic Frontend Component

Create `frontend/src/components/UploadForm.jsx`:

```jsx
import { useState } from 'react';
import axios from 'axios';

export default function UploadForm() {
  const [file, setFile] = useState(null);
  const [loading, setLoading] = useState(false);
  const [jobId, setJobId] = useState(null);

  const handleUpload = async () => {
    if (!file) return;
    
    setLoading(true);
    const formData = new FormData();
    formData.append('file', file);

    try {
      const response = await axios.post('http://localhost:8000/api/upload', formData);
      setJobId(response.data.job_id);
      // Poll for status
      pollStatus(response.data.job_id);
    } catch (error) {
      console.error('Upload failed:', error);
    } finally {
      setLoading(false);
    }
  };

  const pollStatus = async (jobId) => {
    const interval = setInterval(async () => {
      const response = await axios.get(`http://localhost:8000/api/status/${jobId}`);
      if (response.data.status === 'completed') {
        clearInterval(interval);
        // Handle completed model
        console.log('Model URL:', response.data.model_url);
      }
    }, 2000);
  };

  return (
    <div className="p-8">
      <input 
        type="file" 
        accept="image/*"
        onChange={(e) => setFile(e.target.files[0])}
      />
      <button 
        onClick={handleUpload}
        disabled={!file || loading}
        className="ml-4 px-4 py-2 bg-blue-500 text-white rounded"
      >
        {loading ? 'Processing...' : 'Generate 3D Model'}
      </button>
    </div>
  );
}
```

## Testing the Setup

### 1. Test Backend
```bash
cd backend/app/api
uvicorn main:app --reload --port 8000

# In another terminal, test the API
curl http://localhost:8000/
```

### 2. Test Frontend
```bash
cd frontend
npm run dev

# Open browser to http://localhost:5173
```

### 3. Test Full Flow
- Upload an image through the frontend
- Check the network tab in browser DevTools
- Verify the backend receives the request

## Next Steps

1. **Integrate TripoSR into Backend**
   - Create a service class for TripoSR
   - Implement async processing with Celery
   - Add model storage logic

2. **Build 3D Viewer**
   - Use Three.js or @react-three/fiber
   - Implement camera controls
   - Add lighting and materials

3. **Add Database**
   - Set up PostgreSQL
   - Create tables for jobs and models
   - Implement ORM with SQLAlchemy

4. **Deploy to Cloud**
   - Set up GPU server
   - Configure Docker containers
   - Deploy frontend to Vercel/Netlify

## Common Issues & Solutions

### GPU Not Detected
```bash
# Check CUDA installation
python -c "import torch; print(torch.cuda.is_available())"

# If False, reinstall PyTorch with CUDA support
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118
```

### CORS Issues
- Ensure FastAPI CORS middleware is properly configured
- Check that frontend is making requests to correct backend URL

### Model Generation Slow
- Ensure you're using GPU (not CPU)
- Reduce image resolution before processing
- Consider using smaller model variants

### Out of Memory
- Reduce batch size
- Use lower resolution images
- Consider using model quantization

## Development Workflow

1. **Start backend**: `cd backend && uvicorn app.api.main:app --reload`
2. **Start frontend**: `cd frontend && npm run dev`
3. **Start Redis** (for Celery): `redis-server`
4. **Start Celery worker**: `celery -A app.workers.celery_worker worker --loglevel=info`

## Useful Commands

```bash
# Backend
pip freeze > requirements.txt  # Update dependencies
pytest                          # Run tests
black .                         # Format code

# Frontend
npm run build                   # Build for production
npm run preview                 # Preview production build
npm run lint                    # Lint code

# Docker
docker-compose up               # Start all services
docker-compose down             # Stop all services
```

## Resources

- TripoSR Documentation: https://github.com/VAST-AI-Research/TripoSR
- FastAPI Tutorial: https://fastapi.tiangolo.com/tutorial/
- Three.js Examples: https://threejs.org/examples/
- React Three Fiber: https://docs.pmnd.rs/react-three-fiber/

---

**Ready to start coding?** Begin with testing TripoSR locally, then build the upload interface!
