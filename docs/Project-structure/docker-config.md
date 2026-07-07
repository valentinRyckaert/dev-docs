# Docker files

TeSS uses Docker for containerization. Docker is a technology written in Go which helps to create isolated environment for development or deployement. Concretly, your app will run inside an isolated box which is precisely configured to make your application running properly.

!!! note
    This page is a quick presentation of the content of the docker files.
    If you are not familiar with Docker, we strongly recommend to check these ressources :
    
    - [Docker documentation](https://docs.docker.com/)
    - [Docker in 100 seconds](https://www.youtube.com/watch?v=Gjnup-PuquQ)
    - [Docker full course for beginners (2 hours)](https://www.youtube.com/watch?v=fqMOX6JJhGo)

In the project, you will find 3 files related to Docker.

## docker-compose.yml

This file is used for development purposes.

| Container | Image | Purpose | Ports | Key Dependencies | Profiles |
|-----------|-------|---------|-------|------------------|-------------|
| **app** | Builds from `Dockerfile` (development target) | Main Rails application server | `3000:3000` | db, solr, redis | default |
| **db** | `postgres:14.2` | PostgreSQL relational database for persistent data storage | — | — | default |
| **adminer** | `adminer:5.4.2` | Adminer web interface to securely access postgres db| `8080:8080` | db | default |
| **solr** | `solr:8` | Apache Solr search engine for full-text search and indexing capabilities | `8984:8983` | — | default |
| **redis** | `redis:7` | Redis in-memory cache for session storage, caching, and Sidekiq job queue | — | — | default |
| **sidekiq** | `${PREFIX}-app` (reuses app image) | Background job processor that pulls jobs from Redis queues and executes them asynchronously | — | app, db, redis | default |
| **test** | Builds from `Dockerfile` (production target) | Rails test environment that runs the test suite | — | test-db, redis | **test** |
| **test-db** | `postgres:14.2` | Isolated PostgreSQL database for test environment | — | — | **test** |


## docker-compose-prod.yml

This file is used for production only.

| Container | Image | Purpose | Ports | Key Dependencies | Restart Policy |
|-----------|-------|---------|-------|------------------|-----------------|
| **app** | Builds from `Dockerfile` (production target) | Production Rails application server | `3000:3000` | db, solr, redis | **always** |
| **db** | `postgres:14.2` | PostgreSQL relational database for persistent data storage in production | — | — | **always** |
| **solr** | `solr:8` | Apache Solr search engine for full-text search and indexing | — | — | **always** |
| **redis** | `redis:7` | Redis in-memory cache and job queue for session storage, caching, and Sidekiq background jobs | — | — | **always** |
| **sidekiq** | `${PREFIX}-app` (reuses app image) | Background job processor that executes asynchronous tasks from Redis queues | — | app, db, redis | **always** |
| **dbbackups** | `kartoza/pg-backup:14-3.1` | Automated PostgreSQL database backup service | — | db (waits for healthy status) | **on-failure** |

## Dockerfile

This file creates the tess docker image to be used in the docker-compose system.

#### Base Stage

The `base` stage sets up the foundation for all images:

- **Ruby 3.4.9** slim image is used as the starting point to keep the image lightweight
- **Workdir `/code`** is created as the working directory
- **System dependencies** are installed: build tools, curl, git, imagemagick (for image processing), PostgreSQL client (libpq-dev), Node.js, and npm
- **Yarn 1.22.22** is installed globally to manage JavaScript dependencies
- **Supercronic** (a cron alternative) is downloaded, verified via SHA1 checksum, and installed to handle scheduled tasks inside containers
- **Entrypoint** is set to `docker/entrypoint.sh` which runs every time a container starts
- **Port 3000** is exposed for the Rails server

#### Development Stage

The development target **inherits from base** and simply runs:

```
bundle exec rails server -b 0.0.0.0
```

This starts the Rails development server without any asset precompilation or optimization, allowing hot-reloading during development.

#### Production Stage

The production target **inherits from base** and performs a multi-step build:

1. **Copies Gemfile files** and **installs Ruby gems** with `bundle install`
2. **Copies package files** and **installs JavaScript dependencies** with yarn using frozen lockfile (ensuring exact versions)
3. **Copies the entire application code** into the image
4. **Precompiles assets** using `bundle exec rake assets:precompile` for performance
5. **Cleans up sensitive files** (if `CR=True` for container registry): removes `tess.yml`, `secrets.yml`, and `.env` files, then adjusts permissions to allow group read/write access
6. **Runs at startup**: generates crontab from the whenever gem, starts supercronic in the background to run scheduled tasks, and launches the Rails server

The production image is **self-contained** with all dependencies baked in, while sensitive configuration files are mounted separately at runtime.