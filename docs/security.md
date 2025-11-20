# Security Guidelines

## INCA Security Best Practices

Security is paramount for the INCA platform as it handles sensitive engineering documents for critical infrastructure projects.

## Authentication & Authorization

### Password Requirements
- Minimum 12 characters
- Must include uppercase, lowercase, numbers, and special characters
- Password history: prevent reuse of last 5 passwords
- Password expiration: 90 days
- Account lockout: 5 failed attempts

### JWT Tokens
- Use strong secret keys (256-bit minimum)
- Short expiration times (24 hours for access tokens)
- Implement refresh token rotation
- Store tokens securely (httpOnly cookies or secure storage)
- Validate token on every request

### Role-Based Access Control (RBAC)
```javascript
// Example middleware
const authorize = (roles = []) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    if (roles.length && !roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    
    next();
  };
};

// Usage
router.post('/documents', authorize(['ADMIN', 'SUBMITTER']), createDocument);
```

## Input Validation

### Server-Side Validation
Always validate on the server, never trust client input:

```javascript
const { body, validationResult } = require('express-validator');

router.post('/documents',
  [
    body('title').trim().isLength({ min: 3, max: 255 }).escape(),
    body('description').trim().isLength({ max: 5000 }).escape(),
    body('document_type').isIn(['Engineering Design', 'Technical Specification', /* ... */])
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(422).json({ errors: errors.array() });
    }
    // Process request
  }
);
```

### SQL Injection Prevention
- Use parameterized queries or ORM
- Never concatenate user input into SQL queries
- Use prepared statements

```javascript
// Good - Parameterized query
db.query('SELECT * FROM documents WHERE id = ?', [documentId]);

// Bad - String concatenation
db.query(`SELECT * FROM documents WHERE id = ${documentId}`); // VULNERABLE!
```

### XSS Prevention
- Sanitize all user input
- Use Content Security Policy (CSP) headers
- Encode output when rendering HTML
- Use templating engines with auto-escaping

```javascript
// CSP Header
helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'unsafe-inline'"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", "data:", "https:"],
  },
});
```

## File Upload Security

### File Type Validation
```javascript
const ALLOWED_MIME_TYPES = [
  'application/pdf',
  'application/msword',
  'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
  'image/png',
  'image/jpeg'
];

const validateFileType = (file) => {
  if (!ALLOWED_MIME_TYPES.includes(file.mimetype)) {
    throw new Error('Invalid file type');
  }
  
  // Verify file extension matches MIME type
  const ext = path.extname(file.originalname).toLowerCase();
  // Additional validation...
};
```

### File Size Limits
- Enforce maximum file size (e.g., 10MB)
- Monitor total storage usage per user
- Implement rate limiting on uploads

### File Storage
- Store files outside web root
- Use unique, unpredictable filenames
- Implement virus scanning for uploads
- Set proper file permissions (read-only for web server)

```javascript
const uploadPath = path.join(__dirname, '../uploads');
const filename = `${uuid.v4()}-${Date.now()}${path.extname(file.originalname)}`;
const filepath = path.join(uploadPath, filename);
```

## Database Security

### Connection Security
- Use SSL/TLS for database connections
- Limit database user permissions
- Use separate credentials for different environments
- Rotate credentials regularly

### Query Security
```javascript
// Use parameterized queries
const user = await User.findOne({ where: { email: email } });

// Avoid raw queries, but if necessary:
const results = await db.query(
  'SELECT * FROM users WHERE email = $1',
  [email]
);
```

### Sensitive Data
- Hash passwords with bcrypt (cost factor 12+)
- Encrypt sensitive fields at rest
- Use separate encryption keys for different data types
- Implement key rotation

```javascript
const bcrypt = require('bcrypt');
const SALT_ROUNDS = 12;

// Hash password
const hashedPassword = await bcrypt.hash(password, SALT_ROUNDS);

// Verify password
const isValid = await bcrypt.compare(password, hashedPassword);
```

## API Security

