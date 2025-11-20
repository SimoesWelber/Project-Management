# Configuration Files

This directory contains configuration files for the INCA platform.

## Files

- `database.example.json` - Database configuration template
- `environment.example.env` - Environment variables template
- `app.config.example.json` - Application configuration template

## Usage

1. Copy the example files and remove `.example` from the filename
2. Update the values with your actual configuration
3. Never commit actual configuration files with sensitive data to version control

## Configuration Types

### Database Configuration
- Database host and port
- Database name
- Credentials
- Connection pool settings

### Application Configuration
- Server port
- API endpoints
- File upload limits
- Session settings
- CORS settings

### Environment Variables
- NODE_ENV (development/production)
- JWT_SECRET
- Database credentials
- API keys for external services
- Email service configuration

## Security Notes

- Always use environment variables for sensitive data
- Rotate secrets regularly
- Use different configurations for different environments
- Keep production secrets secure
