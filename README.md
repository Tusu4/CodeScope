### CodeScope: The Enterprise-Grade Interactive Visual Debugger & Error Engine
**Master Architectural Specification & Engineering Blueprint**

> **Abstract:** Standard algorithm visualization platforms are built on rigid, "happy-path" assumptions—requiring pristine, bug-free user code to render visual outputs, and failing with opaque stack traces when errors occur. 

**CodeScope** shatters this limitation by introducing a dual-layer **Execution Tracer and Diagnostic Engine**. It parses user code, evaluates partial-execution correctness thresholds, executes step-by-step memory tracking up to failure points, highlights exact breakdown lines with visual red signals, and integrates zero-switch AI diagnostics to keep developers inside a singular, immersive workspace.

---

## Table of Contents
1. [Core Architectural Philosophy & Vision]
2. [Comprehensive 4-Phase Roadmap & Evolution Strategy]
3. [Domain-Wise DSA & Algorithmic Problem-Solving Stack]
4. [End-to-End System Data Flow & Execution Pipeline]
5. [Modular Enterprise Directory Structure]
6. [Deep-Dive Phase Specifications & Algorithmic Mechanics]
7. [Hardware Environment Setup & Multi-Laptop Synchronization]
---

## 1. Core Architectural Philosophy & Vision

### The Paradox of Traditional Visualizers
Traditional code visualizers assume that users already know how to write correct code. If a student or developer types 15 lines of dictionary operations—where lines 1–10 successfully assign temporary variables and sort keys, but lines 11–15 suffer from a reference mismatch or index failure—traditional tools refuse to run or crash outright with an unformatted red error log. 

**The CodeScope Paradigm Shift:**
* **Learning Through Failure:** Developers learn faster by visualizing *where, why, and how* their logic breaks, rather than just watching pre-baked animations of code that already works.
* **Partial-Execution Thresholds:** CodeScope allows execution up to a logic validity threshold. If a script achieves partial structural and execution validity, the engine traces every variable state change up to the exact moment of failure.
* **The Visual Halt Signal:** Instead of crashing silently, CodeScope freezes the canvas at the precise instruction pointer where the exception or logic mismatch occurs, rendering a vivid warning marker alongside the last valid memory heap and local stack frame.

---

## 2. Comprehensive 4-Phase Roadmap & Evolution Strategy

CodeScope is engineered using a modular, decoupled architecture that allows seamless scaling from a local browser-based execution trace tool to an enterprise-grade cloud service.

[ Phase 1: Core Sandbox & Tracer ] 
       │
       ▼ (Modular Expansion)
[ Phase 2: AI Diagnostic Copilot ]
       │
       ▼ (Containerization)
[ Phase 3: Docker Resource Isolation ]
       │
       ▼ (Cloud Scaling)
[ Phase 4: AWS Production Architecture ]


### Phase 1: Core Python Sandbox & Line-by-Line State Tracer (Foundation)
**Client-Side Pre-Validation:** Monaco editor input interception combined with client-side AST token checks to instantly reject non-code strings (e.g., plain text or chat prompts) before reaching network transport layers.
**Backend AST Sanitization:** Python server-side Abstract Syntax Tree parsing that strips hazardous system modules (`os`, `sys`, `subprocess`, file writing primitives, infinite loops without exit checks).
**`sys.settrace()` Execution Engine:** A custom-built Python tracing wrapper that intercepts frame execution events (`call`, `line`, `return`, `exception`), capturing variable states, local dictionaries, and heap references after every executed line.
**Visual Canvas Engine:** React + Zustand rendering pipeline that converts raw step-by-step frame arrays into interactive visual components (Arrays, Dictionaries, Linked Lists, Trees, Graphs).

### Phase 2: Integrated AI Diagnostic Copilot (Retention Engine)
* **Zero-Tab-Switching AI:** Eliminates developer friction by embedding a contextual diagnostic panel directly adjacent to the visual failure marker.
* **Contextual Prompt Engineering:** When execution halts at a failure boundary, the backend packages the code snippet, traceback message, and last valid state into a structured JSON payload sent to an LLM provider (e.g., GPT-4o-mini or local open-source weights).
* **Actionable Fix Suggestions:** Returns plain-language explanations ("*Your temporary dictionary reference was lost during reassignment because the key was overwritten on line 11*") along with syntax correction diffs.

