# Redis

Redis is an in-memory key-value store often used as cache, message broker and data structure server. In TeSS, Redis is used for background job storage, temporary caching and coordination between workers.

> **Note:** If you are not familiar with Redis, consult the official docs first: https://redis.io/documentation.

## What Redis is used for in this project

- Sidekiq queue storage and job state management.
- Geocoding cache for event location lookup.
- Worker coordination for delayed geocoding tasks.
- Temporary token storage in external integration clients.
- Test support by flushing Redis in test setup.

## Main files to inspect

- `config/application.rb` — centralized `redis_url` configuration.
- `config/initializers/sidekiq.rb` — Sidekiq Redis connection setup.
- `config/cable.yml` — Action Cable Redis adapter configuration.
- `app/models/event.rb` — Redis caching for geocode lookups and scheduling.
- `app/workers/geocoding_worker.rb` — worker using Redis for location cache coordination.
- `app/models/concerns/has_test_job.rb` — stores test job IDs in Redis.
- `lib/fairsharing/client.rb` — stores Fairsharing token data in Redis.
- `test/test_helper.rb` — flushes Redis before test runs.
- `test/unit/config_test.rb` — validates Redis URL configuration.

## Project examples

### Sidekiq Redis configuration

`config/initializers/sidekiq.rb` configures both server and client:

```ruby
Sidekiq.configure_server do |config|
  config.redis = { url: TeSS::Config.redis_url }
end

Sidekiq.configure_client do |config|
  config.redis = { url: TeSS::Config.redis_url }
end
```

This means all Sidekiq jobs are persisted and managed through the shared Redis instance.

### Geocode cache example

`app/models/event.rb` checks Redis before requesting external geocoding:

```ruby
redis = Redis.new(url: TeSS::Config.redis_url)
if redis.exists?(location)
  self.latitude, self.longitude = JSON.parse(redis.get(location))
end
```

If coordinates are missing, the code stores them for reuse:

```ruby
redis.set(location, [latitude, longitude].to_json)
```

This reduces repeated calls to the external geocoding service.

### Worker coordination example

`Event#enqueue_geocoding_worker` writes a timestamp to Redis to pace background jobs:

```ruby
last_geocode = redis.get('last_geocode') || Time.now
run_at = [last_geocode.to_i, Time.now.to_i].max + NOMINATIM_DELAY
redis.set('last_geocode', run_at)
GeocodingWorker.perform_at(run_at, [id, location])
```

That prevents geocoding workers from submitting too frequently.

### External token cache example

`lib/fairsharing/client.rb` reuses Redis for OAuth-like token storage:

```ruby
redis = Redis.new(url: TeSS::Config.redis_url)
expiry = redis.hget(REDIS_KEY, 'expiry')
```

This keeps API tokens available across requests and processes.

### Test setup example

`test/test_helper.rb` clears Redis before tests:

```ruby
redis = Redis.new(url: TeSS::Config.redis_url)
redis.flushdb
```

This ensures a clean Redis state for each test run.

## Practical notes

- Redis connection is configured by `TeSS::Config.redis_url`.
- Production and test environments may use different Redis databases via `REDIS_URL` and `REDIS_TEST_URL`.
- Redis is used for both Sidekiq and application-level caching/coordination.
