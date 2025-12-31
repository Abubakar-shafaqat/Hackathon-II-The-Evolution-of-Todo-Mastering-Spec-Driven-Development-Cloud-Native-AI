<!--
SYNC IMPACT REPORT
==================
Version Change: 1.0.0 → 2.0.0 (MAJOR - Backward Incompatible)

Modified Sections:
  - PROJECT PURPOSE: Console app → Full-stack web application
  - PHASE I TECHNICAL CONSTRAINTS → PHASE II TECHNICAL CONSTRAINTS: Python stdlib → Next.js + FastAPI + PostgreSQL
  - 5 BASIC FEATURES: Console operations → Web UI with authentication
  - CODE QUALITY STANDARDS: Python-only → TypeScript + Python standards
  - DATA MODEL: In-memory dataclass → PostgreSQL schema with user isolation
  - USER EXPERIENCE STANDARDS: Console interface → Web responsive design
  - TESTING STANDARDS: unittest → React Testing Library + pytest + E2E
  - SUCCESS CRITERIA: Phase I console → Phase II web deployment

Added Sections:
  - AUTHENTICATION ARCHITECTURE: User registration, login, JWT flow, token refresh
  - DATABASE SCHEMA REQUIREMENTS: PostgreSQL tables (users, tasks) with indexes
  - SECURITY REQUIREMENTS: Password hashing, JWT secrets, CORS, rate limiting, user isolation
  - API CONTRACTS: Authentication endpoints, protected task endpoints
  - UI/UX REQUIREMENTS: Responsive design, 5 pages (landing, login, register, dashboard, 404)
  - PERFORMANCE REQUIREMENTS: Frontend (Lighthouse, FCP, LCP), Backend (API latency), Database
  - DEPLOYMENT REQUIREMENTS: Vercel (frontend), Public API (backend), Neon (database)
  - ERROR HANDLING: Frontend toast notifications, Backend HTTP status codes
  - DEVELOPMENT WORKFLOW: Branching strategy, Conventional Commits, PR review
  - NON-GOALS (PHASE II): Email verification, password reset, social auth, advanced features
  - APPENDIX: Recommended libraries, environment variables template

Removed Sections:
  - DEVELOPMENT PRINCIPLES (Phase I specific): NO MANUAL CODING, SPECIFICATION FIRST
  - FILE ORGANIZATION STANDARDS: speckit references (replaced with monorepo structure)
  - ACCEPTANCE CRITERIA REFERENCE: spec file references (Phase I specific)
  - CONSTITUTION HIERARCHY: speckit hierarchy (Phase I workflow)

Breaking Changes:
  - Architecture: Console → Web (frontend + backend separation)
  - Storage: In-memory Python lists → Neon PostgreSQL with user_id isolation
  - User Model: Single-user → Multi-user with JWT authentication
  - Interface: Terminal CLI → Browser-based responsive UI
  - Deployment: Local Python script → Vercel (frontend) + Public API (backend)
  - Dependencies: stdlib only → Next.js 16+, FastAPI, SQLModel, Better Auth
  - Project Structure: Flat src/ → Monorepo (frontend/, backend/, .specify/, specs/)

Templates Requiring Updates:
  ⚠️  .specify/templates/spec-template.md (Phase II features now require API contracts, authentication)
  ⚠️  .specify/templates/plan-template.md (Phase II requires frontend/backend split, database design)
  ⚠️  .specify/templates/tasks-template.md (Phase II tasks include deployment, database migrations)

Follow-up TODOs:
  - Create new feature specification for Phase II (full-stack web app)
  - Generate implementation plan with frontend/backend architecture
  - Break down tasks for monorepo setup, authentication, API, UI, deployment
  - Set up Neon PostgreSQL database
  - Configure Vercel deployment
  - Implement Better Auth with JWT tokens

Rationale:
  Complete architectural transformation from Phase I (console prototype) to Phase II (production-ready web application). This major version bump reflects backward-incompatible changes across all dimensions: architecture, stack, deployment, user model, and interface. Phase I code will remain in git history for reference, but Phase II requires clean slate implementation following new constitution requirements.

