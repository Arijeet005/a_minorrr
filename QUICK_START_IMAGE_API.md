# 🚀 Quick Start - Image API Fix

## What Was Wrong
1. **🔴 PORT MISMATCH**: Backend looking for ML service on port 5002, but it runs on port 5001
2. **🔴 DEPENDENCIES**: TensorFlow 2.19.0 + NumPy 2.4.2 (too recent, incompatible)
3. **🔴 NO LOGGING**: Couldn't debug what was happening in the ML service

## What's Fixed
✅ Port changed from 5002 → 5001  
✅ TensorFlow 2.17.0 + NumPy 1.26.4 (stable)  
✅ Full logging & diagnostics added  
✅ Documentation updated  

---

## 📋 Setup (5 minutes)

### Terminal 1: ML Service
```bash
cd code/ml-service
pip install -r requirements.txt
python -m uvicorn app:app --host 0.0.0.0 --port 5001
```

**Expected output:**
```
Uvicorn running on http://0.0.0.0:5001
```

### Terminal 2: Backend
```bash
cd code/backend
npm install
node server.js
```

**Expected output:**
```
Server running on port 5000
```

### Terminal 3: Verify Everything
```bash
cd code
node test-image-api.js
```

**You should see ✓ checkmarks for all tests**

---

## 🧪 Manual Test (Optional)

Test ML service directly:
```bash
curl http://localhost:5001/health
```

Should show:
```json
{
  "ok": true,
  "tensorflow_available": true,
  "fv_model_present": true,
  ...
}
```

---

## 🎯 Test in App
1. Go to **InventoryHub** in frontend
2. Upload a food image (tomato, pepper, etc.)
3. You should now get **correct ingredient identification**

---

## ❓ Still Having Issues?

### Check ML Service Health
```bash
curl http://localhost:5001/health
```

### Check Backend Logs
Look for errors about port 5002 → should be gone now

### Reinstall Dependencies
```bash
cd code/ml-service
pip uninstall tensorflow numpy -y
pip install -r requirements.txt
```

### See What's Happening
ML service now logs everything. Check terminal output for:
- `TensorFlow version: 2.17.0`
- `Models directory: ...`
- Prediction logs with confidence

---

## Files Changed
- `backend/controllers/imageController.js` - Fixed port
- `ml-service/requirements.txt` - Fixed dependencies
- `ml-service/app.py` - Added logging
- `ml-service/README.md` - Updated docs
- `test-image-api.js` - New test tool
- `SETUP_IMAGE_API.md` - Detailed guide
