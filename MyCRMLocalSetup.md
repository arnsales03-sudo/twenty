# Twenty CRM - Local Development Setup Guide (Windows)

This guide provides complete step-by-step instructions for setting up Twenty CRM on Windows without Docker.

## Prerequisites

### 1. Node.js v24.5.0
- **Download**: https://nodejs.org/
- **Or use NVM for Windows**: https://github.com/coreybutler/nvm-windows/releases
- **Installation with NVM**:
  ```cmd
  nvm install 24.5.0
  nvm use 24.5.0
  node --version
  ```
- **Required Version**: v24.5.0 (specified in `.nvmrc`)

### 2. Yarn v4.9.2
- Yarn should be automatically available after Node.js installation
- Verify version:
  ```cmd
  yarn --version
  ```
- Should show: `4.9.2`

### 3. PostgreSQL 18
- **Download**: https://www.postgresql.org/download/windows/
- **Installation Steps**:
  1. Download the Windows installer
  2. Run the installer
  3. Set password: `postgres` (or remember your custom password)
  4. Default port: `5432`
  5. Complete the installation
- **Verify Installation**:
  - Open pgAdmin 4 (installed with PostgreSQL)
  - Or check service is running in Windows Services

### 4. Redis (Memurai for Windows)
- **Download**: https://www.memurai.com/get-memurai
- **Alternative**: https://github.com/tporadowski/redis/releases
- **Installation Steps**:
  1. Download Memurai Developer Edition
  2. Run the `.msi` installer
  3. Complete the installation
  4. Service starts automatically
- **Verify Installation**:
  ```cmd
  netstat -an | findstr ":6379"
  ```
  - Should show: `TCP    127.0.0.1:6379         0.0.0.0:0              LISTENING`

## Setup Steps

### Step 1: Clone the Repository
```cmd
git clone https://github.com/twentyhq/twenty.git
cd twenty
```

### Step 2: Switch to Correct Node Version
```cmd
nvm use 24.5.0
```

### Step 3: Install Dependencies
```cmd
yarn install
```
- This will take 5-10 minutes on first run
- Downloads ~2.6 GB of packages

### Step 4: Create Database

#### Option A: Using psql command line
```cmd
psql -U postgres
CREATE DATABASE twenty;
\q
```

#### Option B: Using pgAdmin
1. Open pgAdmin 4
2. Connect to PostgreSQL 18 server
3. Right-click on "Databases"
4. Select "Create" → "Database"
5. Name: `twenty`
6. Click "Save"

### Step 5: Create Database Schema
```cmd
psql -U postgres -d twenty
CREATE SCHEMA IF NOT EXISTS core;
\q
```

### Step 6: Configure Environment Variables

#### Backend Configuration
Copy the example environment file:
```cmd
copy packages\twenty-server\.env.example packages\twenty-server\.env
```

Edit `packages/twenty-server/.env`:
```env
NODE_ENV=development
PG_DATABASE_URL=postgres://postgres:postgres@localhost:5432/twenty
REDIS_URL=redis://localhost:6379
APP_SECRET=your_random_secret_key_change_this_in_production_12345
SIGN_IN_PREFILLED=true
FRONTEND_URL=http://localhost:3001
```

**Important**: Replace `APP_SECRET` with a secure random string in production.

#### Frontend Configuration
Copy the example environment file:
```cmd
copy packages\twenty-front\.env.example packages\twenty-front\.env
```

The default values should work:
```env
REACT_APP_SERVER_BASE_URL=http://localhost:3000
VITE_BUILD_SOURCEMAP=false
```

### Step 7: Run Database Migrations
```cmd
yarn nx run twenty-server:database:migrate
```
- This creates all necessary database tables and schemas
- Should complete with: "Successfully ran target database:migrate"

### Step 8: Fix Backend Start Script (Windows Compatibility)

Edit `packages/twenty-server/project.json`:

Find the `"start"` target and change:
```json
"start": {
  "executor": "nx:run-commands",
  "cache": false,
  "dependsOn": ["build"],
  "options": {
    "cwd": "packages/twenty-server",
    "command": "NODE_ENV=development nest start --watch"
  }
},
```

To:
```json
"start": {
  "executor": "nx:run-commands",
  "cache": false,
  "dependsOn": ["build"],
  "options": {
    "cwd": "packages/twenty-server",
    "command": "nest start --watch"
  }
},
```

### Step 9: Start the Application

#### Option A: Start Both Services Together
```cmd
yarn start
```

#### Option B: Start Services Separately (Recommended for Development)

**Terminal 1 - Backend:**
```cmd
yarn nx run twenty-server:start
```

**Terminal 2 - Frontend:**
```cmd
yarn nx run twenty-front:start
```

**Terminal 3 - Worker (Optional):**
```cmd
yarn nx run twenty-server:worker
```

### Step 10: Access the Application

- **Frontend**: http://localhost:3001
- **Backend API**: http://localhost:3000
- **GraphQL Playground**: http://localhost:3000/graphql

## Verification Checklist

### Check Services are Running

1. **PostgreSQL**:
   ```cmd
   netstat -an | findstr ":5432"
   ```
   Should show: `LISTENING` on port 5432

2. **Redis (Memurai)**:
   ```cmd
   netstat -an | findstr ":6379"
   ```
   Should show: `LISTENING` on port 6379

