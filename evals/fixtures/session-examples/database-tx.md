# Session Example: Database Transactions

This session implements a bank transfer using database transactions for ACID guarantees.

## Code Written

```javascript
// transfer.js
const { Pool } = require('pg');
const pool = new Pool();

async function transferMoney(fromAccountId, toAccountId, amount) {
  const client = await pool.connect();

  try {
    // Start transaction
    await client.query('BEGIN');

    // Check balance
    const { rows } = await client.query(
      'SELECT balance FROM accounts WHERE id = $1 FOR UPDATE',
      [fromAccountId]
    );

    if (rows[0].balance < amount) {
      throw new Error('Insufficient funds');
    }

    // Debit from sender
    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
      [amount, fromAccountId]
    );

    // Credit to receiver
    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
      [amount, toAccountId]
    );

    // Record transaction
    await client.query(
      'INSERT INTO transactions (from_account, to_account, amount, timestamp) VALUES ($1, $2, $3, NOW())',
      [fromAccountId, toAccountId, amount]
    );

    // Commit transaction
    await client.query('COMMIT');

    console.log(`Transferred $${amount} from ${fromAccountId} to ${toAccountId}`);
  } catch (error) {
    // Rollback on error
    await client.query('ROLLBACK');
    console.error('Transaction failed:', error.message);
    throw error;
  } finally {
    client.release();
  }
}

// Usage
transferMoney(101, 102, 50.00)
  .then(() => console.log('Transfer successful'))
  .catch(error => console.error('Transfer failed:', error));
```

## Expected Concepts

**Layer 1 (Surface):**
- `BEGIN` transaction - Starting atomic operation
- `COMMIT` - Persisting changes
- `ROLLBACK` - Reverting on error
- `FOR UPDATE` - Row-level locking
- ACID properties - Atomicity, Consistency, Isolation, Durability

**Layer 2 (Underlying):**
- Transaction isolation levels - READ COMMITTED vs SERIALIZABLE
- Deadlock prevention - Lock ordering strategies
- Optimistic vs pessimistic locking - When to use each
- Race conditions in concurrent transactions - Lost updates, dirty reads
- Two-phase commit - Distributed transaction patterns
