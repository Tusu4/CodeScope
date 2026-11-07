# CodeScope Backend: Execution Engine, AST Sanitizer & Diagnostic API Specification

> **Phase 1 Production Blueprint & Step-by-Step Task Execution Guide**  
> Target Stack: Python 3.11+ | FastAPI 0.100+ | Pytest | Pydantic v2 | Uvicorn

---

## 1. Backend Architecture & System Responsibilities

The CodeScope backend serves as the core intelligence and execution layer of the platform. It does not perform pre-rendered animation; rather, it acts as a **dynamic Python Execution Tracer, AST Security Guard, and State Serializer**.

### Core Responsibilities:
1. **Authentication & Session Management:** Handle secure JWT user registration, login, and session persistence.
2. **AST Security Filtering (`app/services/ast_sanitizer.py`):** Analyze incoming user code strings prior to execution. Reject malicious commands (`os`, `sys`, `subprocess`, file operations, network connections, dynamic `eval`/`exec`) and reject non-code plain text strings.
3. **Execution State Tracing (`app/services/state_tracer.py`):** Execute sanitized Python code in an isolated scope using `sys.settrace()`. Capture variable mutations, array indices, dictionary updates, pointer steps, and stack frames line-by-line.
4. **Partial Execution & Failure Capture:** If user code fails mid-way (e.g., succeeds lines 1–10 but crashes line 11), record all valid frames up to line 10, capture the exact exception and traceback at line 11, mark the payload as `error_halt`, and stream the payload to the frontend.
5. **Deterministic API Contracts:** Serve clean, validated Pydantic JSON responses for frontend consumption.

---

## 2. Strict API Endpoint Contracts

### 2.1 `/api/v1/health` [GET]
* **Purpose:** System health, database connection, and sandbox execution readiness check.
* **Response `200 OK`:**

```json
{
  "status": "healthy",
  "version": "1.0.0",
  "sandbox_ready": true,
  "database_connected": true
}
```

---

### 2.2 `/api/v1/auth/register` [POST]
* **Request Payload:**

```json
{
  "email": "user@codescope.io",
  "username": "developer123",
  "password": "SecurePassword123!"
}
```

* **Response `201 Created`:**

```json
{
  "id": "usr_9981aef2",
  "email": "user@codescope.io",
  "username": "developer123",
  "created_at": "2026-11-06T22:00:00Z"
}
```

---

### 2.3 `/api/v1/auth/login` [POST]
* **Request Payload (Form-encoded / JSON):**

```json
{
  "username": "developer123",
  "password": "SecurePassword123!"
}
```

