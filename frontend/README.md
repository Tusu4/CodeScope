# CodeScope Frontend: Workspace UI, Visual Canvases & State Management Specification

> **Phase 1 Production Blueprint & Step-by-Step Task Execution Guide**  
> Target Stack: React 18 | TypeScript | Vite | Tailwind CSS | Zustand | Monaco Editor | Lucide Icons

---

## 1. Frontend Architecture & System Responsibilities

The CodeScope frontend is an interactive, state-driven visual debugging workbench. It receives deterministic step-frame snapshots from the backend API and transforms them into time-travelable visual renders.

### Core Responsibilities:
1. **Interactive Workspace Layout (`pages/WorkspacePage.tsx`):** A split-pane IDE interface housing the Monaco Code Editor, Visual Debugging Canvas, Pseudocode Line Highlighter, Variable Inspector, and Playback Controls.
2. **Client-Side AST Pre-Flight Inspection (`utils/astValidator.ts`):** Fast client-side regex and syntax checks to block non-code inputs (e.g., plain English text like `"I love you"`) before firing HTTP requests.
3. **Global Execution State Store (`store/useExecutionStore.ts`):** Manage visualization timeline states: current step frame, playback index, play/pause status, playback speed, and partial-execution error flags.
4. **Dynamic Canvas Rendering (`components/visualizer/`):** Domain-driven UI components that translate variable state dictionaries into visually intuitive data structures:
   - `ArrayCanvas.tsx`: 1D arrays, 2D grids, pointer markers (`i`, `j`).
   - `LinkedListCanvas.tsx`: Linked node boxes and pointer references (`head`, `curr`, `next`).
   - `TreeCanvas.tsx`: SVG hierarchical tree graphs and BST nodes (`root`, `.left`, `.right`).
   - `GraphCanvas.tsx`: Network node-edge layout visualizers.
   - `DPMatrixCanvas.tsx`: Dynamic programming table cell-by-cell fills.
5. **Partial-Execution & Error Signal UI (`ErrorHaltBadge.tsx`):** Render a visual red breakdown marker when code fails mid-execution, highlighting the last valid memory heap and pinpointing the exact broken instruction line.

---

## 2. Strict API Payload Contracts (Expected Backend Schema)

The frontend expects the following deterministic JSON schema from `POST /api/v1/execution/run`:

```typescript
export interface ExecutionStepFrame {
  step: number;
  line: number;
  event: 'line' | 'call' | 'return' | 'exception';
  variables: Record<string, string>; // e.g., { "arr": "[4, 2, 7, 1]", "i": "0", "j": "1" }
  stack: string[];
  stdout: string;
}

export interface ExecutionErrorPayload {
  line: number;
  exception_type: string;
  message: string;
  traceback: string;
  last_valid_variables: Record<string, string>;
}

export interface ExecutionResponsePayload {
  status: 'success' | 'error_halt' | 'ast_rejected';
  total_steps: number;
  detected_domain: 'array' | 'linked_list' | 'tree' | 'graph' | 'dp_matrix';
  frames: ExecutionStepFrame[];
  error: ExecutionErrorPayload | null;
}
```

---

## 📁 3. Frontend End-to-End File Tree

