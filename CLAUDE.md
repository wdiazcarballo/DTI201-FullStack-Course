# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is an educational repository for the DTI201 Full-Stack Development course, focusing on teaching GitHub workflows, CI/CD practices, and modern web development. The repository contains hands-on tutorials, visual learning materials, and comprehensive roadmaps for full-stack development.

## Current Repository Structure

### Main Content
- **1 Github Workflow.md**: Complete hands-on tutorial for GitHub workflow using a Random Quote Generator project
- **assets/**: Visual learning materials including full-stack development roadmaps and workflow diagrams
- **README.md**: Basic repository identifier

## Key Tutorial: Random Quote Generator

The main hands-on project demonstrates:
- Complete GitHub workflow (branch → commit → PR → review → merge)
- GitHub Actions CI/CD pipeline
- GitHub Pages deployment
- HTML/CSS/JavaScript fundamentals

### Project Files Structure
```
quote-generator/
├── index.html    # Main HTML structure
├── styles.css    # Styling with gradients and flexbox
├── script.js     # Quote rotation logic
└── .github/
    └── workflows/
        └── deploy.yml  # CI/CD pipeline
```

## Essential Commands

### Git Workflow Commands
```bash
# Clone repository
git clone https://github.com/[username]/quote-generator-[yourname].git

# Create feature branch
git checkout -b feature/[feature-name]

# Stage and commit changes
git add .
git commit -m "feat: description"

# Push branch
git push origin [branch-name]

# Create pull request (using GitHub CLI)
gh pr create --title "Add feature" --body "Description of changes"
```

### GitHub Actions Workflow
The tutorial includes setting up automated CI/CD with:
- HTML validation using `html-validator-cli`
- JavaScript syntax checking with `node -c`
- Automatic deployment to GitHub Pages on merge to main

### Local Development
```bash
# For the static quote generator project
# No build process required - open index.html directly in browser

# If using Live Server extension in VS Code
# Right-click index.html → "Open with Live Server"
```

## Learning Path Technologies

Based on the full-stack roadmap in assets/, the course covers:

### Frontend Stack
- HTML5, CSS3, JavaScript (ES6+)
- React.js, Next.js
- TypeScript
- State Management (Redux/Context)
- Testing with Jest

### Backend Stack
- Node.js, Express.js
- RESTful APIs, GraphQL
- Authentication (JWT, OAuth)
- Testing with Mocha/Jest

### Database & DevOps
- MongoDB (NoSQL), PostgreSQL (SQL)
- Docker, Kubernetes
- GitHub Actions CI/CD
- Cloud platforms (AWS, GCP)

## Project Implementation Timeline

The GitHub Workflow tutorial follows this 60-minute structure:
1. **0:00-0:15**: Repository setup and initial code
2. **0:15-0:25**: Feature branch workflow
3. **0:25-0:35**: Pull request and code review
4. **0:35-0:50**: GitHub Actions CI/CD setup
5. **0:50-0:60**: Deployment and wrap-up

## Important Notes

1. **Educational Focus**: This repository is designed for learning - prioritize clear, well-commented code that demonstrates concepts
2. **GitHub Pages**: The quote generator deploys automatically to `https://[username].github.io/quote-generator-[yourname]/`
3. **Branch Protection**: The tutorial recommends enabling branch protection rules for the main branch
4. **Code Reviews**: Emphasize proper PR descriptions and constructive code review practices