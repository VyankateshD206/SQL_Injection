# SQL Injection Demo - Phase 1 Complete

A deliberately vulnerable web application demonstrating SQL injection attacks, built for educational purposes in the Cybersecurity course.

## Project Structure

```
.
├── docker-compose.yml        # Docker orchestration
├── .env                       # Environment variables
├── database/
│   ├── init.sql              # Database schema & sample data
│   └── Dockerfile            # MySQL image with initialization
├── backend/
│   ├── app.js                # Express server with vulnerable endpoints
│   ├── db.js                 # Database connection
│   ├── routes/
│   │   ├── auth.js           # Login endpoint (vulnerable to logic-altering SQLi)
│   │   └── search.js         # Search endpoint (vulnerable to UNION-based SQLi)
│   ├── package.json
│   └── Dockerfile
└── frontend/
    ├── src/
    │   ├── App.js            # Main React component
    │   ├── components/
    │   │   ├── LoginForm.js   # Login page with injection demos
    │   │   └── SearchForm.js  # Search page with injection demos
    │   └── api/
    │       └── client.js      # Axios HTTP client
    ├── public/
    └── Dockerfile
```

## Getting Started

### Prerequisites

- Docker
- Docker Compose
- (Optional) Node.js 18+ for local development

### Running the Application

1. **Start all containers:**
   ```bash
   docker compose up --build
   ```

2. **Access the application:**
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:5000
   - MySQL: localhost:3306

3. **Stop the application:**
   ```bash
   docker compose down
   ```

## Database

### Credentials
- Username: `sqli_user`
- Password: `demo_password`
- Database: `sqli_db`

### Test Users
| Username | Password | Role |
|----------|----------|------|
| admin | password123 | admin |
| user1 | user_pass | user |
| user2 | secure_pw | user |
| moderator | mod_secret | moderator |

## API Endpoints

### Authentication
- `POST /api/auth/login` - Login (vulnerable to logic-altering SQLi)
- `GET /api/auth/users` - Get all users

### Search
- `GET /api/search?query=<value>` - Search records (vulnerable to UNION-based SQLi)
- `GET /api/search/advanced?column=<col>&value=<value>` - Advanced search (double vulnerability)

## Vulnerable Code Examples

### Login Endpoint (auth.js)
```javascript
const query = `SELECT * FROM users WHERE username = '${username}' AND password = '${password}'`;
```

**Attack:** `admin' OR '1'='1 --` in username field bypasses password check

### Search Endpoint (search.js)
```javascript
const query = `SELECT id, title, content FROM records WHERE title LIKE '%${query}%'`;
```

**Attack:** `' UNION SELECT id, secret_key, secret_value FROM secrets --` extracts secrets

## Phase 1 Deliverables

✅ Docker environment setup (docker-compose.yml, .env, .dockerignore)
✅ Database schema with 3 tables (users, records, secrets)
✅ Sample data for realistic demonstration
✅ Vulnerable backend API with Express
✅ React frontend with interactive UI
✅ Two vulnerable endpoints ready for Phase 2 attack demo
✅ Demo payload hints in frontend for testing

## Next Steps

**Phase 2:** Attack Demonstration
- Execute SQL injection payloads
- Document successful attacks
- Capture results and evidence

**Phase 3:** Defense Implementation
- Replace vulnerable code with prepared statements
- Implement input validation
- Re-run Phase 2 payloads to verify fixes

## Important Notes

⚠️ **This is deliberately vulnerable code for educational purposes only.**
- Do NOT use these patterns in production
- This environment is isolated and safe for testing
- All vulnerabilities are intentional for learning

## Team

- Aaryan Agrawal (B23CS1002)
- Vyankatesh Deshpande (B23CS1079)
- Kanishk Singh Dayma (B23CS1024)
- Abhinash Roy (B23CS1003)

**Course:** Cybersecurity (CSL6010)
**Instructor:** Mohit Kumar Jangid