### Phase 3: Docker Containerization & Security Isolation
* **Ephemeral Sandboxing:** Migrates local process execution to disposable, lightweight Docker containers spawned on-demand via the FastAPI backend.
* **Resource Guardrails:** Implements hard constraints on container execution time (e.g., 3-second timeout limit to thwart infinite loops) and memory allocation (capped at 128MB RAM) to safeguard host infrastructure against denial-of-service attacks.

### Phase 4: Cloud Infrastructure & AWS Production Deployment
* **Decoupled Cloud Topology:** Static frontend dashboard deployment via Amazon S3 and CloudFront CDN.
* **Scalable Compute Clusters:** Backend API and containerized worker nodes deployed on AWS ECS (Elastic Container Service) with Fargate, managed behind an API Gateway with automated token-bucket rate limiting.

---

## 3. Domain-Wise DSA & Algorithmic Problem-Solving Stack

CodeScope shifts away from hardcoded algorithm animations in favor of **Domain-Wise Flexible Canvas Simulation**. The platform auto-detects or routes execution traces into specialized visual domain environments:

### 3.1 Data Structure Domains
* **Array & Matrix Domain (`ArrayCanvas.tsx`):**
  * **1D Dynamic Arrays:** Variable length changes, element swaps, two-pointer convergence, and sliding window boundaries.
  * **2D Grids / Matrices:** Row-major/column-major traversals, matrix rotations, spiral paths, and grid-based state tracking.
