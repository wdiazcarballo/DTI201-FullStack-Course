# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is an educational repository for the **DTI201 Full-Stack Development course**, designed to teach modern web development through hands-on tutorials and workshops. The content is delivered in Thai language with practical, project-based learning materials covering GitHub workflows, React/Vite SPAs, Docker containerization, and database best practices.

## Course Structure

The repository contains sequential tutorial modules, each designed for ~60-90 minute sessions:

1. **GitHub Workflow** - Random Quote Generator with CI/CD
2. **Vite & React SPA** - Single Page Application development
3. **JavaScript Modules** - CommonJS vs ES Modules
4. **Code Quality** - Refactoring and best practices
5. **Database Connection** - MongoDB/Mongoose patterns
6. **API Testing** - CURL command reference
7. **Docker Basics** - Container fundamentals (Week 11)
8. **Git CLI & Workflow** - Version control practices
9. **Docker Commands** - Web hosting with Docker Compose

## Technology Stack

### Frontend
- **React 18** with Vite build tool
- ES6+ JavaScript (prefer ES Modules syntax)
- React Router for SPA navigation
- Lazy loading with React.lazy() and Suspense

### Backend
- **Node.js 14-18** (Alpine images preferred for Docker)
- Express.js for REST APIs
- Mongoose for MongoDB ODM
- CORS middleware for cross-origin requests

### DevOps & Tools
- **Docker & Docker Compose** for containerization
- GitHub Actions for CI/CD
- GitHub Pages for static deployments
- Nginx as reverse proxy in production

### Database
- MongoDB (local or Atlas)
- PostgreSQL (in Docker examples)
- Connection pooling with retry logic

## Key Development Patterns

### Module System
Always use **ES Modules** (not CommonJS) for new code:
```javascript
// ✅ Preferred
import express from 'express';
export const myFunction = () => {};

// ❌ Avoid
const express = require('express');
module.exports = myFunction;
```

Files must use `.js` extension with `"type": "module"` in package.json.

### Database Connection
Use connection pooling with proper error handling and graceful shutdown:
```javascript
// Structure: db/connection.js exports singleton
await dbConnection.connect();
// With retry logic and event listeners
```

Never hardcode connection strings - use environment variables via `.env` files.

### Docker Workflows
All Docker projects follow this structure:
```
project/
├── docker-compose.yml
├── .env.example
├── frontend/
│   └── Dockerfile
├── backend/
│   └── Dockerfile
└── nginx/
    └── default.conf
```

Use multi-stage builds for production images to minimize size.

## Common Commands

### Development Setup
```bash
# Install dependencies
npm install

# Start development server (Vite)
npm run dev

# Start Express backend
npm start
npm run dev  # with nodemon
```

### Docker Commands
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f [service-name]

# Rebuild after changes
docker-compose up -d --build

# Stop and remove everything
docker-compose down -v

# Enter container shell
docker-compose exec [service-name] sh
```

### Testing APIs
```bash
# Health check
curl http://localhost:PORT/api/health

# CRUD operations (examples in tutorial 6)
curl -X POST http://localhost:3000/contact \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Test"}'
```

### Git Workflow
Follow GitHub Flow pattern:
```bash
# Create feature branch
git checkout -b feature/[feature-name]

# Commit with conventional commits
git commit -m "feat: description"
git commit -m "fix: description"

# Create PR using GitHub CLI
gh pr create --title "Title" --body "Description"
```

## Project Architecture Notes

### Quote Generator (Tutorial 1)
- Vanilla HTML/CSS/JS with GitHub Actions deployment
- CI pipeline validates HTML and checks JS syntax
- Auto-deploys to GitHub Pages on main branch merge
- Structure: index.html, styles.css, script.js, .github/workflows/deploy.yml

### React SPA Pattern (Tutorial 2)
- Component-based architecture with hooks (useState, useEffect)
- Custom hooks for reusable logic (e.g., useLocalStorage)
- Error boundaries for graceful error handling
- Route-based code splitting for performance
- Environment variables prefixed with VITE_

### Docker Full-Stack Setup (Tutorials 7, 9)
- Frontend: Node builder → Nginx static server
- Backend: Node with nodemon for hot reload
- Database: PostgreSQL/MongoDB with volume persistence
- Network: All services in shared bridge network
- Health checks on database service with condition-based depends_on

### Database Best Practices (Tutorial 5)
- Singleton connection class with connection pooling
- Retry logic with exponential backoff (5 retries default)
- Event listeners for connection lifecycle
- Graceful shutdown on SIGINT
- Health check middleware for API routes

## File Naming Conventions

- Tutorial files: Numbered with Thai descriptions (e.g., "7 Docker.md")
- Use `.cjs` extension for CommonJS in ES Module projects
- Docker files: `Dockerfile` (production), `Dockerfile.dev` (development)
- Config files: `.env` (gitignored), `.env.example` (committed)

## Workshop Format

Each tutorial follows this structure:
1. **Learning Objectives** - Clear goals (5 min)
2. **Prerequisites Check** - Required tools/knowledge
3. **Hands-on Labs** - Step-by-step implementations (40-60 min)
4. **Troubleshooting** - Common errors and solutions
5. **Homework/Extensions** - Additional challenges
6. **Assessment Criteria** - CLO mapping with percentages

## Important Notes

- All tutorials are designed for **60-90 minute sessions**
- Content is bilingual: Thai instructions with English code/commands
- Emphasize "why" over "what" in explanations
- Include real-world debugging scenarios
- Projects build on each other sequentially

## Documentation Standards

When creating or updating tutorials:
- Include timing estimates for each section
- Provide complete, runnable code examples
- Show both error cases and solutions
- Use emoji sparingly for visual organization (📚 🔧 ✅ ❌)
- Include cheat sheets and command references
- Map to Course Learning Outcomes (CLOs) where applicable

## Troubleshooting References

Common issues are documented in each tutorial:
- Port conflicts → Change ports in docker-compose.yml
- Permission errors → Use `--chown` in Dockerfile COPY
- Module not found → Check `.js` extension in imports
- Database connection refused → Use service names, not localhost
- Container exits immediately → Check logs with docker-compose logs