### HTTPS
- Enforce HTTPS in production
- Use HSTS headers
- Redirect HTTP to HTTPS

```javascript
app.use((req, res, next) => {
  if (process.env.NODE_ENV === 'production' && !req.secure) {
    return res.redirect('https://' + req.headers.host + req.url);
  }
  next();
});
```

### CORS
```javascript
const cors = require('cors');

app.use(cors({
  origin: process.env.FRONTEND_URL,
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

### Rate Limiting
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});

app.use('/api/', limiter);
```

### Request Size Limits
```javascript
app.use(express.json({ limit: '1mb' }));
app.use(express.urlencoded({ extended: true, limit: '1mb' }));
```

## Session Security

### Session Configuration
```javascript
const session = require('express-session');

app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: true, // HTTPS only
    httpOnly: true, // Prevent XSS
    maxAge: 3600000, // 1 hour
    sameSite: 'strict' // CSRF protection
  }
}));
```

## Logging & Monitoring

### Audit Logging
Log all security-relevant events:
- Authentication attempts (success/failure)
- Authorization failures
- Data access
- Configuration changes
- Administrative actions

```javascript
const auditLog = async (userId, action, entityType, entityId, changes) => {
  await AuditLog.create({
    user_id: userId,
    action: action,
    entity_type: entityType,
    entity_id: entityId,
    changes: JSON.stringify(changes),
    ip_address: req.ip,
    timestamp: new Date()
  });
};
```

### Error Handling
- Never expose stack traces to users
- Log detailed errors server-side
- Return generic error messages to clients

```javascript
app.use((err, req, res, next) => {
  console.error(err.stack); // Log detailed error
  
  res.status(500).json({
    success: false,
    error: 'An internal error occurred' // Generic message
  });
});
```

## Dependency Management

### Keep Dependencies Updated
```bash
# Check for vulnerabilities
npm audit

# Update dependencies
npm update

# Fix vulnerabilities
npm audit fix
```

### Use Security Headers
```javascript
const helmet = require('helmet');

app.use(helmet()); // Sets various HTTP headers
```

## Environment Variables

### Never Commit Secrets
- Use `.env` files (gitignored)
- Use secret management services in production
- Rotate secrets regularly
- Use different secrets for each environment

### Secret Management
```javascript
// Load from environment
const config = {
  jwtSecret: process.env.JWT_SECRET,
  dbPassword: process.env.DB_PASSWORD,
  // Never hardcode secrets!
};

// Validate required secrets
const requiredEnvVars = ['JWT_SECRET', 'DB_PASSWORD', 'SESSION_SECRET'];
requiredEnvVars.forEach(varName => {
  if (!process.env[varName]) {
    throw new Error(`Missing required environment variable: ${varName}`);
  }
});
```

## Security Checklist

- [ ] HTTPS enforced
- [ ] Strong authentication implemented
- [ ] Authorization checks on all endpoints
- [ ] Input validation on all inputs
- [ ] SQL injection prevention
- [ ] XSS prevention
- [ ] CSRF protection
- [ ] Secure file uploads
- [ ] Password hashing with bcrypt
- [ ] JWT with secure configuration
- [ ] Rate limiting implemented
- [ ] Security headers configured
- [ ] Audit logging implemented
- [ ] Error handling (no data leakage)
- [ ] Dependencies regularly updated
- [ ] Secrets in environment variables
- [ ] Database connections encrypted
- [ ] Regular security audits

## Incident Response

1. **Detect**: Monitor logs and alerts
2. **Contain**: Isolate affected systems
3. **Investigate**: Determine scope and cause
4. **Remediate**: Fix vulnerabilities
5. **Document**: Record incident details
6. **Learn**: Update procedures

## Resources

- OWASP Top 10: https://owasp.org/www-project-top-ten/
- OWASP Cheat Sheets: https://cheatsheetseries.owasp.org/
- Node.js Security Best Practices: https://nodejs.org/en/docs/guides/security/

## Security Contact

Report security vulnerabilities to: [security@example.com]

**Do not** open public issues for security vulnerabilities.