* **Response `200 OK`:**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 86400
}
```

---

### 2.4 `/api/v1/execution/run` [POST] *(Primary Engine Endpoint)*
* **Headers:** `Authorization: Bearer <JWT_TOKEN>` (Optional for guest mode, mandatory for saved sessions)
* **Request Payload:**

```json
{
  "code": "arr = [4, 2, 7, 1]\nfor i in range(len(arr)):\n    for j in range(len(arr) - 1):\n        if arr[j] > arr[j+1]:\n            arr[j], arr[j+1] = arr[j+1], arr[j]",
  "domain_hint": "array",
  "max_steps": 500
}
```

* **Response Case A: Successful Traced Execution (`200 OK`)**

```json
{
  "status": "success",
  "total_steps": 12,
  "detected_domain": "array",
  "frames": [
    {
      "step": 1,
      "line": 1,
      "event": "line",
      "variables": {
        "arr": "[4, 2, 7, 1]"
      },
      "stack": ["<module>"],
      "stdout": ""
    },
    {
      "step": 2,
      "line": 2,
      "event": "line",
      "variables": {
        "arr": "[4, 2, 7, 1]",
        "i": "0"
      },
      "stack": ["<module>"],
      "stdout": ""
    }
  ],
  "error": null
}
```

* **Response Case B: Partial Execution with Runtime Error / Collapse (`200 OK`)**

```json
{
  "status": "error_halt",
  "total_steps": 5,
  "detected_domain": "array",
  "frames": [
    {
      "step": 1,
      "line": 1,
      "event": "line",
      "variables": {
        "arr": "[10, 20, 30]"
      },
      "stack": ["<module>"],
      "stdout": ""
    }
  ],
  "error": {
    "line": 4,
    "exception_type": "IndexError",
    "message": "list index out of range",
    "traceback": "Traceback (most recent call last):\n  File \"<string>\", line 4, in <module>\nIndexError: list index out of range",
    "last_valid_variables": {
      "arr": "[10, 20, 30]",
      "i": "3"
    }
  }
}
```

* **Response Case C: AST Security Block / Malicious Input (`400 Bad Request`)**

```json
{
  "detail": {
    "status": "ast_rejected",
    "error_type": "SecurityViolation",
    "message": "Security Violation: Usage of prohibited module/function 'os' on line 1.",
    "blocked_node": "Import"
  }
}
```

---

## 📁3. Backend End-to-End File Tree

```text
backend/
├── app/
│   ├── __init__.py                # Package initialization marker
│   ├── main.py                    # FastAPI app entrypoint, CORS & error handlers
│   ├── api/
│   │   ├── __init__.py
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── router.py          # V1 endpoint aggregator router
│   │       └── endpoints/
│   │           ├── __init__.py
│   │           ├── auth.py        # Authentication & JWT endpoints
│   │           ├── execution.py   # Primary code trace POST endpoint
│   │           └── health.py      # Server health endpoint
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py              # Settings, Pydantic BaseSettings & env loading
│   │   ├── security.py            # Passlib bcrypt hashing & JWT tokens
│   │   └── database.py            # Async SQLite/PostgreSQL database connection pool
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py                # User database model
│   │   └── session.py           # Workspace session & history model
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── execution.py         # Payload schemas for incoming code requests
│   │   ├── step_response.py       # Schemas for step frames, errors, and JSON stream
│   │   └── user.py                # Auth request/response schemas
│   ├── services/
│   │   ├── __init__.py
│   │   ├── ast_sanitizer.py       # AST parsing & security validation service
│   │   ├── state_tracer.py      # sys.settrace step-by-step frame execution tracer
│   │   └── llm_engine.py        # Placeholder service for Phase 2 AI diagnostic hints
│   └── sandbox/
│       ├── __init__.py
│       └── runner.py              # Isolated execution scope context manager
├── tests/
│   ├── __init__.py
│   ├── conftest.py                # Pytest fixtures & TestClient configuration
│   ├── test_health.py             # Health endpoint tests
│   ├── test_ast_sanitizer.py      # Unit tests for AST sanitizer rule checks
│   ├── test_state_tracer.py       # Unit tests for line-by-line tracer execution
│   └── test_execution_api.py      # Integration tests for /execution/run
├── Dockerfile                     # Python 3.11 Docker container setup
├── pytest.ini                     # Pytest suite configuration
├── requirements.txt               # Backend Python dependencies
├── README.md                      # Backend Master Blueprint (This document)
└── .env.example                 # Environment variables template
```

---

## 4. Sequential Step-by-Step Task Execution Guide

Follow this sequential checklist to build the backend engine. Do not jump ahead until the verification and testing steps for each task are 100% passing.

---

### Task 1: Environment Setup & Project Boilerplate Initialization
* **Target Folder:** `backend/`
* **Files to Create:**
  - `requirements.txt`
  - `.env.example`
  - `.env`
  - `pytest.ini`
  - `app/__init__.py`
  - `app/main.py`
  - `app/api/v1/endpoints/health.py`
  - `app/api/v1/router.py`


  cd backend

# 1. Create a Python virtual environment
python -m venv venv

# 2. Activate virtual environment

.\venv\Scripts\Activate.ps1


# 3. Upgrade pip and install all packages
python -m pip install --upgrade pip
pip install -r requirements.txt

# 4. Start the FastAPI development server
uvicorn app.main:app --reload --port 8000

#### Actions:
1. Populate `backend/requirements.txt`:
   
   ```text
   fastapi==0.104.1
   uvicorn[standard]==0.24.0
   pydantic==2.5.2
   pydantic-settings==2.1.0
   passlib[bcrypt]==1.7.4
   python-jose[cryptography]==3.3.0
   python-multipart==0.0.6
   pytest==7.4.3
   httpx==0.25.2
   ```

2. Create Virtual Environment and install packages:
   
   ```bash
   cd backend
   python -m venv venv
   # On Windows:
   .\venv\Scripts\Activate.ps1
   
   pip install -r requirements.txt
   ```

3. Implement `backend/app/main.py`: Include CORS middleware for `http://localhost:5173` and mount the `/api/v1` router.
4. Implement `backend/app/api/v1/endpoints/health.py`: Create the `GET /health` route returning status `healthy`.

