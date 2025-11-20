# INCA Development Guide

## Getting Started

This guide will help you set up your development environment for the INCA platform.

## Prerequisites

### Required Software

- **Node.js** (v16 or higher) or **Python** (v3.9 or higher) - depending on backend choice
- **Database**: PostgreSQL 13+ or MySQL 8+
- **Git**: Version control
- **Code Editor**: VS Code, IntelliJ, or your preferred IDE

### Recommended Tools

- **Postman** or **Insomnia** - API testing
- **Docker** - For containerized development
- **pgAdmin** or **DBeaver** - Database management

## Initial Setup

### 1. Clone the Repository

```bash
git clone https://github.com/SimoesWelber/Project-Management.git
cd Project-Management
```

### 2. Backend Setup

```bash
cd backend
# Install dependencies (example for Node.js)
npm install

# Copy environment configuration
cp ../config/environment.example.env .env

# Update .env with your local settings
# Configure database connection, JWT secrets, etc.
```

### 3. Database Setup

```bash
# Create database
createdb inca_dev

# Run migrations (once implemented)
npm run migrate

# Seed initial data (optional)
npm run seed
```

### 4. Frontend Setup

```bash
cd frontend
# Install dependencies
npm install

# Copy environment configuration if needed
cp .env.example .env

# Update API endpoint URLs
```

### 5. Start Development Servers

```bash
# Terminal 1 - Backend
cd backend
npm run dev

# Terminal 2 - Frontend
cd frontend
npm start
```

## Development Workflow

### Branch Strategy

- `main` - Production-ready code
- `develop` - Development branch
- `feature/*` - Feature branches
- `bugfix/*` - Bug fix branches
- `hotfix/*` - Production hotfix branches

### Workflow Steps

1. Create a new branch from `develop`
   ```bash
   git checkout develop
   git pull origin develop
   git checkout -b feature/your-feature-name
   ```

2. Make your changes and commit
   ```bash
   git add .
   git commit -m "Add feature description"
   ```

3. Push and create pull request
   ```bash
   git push origin feature/your-feature-name
   ```

## Code Standards

### Backend

- Use meaningful variable and function names
- Follow the established project structure
- Write unit tests for new functionality
- Document complex logic with comments
- Use async/await for asynchronous operations
- Implement proper error handling

### Frontend

- Use functional components with hooks (React)
- Follow component naming conventions
- Keep components small and focused
- Use PropTypes or TypeScript for type checking
- Implement responsive design
- Optimize for performance

### General

- Write self-documenting code
- Keep functions small and single-purpose
- Use consistent formatting (Prettier/ESLint)
- Update documentation when needed

## Testing

### Running Tests

```bash
# Backend tests
cd backend
npm test

# Frontend tests
cd frontend
npm test

# Run with coverage
npm run test:coverage
```

### Writing Tests

- Write unit tests for business logic
- Write integration tests for API endpoints
- Write E2E tests for critical user flows
- Aim for good test coverage (70%+ minimum)

## Debugging

### Backend Debugging

```bash
# Node.js debugging
node --inspect-brk src/index.js

# Or use IDE debugging configuration
```

### Frontend Debugging

- Use browser DevTools
- React DevTools extension
- Console logging (remove before commit)

## Database Migrations

```bash
# Create new migration
npm run migration:create -- --name your_migration_name

# Run migrations
npm run migrate

# Rollback last migration
npm run migrate:undo
```

## API Documentation

- Document all endpoints using comments
- Include request/response examples
- Specify required vs optional parameters
- Document error responses

## Common Issues

### Port Already in Use

```bash
# Find and kill process using port 3000
lsof -ti:3000 | xargs kill -9
```

### Database Connection Issues

- Verify PostgreSQL is running
- Check credentials in .env file
- Ensure database exists

### Module Not Found

```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
```

## Performance Tips

- Use database indexes appropriately
- Implement caching where beneficial
- Optimize database queries
- Minimize API calls
- Lazy load components and routes

## Security Best Practices

- Never commit secrets or credentials
- Validate all user inputs
- Use parameterized queries (prevent SQL injection)
- Implement rate limiting
- Keep dependencies updated
- Use HTTPS in production
- Implement proper authentication/authorization

## Getting Help

- Check documentation in `/docs`
- Review existing code for patterns
- Ask team members
- Create issues for bugs or questions

## Contributing

Please read the project contributing guidelines before submitting pull requests.
