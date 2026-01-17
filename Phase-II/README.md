# Phase II: Full-Stack Web Application

## Project Overview

Phase II transforms the Phase I console application into a modern, secure, multi-user full-stack web application. This phase introduces user authentication, database persistence, RESTful APIs, and a responsive web interface, enabling users to access their tasks from any device with a web browser.

### What This Phase Does

This phase implements:
- **User Authentication**: Secure registration and login with JWT tokens
- **Database Persistence**: All data stored permanently in PostgreSQL
- **RESTful API**: Complete CRUD operations exposed via HTTP endpoints
- **Responsive Web UI**: Modern interface optimized for all device sizes
- **Multi-User Support**: Each user has isolated, private task data

### Why This Phase is Important

Phase II bridges the gap between a simple console app and a production-ready web application:
1. Transforms single-user to multi-user system
2. Adds data persistence (no more data loss on exit)
3. Enables access from any device via web browser
4. Implements security best practices
5. Establishes API foundation for Phase III AI integration

### Connection to Other Phases

- **From Phase I**: Inherits data model, validation rules, and business logic
- **To Phase III**: Provides authenticated API that AI chatbot will use
- **To Phase IV & V**: Complete application ready for containerization

---

## Objectives

| Objective | Description | Status |
|-----------|-------------|--------|
| User Registration | Allow new users to create accounts | Implemented |
| User Authentication | Secure login with JWT tokens | Implemented |
| Session Management | Token refresh and secure logout | Implemented |
| Task CRUD API | RESTful endpoints for all task operations | Implemented |
| Data Persistence | PostgreSQL database storage | Implemented |
| User Isolation | Users can only access their own tasks | Implemented |
| Responsive UI | Mobile, tablet, and desktop layouts | Implemented |
| Input Validation | Client and server-side validation | Implemented |

---

## Detailed Explanation

### Purpose and Problem Solved

**Problems with Phase I**:
- Single user only - no multi-user support
- Data lost when application exits
- Requires terminal access
- No security or authentication

**Phase II Solutions**:
- Multi-user with complete data isolation
- Persistent PostgreSQL database storage
- Web-based access from any device
- JWT-based authentication with password hashing

### Functional Responsibilities

#### 1. User Registration
- Accept email (unique, RFC 5322 format)
- Accept password (minimum 8 characters)
- Accept name (optional)
- Hash password using bcrypt (cost 12+)
- Create user record in database
- Automatically log in after registration
- Return JWT tokens

#### 2. User Authentication
- Validate email and password
- Verify password against stored hash
- Generate access token (7-day expiry)
- Generate refresh token (30-day expiry, HTTP-only cookie)
- Handle invalid credentials gracefully
- Rate limit login attempts (10 req/min)

#### 3. Session Management
- Store access token in React state (memory)
- Store refresh token in HTTP-only cookie
- Auto-refresh tokens before expiry
- Handle session expiration with redirect
- Secure logout (invalidate tokens)

#### 4. Task CRUD Operations
- **Create**: POST /api/tasks - Create task with user_id from JWT
- **Read**: GET /api/tasks - List user's tasks (filtered by user_id)
- **Update**: PUT /api/tasks/{id} - Update task (verify ownership)
- **Delete**: DELETE /api/tasks/{id} - Delete task (verify ownership)
- **Toggle**: PATCH /api/tasks/{id}/toggle - Toggle completion status

#### 5. Data Persistence
- Users table: id, email, password_hash, name, timestamps
- Tasks table: id, user_id (FK), title, description, completed, timestamps
- Cascade delete: user deletion removes all their tasks
- Indexed queries for performance

#### 6. User Isolation (Critical Security)
- ALL task queries include `WHERE user_id = authenticated_user_id`
- Backend extracts user_id from JWT, never from client input
- Attempting to access another user's task returns 404 (not 403)
- No task IDs exposed that could reveal other users' data