3. **Backend Server**:
   ```cmd
   curl http://localhost:3000/healthz
   ```
   Should return: `{"status":"ok"}`

4. **Frontend**:
   Open browser: http://localhost:3001
   Should show: "Welcome to Twenty" screen

## Common Issues and Solutions

### Issue 1: "Node version doesn't match"
**Solution**:
```cmd
nvm use 24.5.0
node --version
```
Close and reopen terminal if version doesn't change.

### Issue 2: "Unable to Reach Back-end"
**Causes**:
- Backend not running
- PostgreSQL not running
- Redis not running
- Database not created

**Solution**:
1. Check all services are running (see Verification Checklist)
2. Check backend logs for errors
3. Verify database connection in `.env` file

### Issue 3: "schema 'core' does not exist"
**Solution**:
```cmd
psql -U postgres -d twenty -c "CREATE SCHEMA IF NOT EXISTS core;"
```

### Issue 4: "NODE_ENV is not recognized"
**Solution**: This is a Windows issue. Follow Step 8 to fix the start script.

### Issue 5: Port Already in Use
**Solution**:
```cmd
# Find process using port 3000
netstat -ano | findstr :3000

# Kill the process (replace PID with actual process ID)
taskkill /PID <PID> /F
```

### Issue 6: Yarn Install Fails
**Solution**:
1. Clear yarn cache:
   ```cmd
   yarn cache clean
   ```
2. Delete `node_modules` and `.yarn/cache`:
   ```cmd
   rmdir /s /q node_modules
   rmdir /s /q .yarn\cache
   ```
3. Reinstall:
   ```cmd
   yarn install
   ```

## Database Management

### Reset Database
```cmd
yarn nx run twenty-server:database:reset
```

### Create New Migration
```cmd
yarn nx run twenty-server:database:migrate:generate --args.migrationName=YourMigrationName
```

### Revert Last Migration
```cmd
yarn nx run twenty-server:database:migrate:revert
```

## Development Commands

### Run Tests
```cmd
# Unit tests
yarn nx run twenty-server:test

# Integration tests
yarn nx run twenty-server:test:integration
```

### Lint Code
```cmd
yarn nx run twenty-server:lint
yarn nx run twenty-front:lint
```

### Build for Production
```cmd
yarn nx run twenty-server:build
yarn nx run twenty-front:build
```

### Type Check
```cmd
yarn nx run twenty-server:typecheck
yarn nx run twenty-front:typecheck
```

## Project Structure

```
twenty/
├── packages/
│   ├── twenty-front/          # React frontend application
│   ├── twenty-server/          # NestJS backend application
│   ├── twenty-emails/          # Email templates
│   ├── twenty-ui/              # UI component library
│   ├── twenty-shared/          # Shared utilities
│   └── ...
├── .env                        # Root environment variables
├── package.json                # Root package configuration
├── nx.json                     # Nx workspace configuration
└── yarn.lock                   # Dependency lock file
```

## Environment Variables Reference

### Backend (.env in packages/twenty-server/)

| Variable | Description | Default |
|----------|-------------|---------|
| `NODE_ENV` | Environment mode | `development` |
| `PG_DATABASE_URL` | PostgreSQL connection string | `postgres://postgres:postgres@localhost:5432/twenty` |
| `REDIS_URL` | Redis connection string | `redis://localhost:6379` |
| `APP_SECRET` | Secret key for JWT tokens | (generate random string) |
| `FRONTEND_URL` | Frontend application URL | `http://localhost:3001` |
| `PORT` | Backend server port | `3000` |

### Frontend (.env in packages/twenty-front/)

| Variable | Description | Default |
|----------|-------------|---------|
| `REACT_APP_SERVER_BASE_URL` | Backend API URL | `http://localhost:3000` |
| `VITE_BUILD_SOURCEMAP` | Generate source maps | `false` |

## Useful Links

- **Official Documentation**: https://docs.twenty.com/
- **GitHub Repository**: https://github.com/twentyhq/twenty
- **Discord Community**: https://discord.gg/cx5n4Jzs57
- **Contributing Guide**: https://docs.twenty.com/developers/contributing
- **API Documentation**: https://docs.twenty.com/developers/api

## Additional Resources

### PostgreSQL
- **Documentation**: https://www.postgresql.org/docs/
- **pgAdmin**: https://www.pgadmin.org/

### Redis/Memurai
- **Memurai Documentation**: https://docs.memurai.com/
- **Redis Documentation**: https://redis.io/documentation

### Node.js & Yarn
- **Node.js Documentation**: https://nodejs.org/docs/
- **Yarn Documentation**: https://yarnpkg.com/

### NestJS (Backend Framework)
- **Documentation**: https://docs.nestjs.com/

### React (Frontend Framework)
- **Documentation**: https://react.dev/

## Support

If you encounter issues:
1. Check the [GitHub Issues](https://github.com/twentyhq/twenty/issues)
2. Join the [Discord Community](https://discord.gg/cx5n4Jzs57)
3. Read the [Documentation](https://docs.twenty.com/)

## Next Steps

After successful setup:
1. Create your first workspace
2. Explore the CRM features
3. Read the developer documentation
4. Start customizing for your needs

---

**Setup Date**: 2024
**Twenty Version**: 0.2.1
**Guide Version**: 1.0
