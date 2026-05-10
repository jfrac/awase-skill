# Session Example: Promise Error Handling

This session refactors async code to handle errors properly with Promises.

## Code Written

```javascript
// api-client.js
async function fetchUserData(userId) {
  const response = await fetch(`/api/users/${userId}`);
  if (!response.ok) {
    throw new Error(`Failed to fetch user ${userId}`);
  }
  return response.json();
}

async function fetchMultipleUsers(userIds) {
  // Before: Promise.all - fails fast on first error
  // const users = await Promise.all(
  //   userIds.map(id => fetchUserData(id))
  // );

  // After: Promise.allSettled - continues despite errors
  const results = await Promise.allSettled(
    userIds.map(id => fetchUserData(id))
  );

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);

  const failed = results
    .filter(r => r.status === 'rejected')
    .map((r, idx) => ({ userId: userIds[idx], error: r.reason }));

  if (failed.length > 0) {
    console.warn('Some users failed to load:', failed);
  }

  return successful;
}

async function processWithRetry(fn, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      console.log(`Retry ${i + 1}/${retries}`);
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}

// Usage
processWithRetry(() => fetchUserData(123))
  .then(user => console.log('User loaded:', user))
  .catch(error => console.error('Failed after retries:', error));
```

## Expected Concepts

**Layer 1 (Surface):**
- `Promise.all()` - Parallel execution, fail fast
- `Promise.allSettled()` - Parallel execution, wait for all
- `.catch()` error handling - Catching rejected promises
- `async/await` syntax - Modern async code
- `throw new Error()` - Explicit error throwing

**Layer 2 (Underlying):**
- Error propagation in async code - How errors bubble up
- Concurrent vs sequential execution - Performance trade-offs
- Retry strategies - Exponential backoff patterns
- Graceful degradation - Partial success handling
- Race conditions - Multiple async operations timing