Notes:
  - Phase I (v1.0.0) successfully completed with all 5 basic features working in console
  - Phase II (v2.0.0) maintains the 5 core task operations but adds authentication, persistence, web UI
  - All Phase I validation rules preserved (title 1-100 chars, description max 500 chars)
  - User data isolation enforced at database query level (WHERE user_id = authenticated_user_id)
  - JWT tokens: access (15 min) + refresh (7 days) with HTTP-only cookies
  - Mandatory stack specified: no alternatives allowed for Phase II
  - Security requirements elevated for public deployment
-->

# Todo Web App - Constitution
# Phase II: Full-Stack Web Application with Authentication

## PROJECT PURPOSE

Phase II of Hackathon II: Transform the console todo application into a modern full-stack web application with authentication, database persistence, and RESTful API.

**Core Value Proposition:**
- Secure, multi-user todo management accessible via web browser
- Professional authentication with JWT tokens
- Database-backed persistence with user data isolation
- Responsive, modern UI with real-time feedback
- RESTful API for potential future integrations

---

## METADATA
- **Version:** 2.0.0
- **Ratified:** 2025-12-30
- **Last Amended:** 2025-12-30
- **Phase:** II (Full-Stack Web Application)
- **Status:** Active

---

## CHANGE LOG

### Version 2.0.0 (2025-12-30) - MAJOR
**Transformation to Full-Stack Web Application**

**Breaking Changes:**
- Architecture: Console application → Full-stack web application
- Storage: In-memory → Neon Serverless PostgreSQL
- User Model: Single-user → Multi-user with authentication
- Interface: Terminal CLI → Web browser UI
- Deployment: Local execution → Cloud deployment (Vercel + public API)

**Added:**
- Better Auth authentication system with JWT tokens
- Next.js 16+ frontend with TypeScript and Tailwind CSS
- FastAPI backend with SQLModel ORM
- PostgreSQL database schema with user isolation
- Monorepo structure (frontend/ and backend/ directories)
- Security requirements (CORS, rate limiting, input validation)
- Deployment requirements (Vercel for frontend, public API endpoint)
- Integration requirements (API contracts, error handling)

**Modified:**
- All 5 basic features now implemented in web UI
- Task CRUD operations now user-scoped with database persistence
- Error handling expanded for network/database failures
- Testing requirements updated for web stack (RTL, API tests, E2E)

**Rationale:**
Complete architectural transformation to meet Phase II requirements for a production-ready, multi-user web application with modern stack and deployment infrastructure.

---

### Version 1.0.0 (2025-12-30)
**Initial Constitution - Phase I Console Application**
- Python 3.13+ console application with in-memory storage
- 5 basic features (Add, View, Update, Delete, Toggle)
- Single-user, local execution, no persistence
- TDD methodology with comprehensive unit tests

---

## PHASE II TECHNICAL CONSTRAINTS

### MANDATORY STACK

**Frontend:**
- Next.js 16+ (App Router required)
- TypeScript (strict mode)
- Tailwind CSS for styling
- React Server Components where applicable
- Client-side validation with controlled forms

**Backend:**
- Python 3.13+
- FastAPI framework
- SQLModel for ORM
- Pydantic models for validation
- Async/await patterns

**Database:**
- Neon Serverless PostgreSQL
- Connection pooling
- Migrations (Alembic or similar)
- User data isolation enforced at query level

**Authentication:**
- Better Auth library
- JWT tokens (access + refresh)
- HTTP-only cookies for token storage
- Session management with expiration

**Architecture:**
- Monorepo structure:
  ```
  todo-app/
  ├── frontend/          # Next.js application
  ├── backend/           # FastAPI application
  ├── .specify/          # SpecKit Plus artifacts
  └── specs/            # Feature specifications
  ```