```text
frontend/
├── public/                      # Static branding assets & favicon
│   └── favicon.ico
├── src/
│   ├── assets/                  # Tailwind base styles and canvas animations
│   │   └── global.css
│   ├── components/
│   │   ├── common/              # Layout and Navigation
│   │   │   ├── Navbar.tsx       # Platform header navigation
│   │   │   ├── Footer.tsx       # Platform footer
│   │   │   └── ProtectedRoute.tsx# Auth route guard wrapper
│   │   ├── editor/              # Monaco Code Editor & Validator
│   │   │   ├── CodeEditor.tsx   # Monaco Editor instance wrapper
│   │   │   └── ASTWarningBanner.tsx # Non-code / Malicious input alert overlay
│   │   ├── visualizer/          # Interactive Visual Canvases & Debugger Controls
│   │   │   ├── ArrayCanvas.tsx  # 1D array & 2D matrix visualizer
│   │   │   ├── LinkedListCanvas.tsx # Pointer & node reference visualizer
│   │   │   ├── TreeCanvas.tsx   # Binary tree SVG renderer
│   │   │   ├── GraphCanvas.tsx  # Adjacency node-edge network renderer
│   │   │   ├── DPMatrixCanvas.tsx # 2D dynamic programming grid renderer
│   │   │   ├── DebugControls.tsx# Play, Pause, Step Next/Prev, Speed Slider
│   │   │   ├── PseudocodeBox.tsx# Active execution line highlighter
│   │   │   ├── VariableInspector.tsx# Live variable inspector table
│   │   │   └── ErrorHaltBadge.tsx # Red-signal halt badge at point of failure
│   │   └── ai/                  # Phase 2 LLM UI Hooks (Placeholder for Phase 1)
│   │       ├── LLMHintPanel.tsx # AI debugging explanation sidebar
│   │       └── DiagnosticCard.tsx# Step-by-step fix recommendation card
│   ├── pages/                   # Application View Routes
│   │   ├── LandingPage.tsx      # Public landing page
│   │   ├── LoginPage.tsx        # Login portal
│   │   ├── RegisterPage.tsx     # User registration portal
│   │   └── WorkspacePage.tsx    # Split-pane IDE workspace view
│   ├── services/                # Axios Network Clients
│   │   ├── api.ts               # Base Axios client with authorization headers
│   │   └── executionService.ts  # Code execution API endpoint wrapper
│   ├── store/                   # Zustand Global State Management
│   │   ├── useAuthStore.ts      # Authentication state slice
│   │   ├── useExecutionStore.ts # Execution frames, current step & playback store
│   │   └── useUIStore.ts        # UI themes, panel toggles, active domain tab
│   ├── types/                   # TypeScript Types
│   │   ├── execution.ts         # Step frames, errors, and domain interfaces
│   │   └── user.ts                # User profile and session types
│   ├── utils/                   # Client-Side Helper Utilities
│   │   ├── astValidator.ts      # Client-side regex code validator
│   │   └── formatters.ts        # String-to-array/object parser utilities
│   ├── App.tsx                  # App router switch
│   └── main.tsx                 # Vite DOM mount entrypoint
├── Dockerfile                   # Production Nginx frontend container definition
├── index.html                   # HTML entrypoint
├── package.json                 # Dependencies (React, Monaco, Zustand, Lucide, Tailwind)
├── postcss.config.js            # PostCSS configuration for Tailwind
├── tailwind.config.js           # Tailwind CSS design tokens
├── tsconfig.json                # TypeScript strict configuration
├── vite.config.ts               # Vite bundler options & backend dev proxy
└── .env.example                 # Environment variables template
```

---

## 🛠️ 4. Sequential Step-by-Step Task Execution Guide

Execute these tasks in order. Verify each task before moving to the next.

---

### Task 1: Vite + React + TypeScript + Tailwind Setup
* **Target Folder:** `frontend/`
* **Files to Create/Edit:**
  - `package.json`
  - `vite.config.ts`
  - `tailwind.config.js`
  - `src/assets/global.css`
  - `src/main.tsx`
  - `src/App.tsx`

#### Actions:
1. Initialize project dependencies in `frontend/package.json`:
   ```json
   {
     "name": "codescope-frontend",
     "private": true,
     "version": "1.0.0",
     "type": "module",
     "scripts": {
       "dev": "vite",
       "build": "tsc && vite build",
       "preview": "vite preview"
     },
     "dependencies": {
       "@monaco-editor/react": "^4.6.0",
       "axios": "^1.6.2",
       "clsx": "^2.0.0",
       "lucide-react": "^0.292.0",
       "react": "^18.2.0",
       "react-dom": "^18.2.0",
       "react-router-dom": "^6.20.0",
       "tailwind-merge": "^2.0.0",
       "zustand": "^4.4.7"
     },
     "devDependencies": {
       "@types/node": "^20.10.0",
       "@types/react": "^18.2.37",
       "@types/react-dom": "^18.2.15",
       "@vitejs/plugin-react": "^4.2.0",
       "autoprefixer": "^10.4.16",
       "postcss": "^8.4.31",
       "tailwindcss": "^3.3.5",
       "typescript": "^5.2.2",
       "vite": "^5.0.0"
     }
   }
   ```
2. Install packages:
   ```bash
   cd frontend
   npm install
   ```
3. Configure `vite.config.ts` with API proxy mapping `/api/v1` to `http://localhost:8000`.

#### Verification:
```bash
npm run dev
```
Open `http://localhost:5173` in your browser and confirm Vite renders cleanly without errors.

---

### Task 2: TypeScript Interfaces & Global Zustand Stores
* **Target Folder:** `frontend/src/types/` & `frontend/src/store/`
* **Files to Create:**
  - `src/types/execution.ts`
  - `src/store/useExecutionStore.ts`
  - `src/store/useUIStore.ts`

#### Actions:
1. Create `src/types/execution.ts` defining `ExecutionStepFrame`, `ExecutionErrorPayload`, `ExecutionResponsePayload`, and `DomainType`.
2. Implement `src/store/useExecutionStore.ts` using Zustand to hold:
   - `frames: ExecutionStepFrame[]`
   - `currentStepIndex: number`
   - `isPlaying: boolean`
   - `speedMs: number`
   - `errorState: ExecutionErrorPayload | null`
   - Actions: `setExecutionResult()`, `nextStep()`, `prevStep()`, `togglePlay()`, `setStepIndex()`, `reset()`.

#### Verification:
Import `useExecutionStore` into `App.tsx` and verify state updates cleanly during hot reloading.

---

