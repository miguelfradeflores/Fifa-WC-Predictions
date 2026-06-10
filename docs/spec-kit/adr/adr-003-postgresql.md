# ADR-003

**Title:**
PostgreSQL as Primary Database

**Status:**
Accepted

**Context:**
The pool needs strong relational data (users, pools, matches, predictions, results) and
reproducible, consistent ranking calculations. Transactions and referential integrity
are required.

**Decision:**
PostgreSQL will be the primary database.

**Consequences:**

- ACID transactions and strong referential integrity
- Support for complex ranking and aggregation queries
- Recalculation of scores/rankings from source facts (Principle 16)
- Requires schema migration management
