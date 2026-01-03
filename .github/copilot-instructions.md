# Mergington High School Activities API

## Architecture Overview

This is a **FastAPI + MongoDB + Vanilla JS** application for managing high school extracurricular activities. The system has three main layers:

1. **Backend** ([src/app.py](src/app.py), [src/backend/](src/backend/)): FastAPI server with router-based endpoints
2. **Database** ([src/backend/database.py](src/backend/database.py)): MongoDB collections with in-memory initialization patterns
3. **Frontend** ([src/static/](src/static/)): Static HTML/CSS/JS served via FastAPI's StaticFiles mount

### Critical Design Patterns

- **Data Model**: MongoDB collections use **natural keys** (`_id`) - activity names and usernames, not UUIDs
- **Authentication**: Teacher accounts with Argon2 password hashing; username passed via query params `?teacher_username=X`
- **State Management**: Frontend uses `localStorage` for session persistence and validates against `/auth/check-session`
- **Filter Implementation**: Backend supports query params (`day`, `start_time`, `end_time`) for activity filtering at [src/backend/routers/activities.py](src/backend/routers/activities.py#L17-L47)

## Development Workflows

**Run the application**:
```bash
cd /workspaces/skills-copilot-code-review/src
uvicorn app:app --reload
```

**Access points**:
- Frontend: http://localhost:8000/static/index.html
- API Docs: http://localhost:8000/docs
- MongoDB: `mongodb://localhost:27017/` (database: `mergington_high`)

**Testing teacher login**:
- Username: `mrodriguez` / Password: `art123`
- Username: `mchen` / Password: `chess456`

## Backend Conventions

- **All API endpoints** are defined in [src/backend/routers/](src/backend/routers/) - never add routes directly to [src/app.py](src/app.py)
- **Database initialization**: Sample data lives in `initial_activities` and `initial_teachers` dicts in [database.py](src/backend/database.py)
- **Error handling**: Backend logs errors but returns HTTP exceptions to frontend - do NOT propagate internal error details
- **Router pattern**: Use `APIRouter` with `prefix` and `tags`, then include in [app.py](src/app.py) via `app.include_router()`
- **Authentication checks**: See [activities.py signup endpoint](src/backend/routers/activities.py#L69-L77) for the pattern - validate `teacher_username` query param

## Frontend Conventions

- **API calls**: All `fetch()` calls use relative paths (`/activities`, `/auth/login`) - see [app.js](src/static/app.js)
- **State variables**: Global state at top of [app.js](src/static/app.js#L37-L46): `allActivities`, `currentUser`, `currentDay`, etc.
- **Authentication flow**: Check `localStorage.currentUser` → validate with `/auth/check-session` → update UI via `updateAuthUI()`
- **Filter UI updates**: Always update button `.active` class AND call `fetchActivities()` - see [setDayFilter](src/static/app.js#L70-L82)
- **Accessibility**: Use `aria-label` attributes and semantic HTML (existing in [index.html](src/static/index.html))

## Security

- **Input sanitization**: Always validate and sanitize user inputs before database queries
- **Password handling**: Use `hash_password()` and `verify_password()` from [database.py](src/backend/database.py#L16-L35) - never store plain passwords
- **Configuration**: Load from database or environment variables, not hardcoded values

## Code Quality

- **Naming**: Use snake_case for Python, camelCase for JavaScript (matching existing patterns)
- **Error handling**: Explicit try/catch in frontend, HTTPException in backend - no silent failures
- **Optimization**: Methods like `fetchActivities()` are called frequently - optimize database queries
- **Backend-Frontend sync**: When modifying API contracts, verify changes in both [routers/](src/backend/routers/) AND [static/app.js](src/static/app.js)