# Session Example: Redis Cache Implementation

This session implements a caching layer using Redis for API responses.

## Code Written

```javascript
// cache.js
const redis = require('redis');
const client = redis.createClient();

async function getCachedData(key) {
  const cached = await client.get(key);
  if (cached) {
    return JSON.parse(cached);
  }
  return null;
}

async function setCachedData(key, data, ttl = 3600) {
  await client.set(key, JSON.stringify(data), 'EX', ttl);
}

async function fetchWithCache(url) {
  const cacheKey = `api:${url}`;

  // Try cache first
  const cached = await getCachedData(cacheKey);
  if (cached) {
    console.log('Cache hit');
    return cached;
  }

  // Cache miss - fetch from API
  console.log('Cache miss - fetching from API');
  const response = await fetch(url);
  const data = await response.json();

  // Store in cache with 1 hour TTL
  await setCachedData(cacheKey, data, 3600);

  return data;
}

module.exports = { fetchWithCache };
```

## Expected Concepts

**Layer 1 (Surface):**
- `redis.get()` - Reading from cache
- `redis.set()` - Writing to cache with expiration
- TTL (Time To Live) - Cache expiration
- Cache key patterns - Structured naming (`api:${url}`)
- JSON serialization - `JSON.parse()` / `JSON.stringify()`

**Layer 2 (Underlying):**
- Cache invalidation strategies - When and how to expire data
- Cache hit/miss patterns - Performance optimization
- Consistency guarantees - Stale data vs fresh data trade-offs
- Cache stampede prevention - What happens when cache expires under load