### Internal Workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         USER REGISTRATION FLOW                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Browser                    Next.js                  FastAPI             │
│     │                         │                         │                │
│     │── Visit /signup ───────►│                         │                │
│     │◄─── Render Form ────────│                         │                │
│     │                         │                         │                │
│     │── Submit Form ─────────►│                         │                │
│     │   {email, password,     │── POST /api/auth/register ──►│          │
│     │    name}                │   {email, password, name}    │          │
│     │                         │                         │                │
│     │                         │                    ┌────┴────┐           │
│     │                         │                    │Validate │           │
│     │                         │                    │Hash pwd │           │
│     │                         │                    │Create   │           │
│     │                         │                    │user     │           │
│     │                         │                    │Gen JWT  │           │
│     │                         │                    └────┬────┘           │
│     │                         │                         │                │
│     │                         │◄─── {user, tokens} ─────│                │
│     │◄─── Redirect /dashboard─│                         │                │
│     │     Store token in state│                         │                │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                         TASK MANAGEMENT FLOW                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Browser                    Next.js                  FastAPI      DB     │
│     │                         │                         │          │     │
│     │── View Dashboard ──────►│                         │          │     │
│     │                         │── GET /api/tasks ──────►│          │     │
│     │                         │   Authorization: Bearer │          │     │
│     │                         │                    ┌────┴────┐     │     │
│     │                         │                    │Verify   │     │     │
│     │                         │                    │JWT      │     │     │
│     │                         │                    │Extract  │     │     │
│     │                         │                    │user_id  │     │     │
│     │                         │                    └────┬────┘     │     │
│     │                         │                         │          │     │
│     │                         │                         │──SELECT──►     │
│     │                         │                         │  WHERE   │     │
│     │                         │                         │  user_id │     │
│     │                         │                         │◄─tasks───│     │
│     │                         │◄─── [tasks] ────────────│          │     │
│     │◄─── Render Task List ───│                         │          │     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Inputs → Processing → Outputs

| Endpoint | Method | Input | Processing | Output |
|----------|--------|-------|------------|--------|
| /api/auth/register | POST | {email, password, name} | Validate, hash password, create user, generate JWT | {user, access_token, refresh_token} |
| /api/auth/login | POST | {email, password} | Validate, verify password, generate JWT | {user, access_token, refresh_token} |
| /api/auth/logout | POST | Authorization header | Invalidate tokens | {success: true} |
| /api/auth/refresh | POST | Refresh token cookie | Verify refresh token, issue new access token | {access_token} |
| /api/tasks | GET | Authorization header | Verify JWT, query tasks by user_id | [{task}, {task}, ...] |
| /api/tasks | POST | {title, description?}, Auth header | Verify JWT, validate, create with user_id | {task} |
| /api/tasks/{id} | PUT | {title?, description?}, Auth header | Verify JWT, verify ownership, update | {task} |
| /api/tasks/{id} | DELETE | Auth header | Verify JWT, verify ownership, delete | {success: true} |
| /api/tasks/{id}/toggle | PATCH | Auth header | Verify JWT, verify ownership, toggle completed | {task} |

### Dependency on Phase I

| Phase I Component | Phase II Implementation |
|-------------------|-------------------------|
| Task dataclass (models.py) | SQLModel Task entity with user_id FK |
| TodoManager.add_task() | POST /api/tasks route handler |
| TodoManager.get_all_tasks() | GET /api/tasks with user_id filter |
| TodoManager.update_task() | PUT /api/tasks/{id} with ownership check |
| TodoManager.delete_task() | DELETE /api/tasks/{id} with ownership check |
| TodoManager.toggle_complete() | PATCH /api/tasks/{id}/toggle |
| Title validation (1-100 chars) | Pydantic TaskCreate schema |
| Description validation (500 chars) | Pydantic TaskCreate schema |
| In-memory list storage | PostgreSQL tasks table |

### How Phase II Expands the System

| Aspect | Phase I | Phase II |
|--------|---------|----------|
| Users | Single user | Multi-user with isolation |
| Storage | In-memory | PostgreSQL database |
| Access | Terminal only | Any web browser |
| Security | None | JWT auth, password hashing |
| Validation | Python functions | Client + server validation |
| Interface | Text menus | Responsive web UI |
| Persistence | Lost on exit | Permanent storage |

---

## Technology Stack

### Frontend

| Technology | Version | Purpose |
|------------|---------|---------|
| Next.js | 16+ | React framework with App Router |
| TypeScript | 5+ | Type-safe JavaScript |
| Tailwind CSS | 3+ | Utility-first styling |
| React | 19+ | UI component library |
| Better Auth Client | Latest | Auth state management |

### Backend

| Technology | Version | Purpose |
|------------|---------|---------|
| Python | 3.13+ | Backend language |
| FastAPI | 0.100+ | Web API framework |
| SQLModel | 0.0.14+ | ORM (Pydantic + SQLAlchemy) |
| Pydantic | 2+ | Data validation |
| Better Auth | Latest | Authentication library |
| Alembic | 1.12+ | Database migrations |
| bcrypt | 4+ | Password hashing |
| python-jose | 3+ | JWT handling |

### Database

