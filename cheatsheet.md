## Backend Cheat Sheet (FastAPI & SQLite)

### 1. Navigate & Setup Environment

```bash
# Move to backend folder from root
cd backend

# Create a Python virtual environment
python -m venv venv

# Activate the virtual environment (Windows PowerShell)
.\venv\Scripts\Activate.ps1

```

### 2. Install Dependencies & Configure Environment

```bash
# Upgrade pip and install core requirements
python -m pip install --upgrade pip
pip install -r requirements.txt

# Create your local environment variables file (copy example)
copy .env.example .env

```

### 3. Run the Backend Development Server

```bash
# Start FastAPI with hot-reloading on port 8000
uvicorn app.main:app --reload --port 8000

```

> **Access Points:**
> * API Docs (Swagger): `[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)`
> * Alternative Docs: `[http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)`
> 
> 

---

## Frontend Cheat Sheet (React, Vite, TS, Tailwind)

### 1. Navigate & Install Dependencies

```bash
# From root, move to frontend folder (or open a new terminal tab)
cd frontend

# Install node modules
npm install

```

### 2. Configure Environment Variables

```bash
# Create local environment configuration if needed
copy .env.example .env

```

### 3. Run the Frontend Development Server

```bash
# Start Vite development server
npm run dev

```

> **Access Point:**
> * Local App URL: Click the `http://localhost:5173` link shown in your terminal.
> 
> 

---

## Running Both Simultaneously (Recommended Workflow)

Open **two separate terminal tabs** in VS Code:

* **Tab 1:** Run backend (`cd backend`, activate `venv`, run `uvicorn`)
* **Tab 2:** Run frontend (`cd frontend`, run `npm run dev`)


---

## Daily Git Workflow Cheat Sheet

Whenever you modify code, create new files, or fix bugs, follow these 3 core steps:

```bash
# 1. Check which files were modified or added
git status

# 2. Stage all changed files (prepares them for commit)
git add .

# 3. Create a snapshot with a descriptive message
git commit -m "feat: add user authentication API endpoint"

# 4. Push your commits to GitHub
git push origin main

```

---

## Collaboration Workflow

Before you start writing new code on your laptop, always pull the latest changes pushed by your partner to keep your project updated:

```bash
# Pull latest changes from GitHub before starting work
git pull origin main

```