**Deployment:**
- Frontend: Vercel
- Backend: Public API endpoint (user's choice of platform)
- Environment variables for secrets
- CORS configuration for cross-origin requests

---

## AUTHENTICATION ARCHITECTURE

### USER FLOW
1. **Registration:**
   - User provides email + password
   - Backend validates, hashes password (bcrypt/argon2)
   - Creates user record in database
   - Returns success message

2. **Login:**
   - User provides email + password
   - Backend verifies credentials
   - Issues JWT access token (15 min expiry) + refresh token (7 days)
   - Stores refresh token in HTTP-only cookie
   - Returns access token to frontend

3. **Authenticated Requests:**
   - Frontend includes access token in Authorization header
   - Backend validates JWT signature and expiry
   - Extracts user_id from token payload
   - All task queries filtered by user_id

4. **Token Refresh:**
   - When access token expires, frontend uses refresh token
   - Backend validates refresh token, issues new access token
   - On refresh token expiry, user must re-login

5. **Logout:**
   - Frontend discards access token
   - Backend invalidates refresh token (optional: maintain revocation list)

---

## DATABASE SCHEMA REQUIREMENTS

### USERS TABLE
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
```

### TASKS TABLE
```sql
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(100) NOT NULL,
    description VARCHAR(500),
    completed BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_tasks_user_id ON tasks(user_id);
CREATE INDEX idx_tasks_created_at ON tasks(created_at DESC);
```

**User Isolation Rule:**
ALL task queries MUST include `WHERE user_id = <authenticated_user_id>` to prevent data leakage between users.

---

## FEATURES (Phase II)

### AUTHENTICATION FEATURES
1. **User Registration** - Email + password signup with validation
2. **User Login** - Email + password login with JWT token issuance
3. **User Logout** - Token invalidation and session cleanup
4. **Token Refresh** - Automatic access token renewal
5. **Protected Routes** - Client-side route guards for authenticated pages

### TASK MANAGEMENT FEATURES (User-Scoped)
1. **Add Task** - Create new task for authenticated user
2. **View All Tasks** - Display user's tasks with statistics
3. **Update Task** - Modify task title and/or description
4. **Delete Task** - Remove task with confirmation
5. **Toggle Complete** - Mark task complete/incomplete

**All task operations scoped to authenticated user only.**

---

## SECURITY REQUIREMENTS

### AUTHENTICATION SECURITY
- Passwords hashed with bcrypt (cost factor ≥ 12) or Argon2
- JWT secrets stored in environment variables (never hardcoded)
- Access tokens: short expiry (15 minutes recommended)
- Refresh tokens: HTTP-only cookies (not accessible via JavaScript)
- HTTPS required in production (enforced by deployment platforms)

### API SECURITY
- CORS whitelist: frontend origin only
- Rate limiting: 100 requests/minute per IP (adjustable)
- Input validation: Pydantic models on all endpoints
- SQL injection prevention: ORM parameterized queries only
- Error messages: user-friendly, no stack traces in production

### DATA SECURITY
- User isolation: All task queries filtered by user_id
- No cross-user data access (enforced at ORM level)
- Database credentials in environment variables
- Connection pooling with max connections limit

---

## API CONTRACTS

### AUTHENTICATION ENDPOINTS

**POST /api/auth/register**
```json
Request:
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}

Response (201):
{
  "message": "User registered successfully",
  "user": {
    "id": 1,
    "email": "user@example.com"
  }
}

Errors:
- 400: Email already exists
- 422: Validation error (email format, password strength)
```

**POST /api/auth/login**
```json
Request:
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}

Response (200):
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "user": {
    "id": 1,
    "email": "user@example.com"
  }
}
Set-Cookie: refresh_token=<token>; HttpOnly; Secure; SameSite=Strict

Errors:
- 401: Invalid credentials
- 422: Validation error
```

**POST /api/auth/logout**
```json
Response (200):
{
  "message": "Logged out successfully"
}
```

**POST /api/auth/refresh**
```json
Response (200):
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}

Errors:
- 401: Invalid or expired refresh token
```

### TASK ENDPOINTS (Protected)

**All endpoints require Authorization header: `Bearer <access_token>`**

**POST /api/tasks**
```json
Request:
{
  "title": "Buy groceries",
  "description": "Milk, eggs, bread"  // optional
}

Response (201):
{
  "id": 1,
  "user_id": 1,
  "title": "Buy groceries",
  "description": "Milk, eggs, bread",
  "completed": false,
  "created_at": "2025-12-30T10:00:00Z",
  "updated_at": "2025-12-30T10:00:00Z"
}

Errors:
- 401: Unauthorized (invalid/missing token)
- 422: Validation error (title empty or > 100 chars, description > 500 chars)
```

**GET /api/tasks**
```json
Response (200):
{
  "tasks": [
    {
      "id": 1,
      "title": "Buy groceries",
      "description": "Milk, eggs, bread",
      "completed": false,
      "created_at": "2025-12-30T10:00:00Z",
      "updated_at": "2025-12-30T10:00:00Z"
    }
  ],
  "statistics": {
    "total": 1,
    "completed": 0,
    "pending": 1
  }
}

