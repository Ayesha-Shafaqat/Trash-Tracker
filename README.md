# Trash Tracker

Trash Tracker is a waste-site monitoring system that combines a Python/FastAPI backend, a React web dashboard for supervisors, and an Expo mobile app for field workers. The backend can run YOLO inference on satellite images, store detected sites and task records in MySQL, and accept cleanup proof photos from workers.

## Features

- Scan dated satellite-image folders for possible trash sites using the included YOLO model.
- View detections and their locations in the supervisor dashboard.
- Assign cleanup tasks to registered workers and review completed work.
- Use the worker app to view assigned tasks, open a map, and submit a proof photo.
- Serve detection images, worker proof photos, and profile images from the backend.

## Project Layout

```text
backend/                    FastAPI API, model weights, data, and uploaded media
  app/                      API and database models
  models/                   YOLO model weights
  satellite_dataset/        Source images grouped by YYYY-MM-DD
  static/                   Detection, proof, and profile images
frontend/trash-dashboard/   React + Vite supervisor dashboard
worker-app/                 Expo / React Native worker app
requirements.txt            Python backend dependencies
```

## Prerequisites

- Python 3.10 or newer (Python 3.10-3.12 recommended for ML package compatibility).
- Node.js 20.19+ or 22.12+ and npm.
- MySQL or MariaDB running locally. The default connection is configured for a local XAMPP-style MySQL setup.
- For the worker app: Expo Go on a compatible phone, or an Android emulator. Building/running the iOS native app requires macOS and Xcode.
- The included model weights and satellite images. Keep `backend/models/`, `backend/satellite_dataset/`, and `backend/Trash_Locations.kml` in place for detection workflows.

## Setup

### 1. Create the database

Start MySQL, then create the database:

```sql
CREATE DATABASE trash_tracker_db CHARACTER SET utf8mb4;
```

The connection string is currently hard-coded in `backend/app/database.py` as `mysql+pymysql://root:@localhost/trash_tracker_db`. Change it there if your MySQL username, password, host, or database name differs. Start the backend once after creating the database; it creates the application tables automatically.

### 2. Start the backend

From the repository root, create and activate a virtual environment, then install the backend dependencies:

```powershell
py -m venv backend/venv
backend/venv/Scripts/Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
cd backend
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

On macOS or Linux, activate the environment with `source backend/venv/bin/activate` before changing into `backend/`.

When the server is running, the API is at `http://127.0.0.1:8000` and interactive API documentation is at `http://127.0.0.1:8000/docs`. YOLO/PyTorch installation can take a while and requires several gigabytes of free disk space. Inference is slower on CPU; a compatible NVIDIA setup is optional.

### 3. Create a supervisor login

The project does not currently include a supervisor signup screen or seed account. After starting the backend once (so the tables exist), run this in the MySQL client and replace the values:

```sql
USE trash_tracker_db;
INSERT INTO supervisors (name, email, password_hash)
VALUES ('Your Name', 'you@example.com', 'choose-a-local-demo-password');
```

Despite the column name, the current application compares this value as plain text. Use only throwaway local-demo credentials; do not expose this setup publicly or reuse a real password. Workers can create accounts using the worker app's signup screen.

### 4. Start the web dashboard

Open a second terminal at the repository root:

```powershell
cd frontend/trash-dashboard
npm install
npm run dev
```

Open the local URL printed by Vite (usually `http://localhost:5173`). The dashboard currently calls the API at `http://127.0.0.1:8000`, so run it in a browser on the same computer as the backend unless you update those API URLs in the dashboard source.

### 5. Start the worker app (optional)

Open a third terminal at the repository root:

```powershell
cd worker-app
npm install
npx expo start
```

Use the Expo Go app to scan the QR code, or press `a` to open an Android emulator. For a physical phone, replace the hard-coded `192.168.152.248` backend address in the worker app with the computer's current LAN IPv4 address. The address appears in the API URL constants and image URL replacements in `worker-app/app/index.tsx`, `worker-app/app/(tabs)/index.tsx`, `worker-app/app/(tabs)/map.tsx`, and `worker-app/app/(tabs)/upload-proof.tsx`. The phone and computer must be on the same network, the backend must be running with `--host 0.0.0.0`, and the computer firewall must allow port 8000.

## Using Detection

Place supported `.jpg` or `.png` images in a date-named folder under `backend/satellite_dataset/`, for example `backend/satellite_dataset/2026-10-11/`. Keep the KML file at `backend/Trash_Locations.kml`; its placemark names are used to match image/site names to coordinates. In the dashboard, select that date and run a scan. The backend uses `backend/models/best(1).pt`; if this file is missing, the model will not load and scans cannot run.

## Important Notes

- The backend uses permissive development CORS and hard-coded local URLs. This project is not production-hardened; configure secrets, authentication/password hashing, allowed origins, and network access before deploying or exposing it to the internet.
- The default supervisor login is not seeded automatically. See the SQL setup above.
- `backend/static/` contains uploaded images and generated detections. Preserve it if you need those files after moving or backing up the project.
- The root `requirements.txt` is for the Python backend only. Each JavaScript app has its own `package.json` and lockfile; install dependencies separately in each app directory.

## GitHub Description

Waste-site monitoring system with YOLO-based satellite image detection, a FastAPI/MySQL backend, a React supervisor dashboard, and an Expo worker app for task assignment and cleanup proof uploads.