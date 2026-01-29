# GYM-STUDY Infrastructure

Docker Compose configuration for running the complete GYM-STUDY stack.

## Services

- **PostgreSQL** (port 5432): Database
- **Redis** (port 6379): Cache and sessions
- **Backend** (port 5000): Node.js/Express API
- **Frontend** (port 3000): Next.js application

## Quick Start

1. Copy the environment file:
```bash
cp .env.example .env
```

2. Edit `.env` and set your JWT_SECRET (must be at least 32 characters):
```bash
JWT_SECRET=your_super_secret_key_change_in_production_min_32_chars_here
```

3. Start all services:
```bash
docker-compose up -d
```

4. Run database migrations:
```bash
docker exec gym-study-backend npm run migration:run
```

5. Check logs:
```bash
docker-compose logs -f
```

6. Access the application:
- Frontend: http://localhost:3000
- Backend API: http://localhost:5000
- API Health: http://localhost:5000/health

## Development Mode

To run in development mode with hot reload:

```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up
```

## Useful Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# Stop and remove volumes (CAUTION: deletes database)
docker-compose down -v

# View logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f backend

# Rebuild services
docker-compose build

# Access PostgreSQL
docker exec -it gym-study-postgres psql -U postgres -d gym_study

# Access Redis CLI
docker exec -it gym-study-redis redis-cli

# Run migrations
docker exec gym-study-backend npm run migration:run

# Access backend shell
docker exec -it gym-study-backend sh
```

## Environment Variables

See `.env.example` for all available environment variables.

### Required Variables
- `JWT_SECRET`: JWT secret key (min 32 chars)
- `DB_PASSWORD`: PostgreSQL password

### Optional Variables
- OAuth credentials (Google, GitHub)
- SMTP settings (for email)
- Rate limiting settings

## Ports

- 3000: Frontend
- 5000: Backend API
- 5432: PostgreSQL
- 6379: Redis

## Volumes

- `postgres_data`: PostgreSQL data
- `redis_data`: Redis data
- `backend_logs`: Backend application logs

## Networks

All services run on the `gym-study-network` bridge network.