Errors:
- 401: Unauthorized
```

**GET /api/tasks/{task_id}**
```json
Response (200):
{
  "id": 1,
  "title": "Buy groceries",
  "description": "Milk, eggs, bread",
  "completed": false,
  "created_at": "2025-12-30T10:00:00Z",
  "updated_at": "2025-12-30T10:00:00Z"
}

Errors:
- 401: Unauthorized
- 404: Task not found (or belongs to different user)
```

**PATCH /api/tasks/{task_id}**
```json
Request:
{
  "title": "Buy groceries and fruits",  // optional
  "description": "Milk, eggs, bread, apples"  // optional, null to clear
}

Response (200):
{
  "id": 1,
  "title": "Buy groceries and fruits",
  "description": "Milk, eggs, bread, apples",
  "completed": false,
  "created_at": "2025-12-30T10:00:00Z",
  "updated_at": "2025-12-30T10:15:00Z"
}

Errors:
- 401: Unauthorized
- 404: Task not found
- 422: Validation error
```

**DELETE /api/tasks/{task_id}**
```json
Response (200):
{
  "message": "Task deleted successfully"
}

Errors:
- 401: Unauthorized
- 404: Task not found
```

**PATCH /api/tasks/{task_id}/toggle**
```json
Response (200):
{
  "id": 1,
  "title": "Buy groceries",
  "description": "Milk, eggs, bread",
  "completed": true,
  "created_at": "2025-12-30T10:00:00Z",
  "updated_at": "2025-12-30T10:20:00Z"
}

Errors:
- 401: Unauthorized
- 404: Task not found
```

---

## UI/UX REQUIREMENTS

### RESPONSIVE DESIGN
- Mobile-first approach (320px min width)
- Breakpoints: sm (640px), md (768px), lg (1024px), xl (1280px)
- Touch-friendly targets (min 44x44px)
- Accessible keyboard navigation

### PAGES REQUIRED

1. **Landing Page (`/`)**
   - Hero section with app description
   - Login/Register buttons
   - Public, no authentication required

2. **Login Page (`/login`)**
   - Email + password form
   - Client-side validation
   - Error messages display
   - Link to register page

3. **Register Page (`/register`)**
   - Email + password + confirm password form
   - Password strength indicator
   - Email format validation
   - Link to login page

4. **Dashboard (`/dashboard`)**
   - Protected route (requires authentication)
   - Add task form at top
   - Task list with ✓/✗ status indicators
   - Statistics summary (total, completed, pending)
   - Logout button in header

5. **404 Page**
   - Custom not found page
   - Link back to home

### UI COMPONENTS

**Task List Item:**
```
[✓] Buy groceries - Milk, eggs, bread    [Edit] [Delete]
```
- Checkbox for toggle completion (clickable)
- Title and description display
- Edit button → opens inline edit form or modal
- Delete button → confirmation dialog

**Add Task Form:**
```
Title: [___________________]
Description (optional): [___________________]
                        [Add Task]