| Technology | Version | Purpose |
|------------|---------|---------|
| PostgreSQL | 15+ | Relational database |
| Neon | Serverless | Managed PostgreSQL hosting |
| asyncpg | Latest | Async PostgreSQL driver |

### Deployment

| Technology | Purpose |
|------------|---------|
| Vercel | Frontend hosting |
| Railway/Render | Backend hosting |
| Neon | Database hosting |

---

## Folder Structure

```
Phase-II/
├── frontend/                           # Next.js Application
│   ├── src/
│   │   ├── app/                        # App Router pages
│   │   │   ├── page.tsx                # Landing page (/)
│   │   │   ├── layout.tsx              # Root layout with providers
│   │   │   ├── globals.css             # Global styles
│   │   │   ├── dashboard/              # Dashboard route
│   │   │   │   └── page.tsx            # Task management UI
│   │   │   └── (auth)/                 # Auth route group
│   │   │       ├── login/
│   │   │       │   └── page.tsx        # Login page
│   │   │       └── signup/
│   │   │           └── page.tsx        # Registration page
│   │   │
│   │   ├── components/                 # React Components
│   │   │   ├── task/                   # Task components
│   │   │   │   ├── TaskList.tsx        # Task list container
│   │   │   │   ├── TaskItem.tsx        # Individual task
│   │   │   │   ├── TaskForm.tsx        # Create/edit form
│   │   │   │   └── TaskFilter.tsx      # Filter controls
│   │   │   ├── auth/                   # Auth components
│   │   │   │   ├── LoginForm.tsx       # Login form
│   │   │   │   └── SignupForm.tsx      # Signup form
│   │   │   └── ui/                     # Shared UI components
│   │   │       ├── Button.tsx
│   │   │       ├── Input.tsx
│   │   │       └── Modal.tsx
│   │   │
│   │   ├── lib/                        # Utilities
│   │   │   ├── api.ts                  # Task API client
│   │   │   ├── auth.ts                 # Auth utilities
│   │   │   └── utils.ts                # Helper functions
│   │   │
│   │   ├── types/                      # TypeScript types
│   │   │   └── index.ts                # Shared type definitions
│   │   │
│   │   └── middleware.ts               # Auth route protection
│   │
│   ├── public/                         # Static assets
│   ├── package.json                    # Dependencies
│   ├── tsconfig.json                   # TypeScript config
│   ├── tailwind.config.js              # Tailwind config
│   ├── next.config.js                  # Next.js config
│   └── .env.example                    # Environment template
│
├── backend/                            # FastAPI Application
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                     # FastAPI entry point
│   │   ├── config.py                   # Settings management
│   │   ├── database.py                 # Database connection
│   │   │
│   │   ├── models/                     # SQLModel entities
│   │   │   ├── __init__.py
│   │   │   ├── user.py                 # User model
│   │   │   └── task.py                 # Task model
│   │   │
│   │   ├── schemas/                    # Pydantic schemas
│   │   │   ├── __init__.py
│   │   │   ├── user.py                 # User request/response
│   │   │   └── task.py                 # Task request/response
│   │   │
│   │   ├── routes/                     # API route handlers
│   │   │   ├── __init__.py
│   │   │   ├── auth.py                 # Auth endpoints
│   │   │   └── tasks.py                # Task CRUD endpoints
│   │   │
│   │   └── middleware/                 # Custom middleware
│   │       ├── __init__.py
│   │       └── auth.py                 # JWT verification
│   │
│   ├── tests/                          # Backend tests
│   │   ├── __init__.py
│   │   ├── conftest.py                 # Test fixtures
│   │   ├── test_auth.py                # Auth endpoint tests
│   │   └── test_tasks.py               # Task endpoint tests
│   │
│   ├── alembic/                        # Database migrations
│   │   ├── versions/                   # Migration files
│   │   ├── env.py                      # Alembic config
│   │   └── alembic.ini                 # Alembic settings
│   │
│   ├── requirements.txt                # Python dependencies
│   └── .env.example                    # Environment template
│
├── specs/                              # Specifications
│   └── 001-phase2-fullstack-web-app/
│       ├── spec.md                     # Feature specification
│       ├── plan.md                     # Architecture plan
│       ├── data-model.md               # Database schema
│       ├── research.md                 # Tech decisions
│       ├── quickstart.md               # Setup guide
│       ├── tasks.md                    # Implementation tasks
│       ├── contracts/
│       │   ├── auth-api.md             # Auth API contract
│       │   └── tasks-api.md            # Tasks API contract
│       └── checklists/
│           └── requirements.md         # Requirements checklist
│
├── CLAUDE.md                           # Development rules
└── README.md                           # This documentation
```

