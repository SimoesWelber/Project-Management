# Quick Start Guide

## INCA - INGENIA Cuniculus Aprobation

This quick start guide will help you get the INCA platform up and running quickly.

## Prerequisites

Before you begin, ensure you have:
- Git installed
- A code editor (VS Code recommended)
- Database (PostgreSQL or MySQL)
- Node.js (v16+) or Python (v3.9+) - depending on your technology choice

## Step-by-Step Setup

### 1. Clone the Repository

```bash
git clone https://github.com/SimoesWelber/Project-Management.git
cd Project-Management
```

### 2. Review the Project Structure

```
Project-Management/
├── backend/              # Backend API and services
├── frontend/             # Frontend web application
├── docs/                 # Comprehensive documentation
├── config/               # Configuration templates
└── README.md            # Project overview
```

### 3. Configure Your Environment

Copy the configuration templates:

```bash
# Database configuration
cp config/database.example.json config/database.json

# Environment variables
cp config/environment.example.env .env

# Application configuration
cp config/app.config.example.json config/app.config.json
```

Edit these files with your actual values:
- Database connection details
- JWT secrets
- API keys
- File upload paths

### 4. Set Up the Database

Create your database:

```bash
# For PostgreSQL
createdb inca_dev

# For MySQL
mysql -u root -p -e "CREATE DATABASE inca_dev;"
```

### 5. Install Dependencies

#### Backend
```bash
cd backend
npm install  # or pip install -r requirements.txt for Python
```

#### Frontend
```bash
cd frontend
npm install
```

### 6. Run Migrations (Once Implemented)

```bash
cd backend
npm run migrate
```

### 7. Start the Development Servers

#### Terminal 1 - Backend
```bash
cd backend
npm run dev
```

#### Terminal 2 - Frontend
```bash
cd frontend
npm start
```

### 8. Access the Application

- Frontend: http://localhost:3001
- Backend API: http://localhost:3000
- API Documentation: http://localhost:3000/api/docs (once implemented)

## Default Credentials

Once user management is implemented, default admin credentials will be:
- Username: admin
- Password: (to be set during first run)

## Next Steps

1. **Read the Documentation**: Check out `/docs` for detailed information
2. **Review the Architecture**: Understand the system design in `docs/architecture.md`
3. **Database Schema**: Review `docs/database.md` for data structure
4. **Development Guide**: Follow `docs/development.md` for development practices

## Common Commands

```bash
# Backend
npm run dev          # Start development server
npm test             # Run tests
npm run migrate      # Run database migrations
npm run lint         # Lint code

# Frontend
npm start            # Start development server
npm test             # Run tests
npm run build        # Build for production
npm run lint         # Lint code
```

## Troubleshooting

### Port Already in Use
```bash
# Find and kill process
lsof -ti:3000 | xargs kill -9
```

### Database Connection Failed
- Check database is running
- Verify credentials in configuration
- Ensure database exists

### Module Not Found
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
```

## Getting Help

- Check the [Documentation](./docs/README.md)
- Review [Contributing Guidelines](./CONTRIBUTING.md)
- Open an issue on GitHub
- Contact the project maintainers

## Important Notes

⚠️ **Security**: Never commit actual configuration files with secrets to version control

⚠️ **Environment**: Always use `.env` files for sensitive data

⚠️ **Backup**: Regular database backups are recommended

## What's Next?

After completing the quick start:
1. Implement authentication
2. Create database models
3. Build API endpoints
4. Develop frontend components
5. Add tests
6. Deploy to production

Refer to the [Project Roadmap](./docs/roadmap.md) for the complete development plan.