```

**Statistics Bar:**
```
Summary: 5 completed, 3 pending, 8 total
```

**Navigation:**
```
Todo App                      [user@example.com] [Logout]
```

---

## CODE QUALITY STANDARDS

### TYPESCRIPT/JAVASCRIPT
- Strict TypeScript mode enabled
- No `any` types (use `unknown` or proper types)
- All functions have return type annotations
- Props interfaces for all React components
- Async/await over callbacks
- Error boundaries for React components

### PYTHON
- Type hints on all functions (PEP 484)
- Pydantic models for all API request/response bodies
- Docstrings (Google style) for public functions
- Async endpoints where I/O-bound
- Exception handling with specific error types
- No global state (use dependency injection)

### REACT COMPONENTS
- Functional components with hooks
- Server Components by default (use 'use client' when needed)
- Component file structure:
  ```tsx
  'use client'  // if needed

  import statements

  interface Props { ... }

  export default function ComponentName({ props }: Props) {
    // hooks
    // handlers
    // render
  }
  ```

### CSS/TAILWIND
- Utility-first approach
- Component classes extracted when >5 utilities repeated
- Responsive prefixes (sm:, md:, lg:)
- Dark mode support optional (not required for Phase II)

---

## TESTING REQUIREMENTS

### FRONTEND TESTING
- **Unit Tests:** React Testing Library for components
- **Integration Tests:** Form submissions, API mocking
- **E2E Tests:** Critical paths (register → login → add task → logout)
- **Target Coverage:** >70% for Phase II

**Example Test:**
```tsx
test('should add a new task', async () => {
  render(<Dashboard />)
  const input = screen.getByLabelText('Title')
  const button = screen.getByText('Add Task')

  await userEvent.type(input, 'New task')
  await userEvent.click(button)

  expect(screen.getByText('New task')).toBeInTheDocument()
})
```

### BACKEND TESTING
- **Unit Tests:** Business logic functions
- **API Tests:** All endpoints with pytest + TestClient
- **Database Tests:** Use test database, rollback after each test
- **Target Coverage:** >80% for Phase II

**Example Test:**
```python
def test_create_task_authenticated(client, auth_token):
    response = client.post(
        "/api/tasks",
        json={"title": "Test task"},
        headers={"Authorization": f"Bearer {auth_token}"}
    )
    assert response.status_code == 201
    assert response.json()["title"] == "Test task"