* **Linked Structure Domain (`LinkedListCanvas.tsx`):**
  * **Singly & Doubly Linked Lists:** Node memory allocations, `.next` and `.prev` pointer reassignments, cycle detection (Floyd's Tortoise & Hare), and list reversals.
* **Hierarchical Tree Domain (`TreeCanvas.tsx`):**
  * **Binary Trees & Binary Search Trees (BST):** Recursive traversals (Inorder, Preorder, Postorder, Level-Order), BST insertions/deletions, and tree depth/height tracking.
  * **Heaps & Priority Queues:** Min-Heap/Max-Heap array transformations, `heapify` steps, and root extractions.
* **Graph Network Domain (`GraphCanvas.tsx`):**
  * **Adjacency Lists & Adjacency Matrices:** Directed/undirected graph traversals, active vertex highlights, edge weight updates, and visited set representations.
* **Key-Value & Hash Domain (`HashMapCanvas.tsx`):**
  * **Dictionaries & Sets:** Key-value insertions, hash collisions, frequency counting maps, and bucket lookups.
* **Tabular / DP Domain (`DPTableCanvas.tsx`):**
  * **1D & 2D Memoization Grids:** Cell-by-cell state generation for dynamic programming problems.

### 3.2 Algorithmic Problem Classes Handled
* **Searching & Sorting:** Binary Search pointer movements, Linear Search, Bubble Sort, Selection Sort, Insertion Sort, Merge Sort divide-and-conquer partitions, and Quick Sort pivot partitioning.
* **Recursion & Backtracking:** Visualizing stack frame expansion/contraction, base-case hits, and state rollbacks (e.g., N-Queens, Subsets, Permutations).
* **Graph Traversals & Shortest Path:** Breadth-First Search (BFS queue states), Depth-First Search (DFS recursion stacks), and Dijkstra’s algorithm edge relaxations.
* **Dynamic Programming:** Classical DP problems including 0/1 Knapsack, Longest Common Subsequence (LCS), Coin Change, and Minimum Path Sum.

---

## 4. End-to-End System Data Flow & Execution Pipeline

Understanding the exact lifecycle of a user code submission from keystroke to visual canvas render requires tracing through the multi-tier validation and execution pipeline:

1. User types code in Monaco Editor (React Frontend)
   │
   ▼
2. Client-Side AST Pre-Validator (astValidator.ts)
   ├── IF input contains plain text / non-code ("I love you"):
   │     └── Triggers ASTWarningBanner.tsx (Stops execution locally)
   └── IF input is valid Python syntax:
         └── Sends HTTP POST /api/v1/execution/run with JWT headers
   │
   ▼
3. FastAPI Backend Gateway (app/api/v1/endpoints/execution.py)
   ├── Authenticates token & verifies rate limits
   └── Passes code string to AST Sanitizer Service
   │
   ▼
4. AST Security Sanitizer Service (app/services/ast_sanitizer.py)
   ├── Parses code into Abstract Syntax Tree
   ├── Rejects forbidden nodes (Import, FunctionDef hazards, System calls)
   └── Passes sanitized AST node tree to Execution Sandbox
   │
   ▼
5. Python Execution Sandbox & Tracer (app/services/state_tracer.py)
   ├── Attaches sys.settrace() callback hook
   ├── Executes code line-by-line in a controlled namespace frame
   ├── Captures variable states, memory mappings, and call stacks per line
   ├── CASE A (Success): Returns complete array of sequential state frames
   └── CASE B (Exception/Failure): Captures last valid state frame + raw traceback
   │
   ▼
6. Response Serialization & Stream Back
   └── Pydantic Schema packages execution frames and error status codes
   │
   ▼
7. Frontend Zustand Execution Store (useExecutionStore.ts)
   ├── Receives frame array and updates playback timeline
   └── IF error status detected:
         └── Triggers Phase 2: Async call to LLM Diagnostic Engine (/api/v1/llm/diagnose)
   │
   ▼
8. Visual Canvas Renderer & UI Feedback (React Components)
   ├── Renders step-by-step animation controls (Play, Pause, Step, Slider)
   ├── Highlights active execution line in Monaco Editor
   └── Displays Red Halt Badge & AI Diagnostic Hint Panel at point of failure

---

## 5. Modular Enterprise Directory Structure

codescope/
├── .github/
│   └── workflows/
│       ├── frontend-ci.yml              # Automated React build, lint & test pipeline
│       └── backend-ci.yml               # Automated Python test suite & Docker build check
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   └── v1/
│   │   │       ├── endpoints/
│   │   │       │   ├── auth.py          # User registration, login & JWT token issuance
│   │   │       │   ├── execution.py     # Code payload intake, AST validation & tracer trigger
│   │   │       │   ├── llm.py           # Phase 2: AI diagnostic hint generation endpoints
│   │   │       │   └── health.py        # System heartbeat, DB connection & sandbox readiness check
│   │   │       └── router.py            # Aggregator router prefixing all v1 API endpoints
│   │   ├── core/
│   │   │   ├── config.py            # Pydantic BaseSettings management for environment variables
│   │   │   ├── security.py          # Password hashing (bcrypt) & JWT signing/verification
│   │   │   └── database.py          # SQLAlchemy session maker & database connection pooling
│   │   ├── models/                  # SQLAlchemy Database ORM Models
│   │   │   ├── user.py              # User account schema (ID, hashed_password, created_at)
│   │   │   └── session.py           # Saved debug session records and code history snapshots
│   │   ├── schemas/                 # Pydantic Data Transfer Validation Schemas
│   │   │   ├── execution.py     # Request/Response models for code frames, variables, & errors
│   │   │   ├── llm.py           # Diagnostic prompt and AI response payload schemas
│   │   │   └── user.py              # Registration and login validation schemas
│   │   ├── services/                # Core Business Logic & Middleware Services
│   │   │   ├── ast_sanitizer.py     # AST parsing engine blocking unsafe modules & imports
│   │   │   ├── state_tracer.py      # sys.settrace engine capturing variable mutations per line
│   │   │   └── llm_engine.py        # OpenAI/Anthropic/Local LLM wrapper for diagnostic hints
│   │   ├── sandbox/                 # Isolated Execution Enclosures
│   │   │   ├── runner.py            # Local process spawner for Phase 1 execution
│   │   │   └── docker_executor.py   # Phase 3: Ephemeral Docker container sandbox wrapper
│   │   └── main.py                  # FastAPI application entrypoint, CORS & exception middleware
│   ├── tests/                       # Pytest Automated Test Suite
│   │   ├── test_ast.py              # Security test cases verifying malicious code rejection
│   │   ├── test_tracer.py           # Execution tracer verification across loops and functions
│   │   └── test_execution_api.py    # Integration tests for FastAPI endpoints
│   ├── Dockerfile                     # Multi-stage production Dockerfile for Python backend
│   ├── pytest.ini                     # Pytest suite config
│   ├── requirements.txt               # Pinned Python package dependencies (FastAPI, Pydantic, etc.)
│   └── .env.example                 # Template environment variables file
├── frontend/
│   ├── public/                      # Static assets, branding logos & favicons
│   ├── src/
│   │   ├── assets/                  # Global CSS styles, Tailwind configurations & SVGs
│   │   ├── components/
│   │   │   ├── common/              # Shared Layout & Structural Components
│   │   │   │   ├── Navbar.tsx       # Main platform navigation header and profile menu
│   │   │   │   ├── Footer.tsx       # Platform copyright and documentation links
│   │   │   │   └── ProtectedRoute.tsx# Higher-order component guarding authenticated routes
│   │   │   ├── editor/              # Monaco Editor & Input Validation Components
│   │   │   │   ├── CodeEditor.tsx   # Monaco editor instance wrapper with theme bindings
│   │   │   │   └── ASTWarningBanner.tsx # Instant UI banner for invalid/non-code string entries
│   │   │   ├── visualizer/          # Interactive Visual Canvas Engines
│   │   │   │   ├── ArrayCanvas.tsx  # Dynamic 1D array & 2D matrix state visualizer
│   │   │   │   ├── LinkedListCanvas.tsx # Pointer, node, & reference link renderer
│   │   │   │   ├── TreeCanvas.tsx   # Binary tree, BST, & AVL structural renderer
│   │   │   │   ├── GraphCanvas.tsx  # Node and weighted edge adjacency visualizer
│   │   │   │   ├── DPMatrixCanvas.tsx # 2D dynamic programming grid fill renderer
│   │   │   │   ├── DebugControls.tsx# Play, Pause, Step Forward/Back, & Speed slider bar
│   │   │   │   └── ErrorHaltBadge.tsx # Line-break red signal marker & traceback inspector
│   │   │   └── ai/                  # Phase 2 AI Integration Components
│   │   │       ├── LLMHintPanel.tsx # Sliding sidebar displaying contextual AI fixes
│   │   │       └── DiagnosticCard.tsx# Structured card formatting step-by-step repair guides
│   │   ├── pages/                   # Main Application View Routes
│   │   │   ├── LandingPage.tsx      # Public landing page showcasing product value proposition
│   │   │   ├── LoginPage.tsx        # User login authentication portal
│   │   │   ├── RegisterPage.tsx     # New user account registration portal
│   │   │   └── WorkspacePage.tsx    # Split-pane IDE workspace (Editor + Visualizer + AI)
│   │   ├── services/                # Axios API Communication Wrappers
│   │   │   ├── api.ts               # Base Axios client with interceptors for JWT injection
│   │   │   ├── executionService.ts  # API wrapper for sending code submissions and fetching frames
│   │   │   └── aiService.ts         # API wrapper for requesting LLM error diagnostics
│   │   ├── store/                   # Zustand Global State Management Slices
│   │   │   ├── useAuthStore.ts      # Authentication tokens and user profile state
│   │   │   ├── useExecutionStore.ts # Execution frames, current playback index, & pause states
│   │   │   └── useUIStore.ts        # Theme settings, panel collapsing, & modal toggles
│   │   ├── types/                   # TypeScript Type Definitions & Interfaces
│   │   │   ├── execution.ts         # Strict interfaces for step frames, variables, & errors
│   │   │   └── user.ts                # User profile and auth state type definitions
│   │   ├── utils/                   # Client-Side Helper Utilities
│   │   │   ├── astValidator.ts      # Fast client-side pre-flight check filtering plain text
│   │   │   └── formatters.ts        # Variable state formatters and memory address serializers
│   │   ├── App.tsx                  # Root React component managing router switch
│   │   └── main.tsx                 # Vite application entrypoint and DOM renderer
│   ├── Dockerfile                     # Nginx multi-stage container build for frontend
│   ├── package.json                 # Node dependencies (React, Zustand, Monaco, Lucide, etc.)
│   ├── tsconfig.json                # TypeScript compiler strict configurations
│   ├── vite.config.ts               # Vite bundler options and backend proxy mappings
│   └── .env.example                 # Frontend environment variables template
├── docker-compose.yml               # Multi-container local orchestration (Backend + Frontend)
├── .gitignore                       # Global ignore rules for node_modules, .venv, build artifacts
└── README.md                        # Master Project Specification Blueprint


---

## 6. Deep-Dive Phase Specifications & Algorithmic Mechanics

### A. The AST Security Sanitizer (`backend/app/services/ast_sanitizer.py`)
To prevent malicious code execution on the server or sandbox environment, every submitted snippet passes through an AST security inspection before execution:
1. **Node Visitor Pattern:** Uses Python's built-in `ast.NodeVisitor` to traverse every syntax node in the user script.
2. **Blocked Node Classes:** Automatically raises a security exception if it encounters:
   * `ast.Import` / `ast.ImportFrom` (Blocking arbitrary module imports like `os`, `sys`, `subprocess`, `socket`).
   * `ast.Call` targeting dangerous built-ins (e.g., `eval`, `exec`, `open`, `__import__`).
   * `ast.Global` or `ast.Nonlocal` modifications targeting internal server state.

### B. The Line-by-Line State Tracer (`backend/app/services/state_tracer.py`)
The heart of Phase 1 execution tracking relies on CPython's debugging hooks:
```python
import sys
import traceback

class ExecutionTracer:
    def __init__(self):
        self.frames_log = []

    def trace_calls(self, frame, event, arg):
        if event == 'line':
            lineno = frame.f_lineno
            # Capture local variables, excluding built-ins
            local_vars = {k: repr(v) for k, v in frame.f_locals.items() if not k.startswith('__')}
            self.frames_log.append({
                "line": lineno,
                "variables": local_vars,
                "event": event
            })
        return self.trace_calls
```

* **Frame Serialization:** Non-serializable objects (like open file handles or custom class instances) are safely formatted into string representations so JSON payloads do not fail during transport.
* **Exception Capture:** If an exception occurs mid-execution, the tracer catches the stack frame, extracts the exact line number of failure, serializes the traceback message, and sets the `status` flag to `"error_halt"`.

### C. The Visual Red Signal & Halt Badge (`frontend/src/components/visualizer/ErrorHaltBadge.tsx`)
When the backend returns an execution response containing an error halt flag:
* The Zustand store (`useExecutionStore`) pauses playback automatically at the final valid index.
* The Monaco Editor underlines the exact offending line in red.
* The Canvas renders a glowing **Red Halt Badge** detailing the runtime exception (`IndexError: list index out of range`).
* Simultaneously, an asynchronous fetch request is dispatched to the Phase 2 AI endpoint (`/api/v1/llm/diagnose`), loading a human-readable explanation into the `LLMHintPanel.tsx` sidebar.

---

## 7. Hardware Environment Setup & Multi-Laptop Synchronization

Because CodeScope is built collaboratively across two developer laptops, standardized environment configuration ensures 100% parity between local testing and production deployment.

### Step 1: Clone & Repository Initialization
```bash
git clone https://github.com/your-username/codescope.git
cd codescope
```

### Step 2: Docker Desktop Installation (Recommended for Containerized Sandbox)
1. Navigate to [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop).
2. Download the installer for your operating system (Windows Intel/AMD or Apple Silicon / Mac). *File size: ~800MB.*
3. Run the installer, ensure **WSL 2 integration** is enabled (Windows), and restart your machine.
4. Verify installation via terminal:
   ```
   docker --version
   docker compose version
   ```

### Step 3: Node.js (v20+ LTS) & Python (v3.11+) Installation
* **Node.js:** Download and install from [nodejs.org](https://nodejs.org/). Verify: `node -v` and `npm -v`.
* **Python:** Download and install from [python.org](https://python.org/downloads/). **Crucial:** Check the box **"Add Python to PATH"** during installation. Verify: `python --version` and `pip --version`.

### Step 4: Local Multi-Container Orchestration
To spin up both the FastAPI backend and React frontend simultaneously with live hot-reloading:
```bash
docker compose up --build
```
* **Frontend Dashboard:** Accessible at `http://localhost:5173`
* **Backend API & Swagger Docs:** Accessible at `http://localhost:8000/docs