### Task 3: Client-Side AST Pre-Validator Utility & Alert Banner
* **Target Folder:** `frontend/src/utils/` & `frontend/src/components/editor/`
* **Files to Create:**
  - `src/utils/astValidator.ts`
  - `src/components/editor/ASTWarningBanner.tsx`

#### Actions:
1. Implement `validateCodeInput(code: string)` in `src/utils/astValidator.ts`:
   - Checks if string is non-code plain text (e.g., checks word counts without Python operators/keywords).
   - Rejects illegal tokens locally before network transmission.
2. Build `ASTWarningBanner.tsx`: Render an amber alert banner over the editor when pre-validation fails.

---

### Task 4: Monaco Code Editor Integration & Line Highlighter
* **Target Folder:** `frontend/src/components/editor/`
* **Files to Create:**
  - `src/components/editor/CodeEditor.tsx`

#### Actions:
1. Wrap `@monaco-editor/react` in `CodeEditor.tsx`.
2. Sync active line highlighting with `currentStepIndex` from `useExecutionStore`:
   - Highlight current line in blue during valid steps.
   - Highlight line in red if `currentStepIndex` reaches an `errorState.line`.

#### Verification:
Pass sample python code string to `CodeEditor.tsx` and verify line decorations update as `currentStepIndex` changes.

---

### Task 5: Debug Playback Controls & Variable Inspector
* **Target Folder:** `frontend/src/components/visualizer/`
* **Files to Create:**
  - `src/components/visualizer/DebugControls.tsx`
  - `src/components/visualizer/VariableInspector.tsx`
  - `src/components/visualizer/ErrorHaltBadge.tsx`

#### Actions:
1. Build `DebugControls.tsx`: Include Play, Pause, Step Forward, Step Backward, Reset buttons, and Speed Slider (100ms - 2000ms).
2. Build `VariableInspector.tsx`: Render a live table listing variable names and formatted values for the current frame step.
3. Build `ErrorHaltBadge.tsx`: Display a glowing red warning card showing exception type, message, line number, and raw traceback when execution halts due to a runtime error.

---

### Task 6: Visual Canvases (Domain Renderers)
* **Target Folder:** `frontend/src/components/visualizer/`
* **Files to Create:**
  - `src/components/visualizer/ArrayCanvas.tsx`
  - `src/components/visualizer/LinkedListCanvas.tsx`
  - `src/components/visualizer/TreeCanvas.tsx`
  - `src/components/visualizer/DPMatrixCanvas.tsx`

#### Actions:
1. Build `ArrayCanvas.tsx`: Parse array string values (e.g., `"[4, 2, 7, 1]"`) into vertical bar charts or index boxes with active pointer tags (`i`, `j`).
2. Build `LinkedListCanvas.tsx`: Parse node objects and render linked boxes connected with SVG arrows (`head -> node1 -> node2`).
3. Build `TreeCanvas.tsx`: Parse tree structures and render interactive node-link trees (`root`, `.left`, `.right`).
4. Build `DPMatrixCanvas.tsx`: Parse 2D array strings into dynamic grid cells highlighting active read/write cells.

---

### Task 7: Split-Pane Workspace Assembly & Backend API Wiring
* **Target Folder:** `frontend/src/services/` & `frontend/src/pages/`
* **Files to Create:**
  - `src/services/api.ts`
  - `src/services/executionService.ts`
  - `src/pages/WorkspacePage.tsx`

#### Actions:
1. Configure Axios client in `src/services/api.ts`.
2. Implement `executeCode(code, domainHint)` in `src/services/executionService.ts` to call `POST /api/v1/execution/run`.
3. Assemble complete IDE in `src/pages/WorkspacePage.tsx`:
   - Left Pane: Monaco Code Editor + `ASTWarningBanner`.
   - Right Top Pane: Active Domain Canvas + `ErrorHaltBadge`.
   - Right Bottom Pane: `VariableInspector` + `PseudocodeBox`.
   - Bottom Dock: `DebugControls`.

#### Verification & Testing:
1. Start FastAPI backend (`uvicorn app.main:app --reload --port 8000`).
2. Start Vite frontend (`npm run dev`).
3. Type Bubble Sort code into the Monaco Editor and click **Run & Debug**.
4. Step through execution forward and backward using `DebugControls`. Verify array bars re-order smoothly.
5. Intentionally introduce an `IndexError` on line 10. Click **Run & Debug**. Verify that steps 1–9 play successfully, the canvas freezes at line 10, and `ErrorHaltBadge.tsx` flashes red with the traceback breakdown.

---

## 5. Master Command Cheat Sheet

| Action                        | Command           | Expected Output                            |
| :---                          | :---              | :---                                       |
| **Install Dependencies**      | `npm install`     | All node modules installed cleanly         |
| **Start Frontend Dev Server** | `npm run dev`     | `Local: http://localhost:5173`             |
| **Build for Production**      | `npm run build`   | Clean TypeScript compilation & dist folder |
| **Preview Build**             | `npm run preview` | Production bundle running locally          |
