# Vercel Deployment Configuration Guide

## Issue Resolved
Fixed the 404 Not Found error at `/api/auth/signup` endpoint.

## Root Causes Identified
1. **CORS blocking** - Backend only allowed localhost origins
2. **Route precedence** - Catch-all route intercepted API requests
3. **Missing environment variables** - Frontend didn't know backend URL

## Changes Made

### Backend (`backend/src/index.js`)
- ✅ Dynamic CORS configuration supporting multiple origins
- ✅ Proper route ordering (API routes before catch-all)
- ✅ Health check endpoint `/api/health`
- ✅ API route protection in production mode

### Backend Socket.io (`backend/src/lib/socket.js`)
- ✅ Updated CORS to match allowed origins

### Vercel Configuration (`backend/vercel.json`)
- ✅ Explicit API route handling with all HTTP methods
- ✅ Production environment variable set

### Frontend (`frontend/src/lib/axios.js`)
- ✅ Environment variable support for API URL
- ✅ Proper fallback to production/development URLs

## Deployment Steps

### 1. Backend Deployment on Vercel

**Environment Variables to Set:**
```
MONGODB_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret_key
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
NODE_ENV=production
FRONTEND_URL=https://chatty-liart.vercel.app
PORT=5001
```

**Vercel Settings:**
- Root Directory: `backend`
- Build Command: (leave empty or `npm install`)
- Output Directory: (leave empty)
- Install Command: `npm install`

### 2. Frontend Deployment on Vercel

**Environment Variables to Set:**
```
VITE_API_URL=https://your-backend-url.vercel.app/api
```
Replace `your-backend-url` with your actual backend Vercel deployment URL.

**Vercel Settings:**
- Root Directory: `frontend`
- Build Command: `npm run build`
- Output Directory: `dist`
- Install Command: `npm install`

### 3. Verification Steps

After deployment:

1. **Test Health Check:**
   ```
   GET https://your-backend-url.vercel.app/api/health
   ```
   Should return: `{"status":"OK","message":"Server is running"}`

2. **Test Signup Endpoint:**
   ```
   POST https://your-backend-url.vercel.app/api/auth/signup
   Content-Type: application/json
   
   {
     "fullName": "Test User",
     "email": "test@example.com",
     "password": "test123"
   }
   ```

3. **Check Frontend Connection:**
   - Open browser console on frontend
   - Attempt to sign up
   - Verify no CORS errors
   - Check Network tab for successful API calls

## Troubleshooting

### Still Getting 404?
1. Verify backend deployment succeeded
2. Check Vercel Function logs for errors
3. Ensure environment variables are set correctly

### CORS Errors?
1. Verify `FRONTEND_URL` in backend matches your frontend URL exactly
2. Check that frontend `VITE_API_URL` points to correct backend
3. Ensure both include proper protocol (https://)

### Connection Issues?
1. Test backend health endpoint directly
2. Check MongoDB connection string is correct
3. Verify all environment variables are set

## Important Notes

- **Separate Deployments**: Backend and frontend should be deployed as separate Vercel projects
- **Environment URLs**: Update `FRONTEND_URL` and `VITE_API_URL` after initial deployment
- **Redeploy**: After setting environment variables, trigger a redeploy for changes to take effect
- **HTTPS Only**: Vercel deployments use HTTPS, ensure URLs use `https://` not `http://`

## Testing Locally

Before deploying, test the production build locally:

```bash
# Backend
cd backend
NODE_ENV=production FRONTEND_URL=http://localhost:5173 npm start

# Frontend (new terminal)
cd frontend
VITE_API_URL=http://localhost:5001/api npm run build
npm run preview
```

---
**After following these steps, your 404 error should be resolved!**