#### Verification & Testing:
* **Run Server:**
  
  ```bash
  uvicorn app.main:app --reload --port 8000
  ```

* **Test manually:** Open `http://localhost:8000/docs` in your browser and execute `GET /api/v1/health`. Ensure it returns `200 OK`.
* **Automated Test:** Create `tests/test_health.py` and run `pytest`.

---

### Task 2: AST Security Sanitizer Implementation
* **Target Folder:** `backend/app/services/`
* **Files to Create/Edit:**
  - `app/services/ast_sanitizer.py`
  - `tests/test_ast_sanitizer.py`

#### Actions:
1. Implement `ASTSanitizer` class using Python's `ast.NodeVisitor` inside `app/services/ast_sanitizer.py`.
2. Define `DISALLOWED_IMPORTS = {'os', 'sys', 'subprocess', 'socket', 'shutil', 'builtins', 'importlib', 'requests', 'urllib'}`.
3. Define `DISALLOWED_BUILTINS = {'eval', 'exec', 'open', '__import__', 'compile'}`.
4. Add check for non-code text input (e.g., if AST parsing fails with a `SyntaxError` on a plain text phrase like `"I love you"`, catch it and throw a clean `ASTValidationError("Input is plain text or invalid Python syntax")`).

#### Verification & Testing:
1. Create unit tests in `tests/test_ast_sanitizer.py`:
   - Test 1: Valid sorting code $\rightarrow$ Pass.
   - Test 2: Code containing `import os; os.system('rm -rf /')` $\rightarrow$ Raise `ASTValidationError`.
   - Test 3: Code containing `eval("5 + 5")` $\rightarrow$ Raise `ASTValidationError`.
   - Test 4: String `"Hello I am developer"` $\rightarrow$ Catch syntax error, raise clean validation error.
2. **Run Pytest:**
   
   ```bash
   pytest tests/test_ast_sanitizer.py
   ```

---

### Task 3: `sys.settrace()` Line-by-Line State Tracer Engine
* **Target Folder:** `backend/app/services/` & `backend/app/sandbox/`
* **Files to Create/Edit:**
  - `app/schemas/step_response.py`
  - `app/sandbox/runner.py`
  - `app/services/state_tracer.py`
  - `tests/test_state_tracer.py`

#### Actions:
1. Create Pydantic response models in `app/schemas/step_response.py`:
   - `StepFrame`: `step` (int), `line` (int), `event` (str), `variables` (Dict[str, str]), `stack` (List[str]).
   - `ExecutionError`: `line` (int), `exception_type` (str), `message` (str), `traceback` (str), `last_valid_variables` (Dict).
   - `ExecutionResult`: `status` (str), `total_steps` (int), `detected_domain` (str), `frames` (List[StepFrame]), `error` (Optional[ExecutionError]).
2. Implement `ExecutionTracer` in `app/services/state_tracer.py`:
   - Use `sys.settrace()` hook to step through code execution.
   - Filter out internal frame variables (ignore variables starting with `__`).
   - Safely convert complex structures into deterministic string representations (e.g., `repr(val)`).
   - Enforce a maximum step threshold (e.g., `max_steps=500`) to prevent infinite loops.
   - Catch exceptions: if code fails at line 11 after working lines 1–10, populate the `frames` up to line 10, set status to `"error_halt"`, and record the exception details at line 11 in the `error` object.

