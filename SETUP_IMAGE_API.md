# Image API Setup & Troubleshooting Guide

## Issues Fixed

### 1. ✅ Port Mismatch (CRITICAL)
- **Problem**: Backend was looking for ML service on port `5002`, but it runs on port `5001`
- **Fixed in**: `code/backend/controllers/imageController.js` (line 36)
- **Changed**: `http://localhost:5002/identify-ingredient` → `http://localhost:5001/identify-ingredient`

### 2. ✅ TensorFlow/NumPy Compatibility
- **Problem**: tensorflow==2.19.0 + numpy==2.4.2 (too recent, likely incompatible)
- **Fixed in**: `code/ml-service/requirements.txt`
- **Updated to stable versions**:
  - tensorflow: 2.19.0 → 2.17.0
  - numpy: 2.4.2 → 1.26.4
  - pillow: 11.3.0 → 10.3.0
  - Other dependencies updated for compatibility

### 3. ✅ Enhanced Model Loading & Logging
- Added debug logging to ML service
- Enhanced `/health` endpoint to show TensorFlow status
- Improved error messages in `/identify-ingredient` endpoint
- Added filename tracking in responses

### 4. ✅ Updated Documentation
- Added `/identify-ingredient` endpoint to README
- Documented troubleshooting steps

## Setup Instructions

### Step 1: Fresh ML Service Setup
```bash
cd code/ml-service

# Remove old packages to avoid conflicts
pip uninstall tensorflow numpy pandas scikit-learn -y

# Install fresh dependencies
pip install -r requirements.txt
```

### Step 2: Start ML Service
```bash
cd code/ml-service
python -m uvicorn app:app --host 0.0.0.0 --port 5001
```

### Step 3: Verify ML Service Health
```bash
curl http://localhost:5001/health
```

Expected output (should show all models and TensorFlow status):
```json
{
  "ok": true,
  "models_dir": "C:\\Users\\...\\models",
  "food_waste_model_present": true,
  "dish_recommender_model_present": true,
  "fv_model_present": true,
  "fv_labels_present": true,
  "tensorflow_available": true,
  "tensorflow_version": "2.17.0"
}
```

### Step 4: Start Backend
```bash
cd code/backend
npm install
node server.js
```

### Step 5: Test Image API
```bash
# Upload a test image (jpg/png)
curl -X POST -F "image=@path/to/image.jpg" http://localhost:5000/api/images/identify-ingredient
```

Expected response:
```json
{
  "success": true,
  "data": {
    "filename": "image.jpg",
    "name": "tomato",
    "confidence": 0.95,
    "ml": {
      "provider": "ml-service",
      "url": "http://localhost:5001/identify-ingredient",
      "modelFile": "FV.h5"
    }
  }
}
```

## Debugging

### If you still get "Unknown" predictions:
1. Check `/health` endpoint - verify `tensorflow_available` is `true`
2. Check that FV.h5 and FV.labels.json exist in `models/` folder
3. Check backend logs for connection errors
4. Ensure both services are running and can communicate

### If TensorFlow fails to import:
```bash
# Full reinstall
cd code/ml-service
pip install --force-reinstall tensorflow==2.17.0
```

### If you see port 5002 errors in backend logs:
- Backend imageController.js has been updated to use port 5001
- Restart the backend service

## File Changes Summary

1. **code/backend/controllers/imageController.js** - Port changed from 5002 to 5001
2. **code/ml-service/requirements.txt** - Dependency versions stabilized
3. **code/ml-service/app.py** - Added logging and enhanced health check
4. **code/ml-service/README.md** - Updated documentation