### Folder Details

| Folder | Purpose |
|--------|---------|
| `frontend/src/app/` | Next.js App Router pages and layouts |
| `frontend/src/components/` | Reusable React components by feature |
| `frontend/src/lib/` | API clients and utility functions |
| `backend/app/models/` | SQLModel database entities |
| `backend/app/schemas/` | Pydantic request/response validation |
| `backend/app/routes/` | FastAPI route handlers |
| `backend/alembic/` | Database migration scripts |
| `specs/` | SDD specification documents |

---

## Setup & Installation

### Prerequisites

- Python 3.13+
- Node.js 18+
- PostgreSQL database (or Neon account)
- Git

### Backend Setup

```bash
# Navigate to backend
cd Phase-II/backend

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your values:
# DATABASE_URL=postgresql://user:password@host:5432/dbname
# JWT_SECRET=your-32-character-secret-key
# CORS_ORIGINS=http://localhost:3000

# Run database migrations
alembic upgrade head

# Start development server
uvicorn app.main:app --reload --port 8000
```

### Frontend Setup

```bash
# Navigate to frontend (new terminal)
cd Phase-II/frontend

# Install dependencies
npm install

# Configure environment
cp .env.example .env.local
# Edit .env.local:
# NEXT_PUBLIC_API_URL=http://localhost:8000

# Start development server
npm run dev
```

### Access the Application

- Frontend: http://localhost:3000
- Backend API: http://localhost:8000
- API Documentation: http://localhost:8000/docs

---

## Usage Instructions

### User Registration

1. Open http://localhost:3000
2. Click "Sign Up"
3. Enter email, password (8+ characters), and name
4. Click "Create Account"
5. You'll be redirected to dashboard

### User Login

1. Open http://localhost:3000
2. Click "Login"
3. Enter email and password
4. Click "Sign In"

### Task Management

**Creating Tasks:**
1. Click "Add Task" button
2. Enter title (required) and description (optional)
3. Click "Create"

**Viewing Tasks:**
- All tasks displayed on dashboard
- Filter by: All, Pending, Completed
- Tasks sorted by creation date (newest first)

**Updating Tasks:**
1. Click edit icon on task
2. Modify title or description
3. Click "Save"

**Deleting Tasks:**
1. Click delete icon on task
2. Confirm deletion in modal
3. Task removed

**Toggling Completion:**
- Click checkbox to toggle complete/incomplete
- Visual indicator updates immediately

### Logout

1. Click user menu (top right)
2. Click "Logout"
3. Redirected to login page

---

## API Reference

### Authentication Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| /api/auth/register | POST | Create new user account |
| /api/auth/login | POST | Authenticate and get tokens |
| /api/auth/logout | POST | Invalidate session |
| /api/auth/refresh | POST | Get new access token |

### Task Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| /api/tasks | GET | List all user's tasks |
| /api/tasks | POST | Create new task |
| /api/tasks/{id} | GET | Get specific task |
| /api/tasks/{id} | PUT | Update task |
| /api/tasks/{id} | DELETE | Delete task |
| /api/tasks/{id}/toggle | PATCH | Toggle completion |

All task endpoints require `Authorization: Bearer <token>` header.

---

## Testing

### Backend Tests

```bash
cd Phase-II/backend

# Run all tests
pytest

# Run with coverage
pytest --cov=app

# Run specific test file
pytest tests/test_auth.py
pytest tests/test_tasks.py
```

### Frontend Tests

```bash
cd Phase-II/frontend

# Run unit tests
npm test

# Run E2E tests
npm run test:e2e
```

---

## Future Scope (Addressed in Later Phases)

| Feature | Phase |
|---------|-------|
| AI Chatbot Interface | Phase III |
| Natural Language Commands | Phase III |
| MCP Server Integration | Phase III |
| Docker Containerization | Phase IV |
| Kubernetes Deployment | Phase IV |
| Helm Charts | Phase IV |
| Production Monitoring | Phase V |
| AI-Assisted DevOps | Phase V |

---

## Conclusion

Phase II successfully transforms the Phase I console application into a production-ready full-stack web application. Key achievements:

1. **Multi-User Architecture**: Complete user isolation with JWT authentication
2. **Data Persistence**: PostgreSQL database with proper relationships
3. **RESTful API**: Well-documented endpoints following best practices
4. **Responsive UI**: Modern interface working on all devices
5. **Security**: Password hashing, token management, CORS configuration
6. **Extensibility**: Clean architecture ready for Phase III AI integration

The API foundation established here enables Phase III to add AI capabilities without modifying the core task management logic.