```

---

## PERFORMANCE REQUIREMENTS

### FRONTEND
- Lighthouse Performance Score: >80
- First Contentful Paint (FCP): <2 seconds
- Largest Contentful Paint (LCP): <2.5 seconds
- Time to Interactive (TTI): <3 seconds
- Image optimization: next/image for all images
- Code splitting: dynamic imports for heavy components

### BACKEND
- API response time: <500ms for p95
- Database connection pooling: max 10 connections
- Query optimization: use indexes on user_id, created_at
- Pagination for task lists if >100 tasks (optional for Phase II)

### DATABASE
- Neon Serverless PostgreSQL auto-scaling
- Connection pooling configured
- Indexes on foreign keys and query columns

---

## DEPLOYMENT REQUIREMENTS

### FRONTEND (VERCEL)
- Automatic deployments from main branch
- Environment variables:
  - `NEXT_PUBLIC_API_URL`: Backend API base URL
- Build command: `npm run build`
- Output directory: `.next`
- Node.js version: 20.x

### BACKEND (PUBLIC API ENDPOINT)
- Publicly accessible HTTPS endpoint
- Environment variables:
  - `DATABASE_URL`: Neon PostgreSQL connection string
  - `JWT_SECRET`: Secret for signing tokens
  - `JWT_REFRESH_SECRET`: Secret for refresh tokens
  - `ALLOWED_ORIGINS`: Frontend origin for CORS
- Health check endpoint: `GET /health`
- ASGI server: Uvicorn with 4 workers

### DATABASE (NEON)
- Database created in Neon dashboard
- Connection string copied to backend env vars
- Auto-suspend enabled for cost savings
- Backups enabled (daily)

---

## ERROR HANDLING

### FRONTEND
- Network errors: "Unable to connect to server. Please try again."
- Validation errors: Display field-specific messages
- Authentication errors: Redirect to login page
- Generic errors: "An error occurred. Please try again."
- Toast notifications for feedback (success/error)

### BACKEND
- 400: Bad Request (validation errors)
- 401: Unauthorized (missing/invalid token)
- 404: Not Found (resource doesn't exist)
- 422: Unprocessable Entity (detailed validation errors)
- 500: Internal Server Error (log details, show generic message)

**Error Response Format:**
```json
{
  "detail": "User-friendly error message",
  "errors": [  // optional, for validation errors
    {
      "field": "email",
      "message": "Invalid email format"
    }
  ]
}
```

---

## DEVELOPMENT WORKFLOW

### BRANCHING STRATEGY
- `main`: Production-ready code
- `develop`: Integration branch for Phase II work
- Feature branches: `feature/<feature-name>`

### COMMIT STANDARDS
- Conventional Commits format
- Examples:
  - `feat(auth): implement JWT login`
  - `fix(tasks): correct user isolation query`
  - `docs(readme): add deployment instructions`

### CODE REVIEW
- All changes via Pull Requests
- Self-review before requesting review
- Check tests pass before merging

---

## SUCCESS CRITERIA (PHASE II)

### FUNCTIONAL REQUIREMENTS
- ✅ User can register with email + password
- ✅ User can login and receive JWT tokens
- ✅ User can logout and invalidate session
- ✅ User can add tasks (title + optional description)
- ✅ User can view all their tasks with statistics
- ✅ User can update task title and/or description
- ✅ User can delete tasks with confirmation
- ✅ User can toggle task completion status
- ✅ Users only see their own tasks (data isolation)
- ✅ Tasks persist in database across sessions

### TECHNICAL REQUIREMENTS
- ✅ Frontend deployed to Vercel
- ✅ Backend API publicly accessible
- ✅ Database hosted on Neon PostgreSQL
- ✅ All API endpoints protected with JWT authentication
- ✅ CORS configured correctly
- ✅ Input validation on frontend and backend
- ✅ Passwords hashed securely
- ✅ User data isolated at database level

### QUALITY REQUIREMENTS
- ✅ TypeScript strict mode with no errors
- ✅ Frontend tests pass (>70% coverage)
- ✅ Backend tests pass (>80% coverage)
- ✅ No console errors in browser
- ✅ Responsive design works on mobile and desktop
- ✅ Lighthouse Performance Score >80

### DOCUMENTATION REQUIREMENTS
- ✅ README updated with Phase II setup instructions
- ✅ API documentation (endpoints, request/response formats)
- ✅ Environment variables documented
- ✅ Deployment instructions for both frontend and backend

---

## NON-GOALS (PHASE II)

**Explicitly Out of Scope:**
- ❌ Email verification for registration
- ❌ Password reset functionality
- ❌ Social authentication (Google, GitHub, etc.)
- ❌ Task priorities, tags, or categories
- ❌ Due dates or reminders
- ❌ Task sharing or collaboration
- ❌ Real-time updates (WebSockets)
- ❌ Task search or filtering
- ❌ Bulk operations
- ❌ Dark mode toggle
- ❌ Internationalization (i18n)
- ❌ Advanced analytics or reporting

**Future Phases May Include:**
- Phase III: Advanced task features (priorities, tags, due dates)
- Phase IV: Collaboration features (sharing, comments)
- Phase V: Mobile app (React Native)

---

## GOVERNANCE

### CONSTITUTION UPDATES
- Minor updates (clarifications, formatting): Can be done directly
- Major updates (new requirements, breaking changes): Require version bump and change log entry
- All updates must maintain backward compatibility unless explicitly marked as BREAKING

### DECISION AUTHORITY
- Technical decisions: Developer (with ADR documentation for significant choices)
- Feature scope: Must align with Phase II requirements
- Stack changes: Not allowed (mandatory stack is fixed)

### DISPUTE RESOLUTION
- Ambiguities in constitution: Clarify with user/architect
- Conflicts between requirements: Prioritize security > functionality > performance
- Technical blockers: Document and propose alternatives

---

## APPENDIX

### RECOMMENDED LIBRARIES

**Frontend:**
- `react-hook-form` - Form management
- `zod` - Schema validation
- `axios` or `fetch` - HTTP client
- `react-hot-toast` - Toast notifications
- `@headlessui/react` - Unstyled UI components

**Backend:**
- `python-jose[cryptography]` - JWT handling
- `passlib[bcrypt]` - Password hashing
- `python-multipart` - Form data parsing
- `alembic` - Database migrations
- `pytest` - Testing framework
- `httpx` - Test client

### ENVIRONMENT VARIABLES TEMPLATE

**Frontend (.env.local):**
```
NEXT_PUBLIC_API_URL=http://localhost:8000
```

**Backend (.env):**
```
DATABASE_URL=postgresql://user:password@host/dbname
JWT_SECRET=your-secret-key-here
JWT_REFRESH_SECRET=your-refresh-secret-here
ALLOWED_ORIGINS=http://localhost:3000
ACCESS_TOKEN_EXPIRE_MINUTES=15
REFRESH_TOKEN_EXPIRE_DAYS=7
```

---

**Version**: 2.0.0 | **Ratified**: 2025-12-30 | **Last Amended**: 2025-12-30

**END OF CONSTITUTION**
