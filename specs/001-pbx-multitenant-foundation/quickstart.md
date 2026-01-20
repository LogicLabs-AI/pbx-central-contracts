# Quickstart: Multitenant PBX Foundation

## Prerequisites

- **Rust**: 1.75+ (`rustup update`)
- **Node.js**: 20+
- **PostgreSQL**: 15+ (Running on default port 5432)
- **FreeSWITCH**: 1.10+ (Installed locally or accessible via network)
- **FFmpeg**: Installed and in PATH

## Configuration

1. **Environment Variables**:
   Create a `.env` file in `pbx-server/`:
   ```bash
   DATABASE_URL=postgres://postgres:password@localhost:5432/pbx_central
   ESL_HOST=127.0.0.1
   ESL_PORT=8021
   ESL_PASSWORD=ClueCon
   MEDIA_ROOT=/var/lib/pbx/media
   JWT_SECRET=your_dev_secret_key
   ```

2. **Database Setup**:
   ```bash
   # Install sqlx-cli
   cargo install sqlx-cli

   # Create DB
   cd pbx-server
   sqlx database create
   sqlx migrate run
   ```

## Running the Backend

```bash
cd pbx-server
cargo run
# Server will listen on http://127.0.0.1:3000
```

## Running the Frontend

```bash
cd pbx-web
npm install
npm run dev
# Dashboard at http://localhost:5173
```

## Testing

### Unit Tests
```bash
cd pbx-server
cargo test
```

### SIP E2E Test (Manual)
1. Login to Dashboard (seed user: `admin@global.com` / `password`).
2. Create Tenant.
3. Create Extension 1000.
4. Copy Credentials.
5. Register a Softphone (Zoiper or SIP.js demo) to FreeSWITCH IP.
6. Verify registration in FS CLI: `sofia status profile internal`.