#### Verification & Testing:
1. Create `tests/test_state_tracer.py`:
   - Test A (Happy Path): Run Bubble Sort code snippet. Verify that `frames` contains line-by-line variable updates for `arr`, `i`, `j`.
   - Test B (Runtime Collapse): Run code with an `IndexError` on line 4. Verify `status == "error_halt"`, `frames` has previous steps, and `error.line == 4`.
2. **Run Pytest:**
  
   ```bash
   pytest tests/test_state_tracer.py
   ```

---

### Task 4: Domain Detector Auto-Parser
* **Target Folder:** `backend/app/services/`
* **Files to Edit:**
  - `app/services/ast_sanitizer.py`

#### Actions:
1. Enhance AST Sanitizer to inspect data structures used in the code snippet.
2. Auto-detect domain tags:
   - If AST contains node attributes like `.left`, `.right`, `TreeNode`, `.val` $\rightarrow$ Tag as `"tree"`.
   - If AST contains `.next`, `.prev`, `Node` $\rightarrow$ Tag as `"linked_list"`.
   - If AST contains 2D index accesses `grid[i][j]` or matrix patterns $\rightarrow$ Tag as `"dp_matrix"`.
   - Default fallback $\rightarrow$ Tag as `"array"`.

#### Verification & Testing:
1. Add test assertions in `tests/test_ast_sanitizer.py` verifying domain detection logic.

---

### Task 5: Primary Execution API Endpoint Integration
* **Target Folder:** `backend/app/api/v1/endpoints/`
* **Files to Create/Edit:**
  - `app/schemas/execution.py`
  - `app/api/v1/endpoints/execution.py`
  - `app/api/v1/router.py`
  - `tests/test_execution_api.py`

#### Actions:
1. Define request schema `ExecutionRequest` in `app/schemas/execution.py` (`code: str`, `domain_hint: Optional[str]`, `max_steps: Optional[int]`).
2. Build `POST /api/v1/execution/run` in `app/api/v1/endpoints/execution.py`:
   - **Step 1:** Call `ASTSanitizer.validate(payload.code)`. If invalid, return HTTP `400 Bad Request` with custom error detail.
   - **Step 2:** Pass code to `ExecutionTracer.trace_execution()`.
   - **Step 3:** Return structured `ExecutionResult` payload (`200 OK`).
3. Register endpoint in `app/api/v1/router.py`.

#### Verification & Testing:
1. Run server `uvicorn app.main:app --reload`.
2. Open Swagger UI at `http://localhost:8000/docs`.
3. Submit sample POST request to `/api/v1/execution/run` with Python code.
4. Run full integration suite:
   
   ```bash
   pytest tests/test_execution_api.py
   ```

---

### Task 6: Auth & Session Module (Optional for Local Phase 1 MVP, Required for Production)
* **Target Folder:** `backend/app/core/`, `app/models/`, `app/api/v1/endpoints/`
* **Files to Create/Edit:**
  - `app/core/security.py`
  - `app/core/database.py`
  - `app/models/user.py`
  - `app/schemas/user.py`
  - `app/api/v1/endpoints/auth.py`

#### Actions:
1. Set up password hashing with Passlib (`bcrypt`) and JWT token generation using `python-jose` in `app/core/security.py`.
2. Implement `/register` and `/login` routes in `app/api/v1/endpoints/auth.py`.

#### Verification & Testing:
1. Execute `pytest` across all test modules:
   
   ```bash
   pytest -v
   ```

---

## 5. Master Verification & Command Cheat Sheet

| Action | Command | Expected Output |
| :--- | :--- | :--- |
| **Activate Virtual Env** | `.\venv\Scripts\Activate.ps1` (Win) / `source venv/bin/activate` (Mac) | `(venv)` prefix in terminal |
| **Start Backend Dev Server** | `uvicorn app.main:app --reload --port 8000` | `Uvicorn running on http://127.0.0.1:8000` |
| **View OpenAPI Docs** | Open browser to `http://localhost:8000/docs` | Interactive Swagger UI |
| **Run All Unit & Integration Tests** | `pytest -v` | All tests marked `PASSED` |
| **Run Specific Test Suite** | `pytest tests/test_state_tracer.py -v` | State tracer tests `PASSED` |