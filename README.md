# ☕ CoffeeQL

**One query language. Every database.**

CoffeeQL is a unified query language that works across PostgreSQL, MySQL, MongoDB, and Redis with the same syntax, same chaining, and same mental model regardless of which database you're talking to.

```js
import { CoffeeQL, PostgresAdapter, MongoAdapter } from 'coffeeql'

const db = new CoffeeQL({
  adapters: {
    'users[]':    new PostgresAdapter({ uri: process.env.PG_URI }),
    'products{}': new MongoAdapter({ uri: process.env.MONGO_URI, db: 'shop' }),
  }
})

await db.connect()

// Cross-adapter JOIN — PostgreSQL + MongoDB in one query
const result = await db.query(`
  users[]
    .where(plan = "pro", active = true)
    .mix(products{} ON users[].id = products{}.user_id)
    .give(users[].name, products{}.title, products{}.price)
    .sort(products{}.price, DESC)
    .cup(10)
`)

console.log(result.rows)
```

No `pg`. No `mongodb`. No `mysql2`. No `ioredis`. Just CoffeeQL.

---

## Why CoffeeQL

| Problem | Without CoffeeQL | With CoffeeQL |
|---|---|---|
| Query PostgreSQL | `SELECT name FROM users WHERE plan = $1` | `users[].where(plan = "pro").give(name)` |
| Query MongoDB | `db.products.find({ category: "coffee" })` | `products{}.where(category = "coffee")` |
| Cross-DB JOIN | Impossible in one query | `users[].mix(products{} ON users[].id = products{}.user_id)` |
| Switch databases | Rewrite all queries | Change adapter config |
| Geo search | PostGIS extension | `.where(location.near(28.6, 77.2, 5km))` |
| AI similarity | pgvector extension | `.where(embed.like("espresso machine").threshold(0.85))` |

---

## Installation

```bash
npm install coffeeql
```

Peer dependencies — install only what you use:

```bash
npm install pg          # PostgreSQL
npm install mongodb     # MongoDB
npm install mysql2      # MySQL
npm install ioredis     # Redis
```

---

## The Two Brackets

CoffeeQL uses two bracket types to distinguish database kinds:

```
users[]          → Structured   → PostgreSQL, MySQL, SQLite
products{}       → Unstructured → MongoDB
session:id{}     → Key-Value    → Redis (namespace:key pattern)
```

Same query methods work on both:

```coffeescript
# PostgreSQL
users[].where(plan = "pro").give(name, email).cup(10)

# MongoDB — identical syntax
products{}.where(category = "coffee").give(name, price).cup(10)

# Redis
session:usr_001{}.give(*)
```

---

## Setup

### Single database

```js
import { CoffeeQL, PostgresAdapter } from 'coffeeql'

const db = new CoffeeQL({
  adapters: {
    'users[]':  new PostgresAdapter({ uri: 'postgresql://user:pass@localhost/mydb' }),
    'orders[]': new PostgresAdapter({ uri: 'postgresql://user:pass@localhost/mydb' }),
  }
})

await db.connect()
```

### Multiple databases

```js
import { CoffeeQL, PostgresAdapter, MongoAdapter, RedisAdapter } from 'coffeeql'

const db = new CoffeeQL({
  adapters: {
    'users[]':        new PostgresAdapter({ uri: PG_URI }),
    'products{}':     new MongoAdapter({ uri: MONGO_URI, db: 'shop' }),
    'session:{}':     new RedisAdapter({ uri: REDIS_URI }),
  }
})

await db.connect()
```

---

## Query Reference

### Reading data

```coffeescript
# Basic read
users[]

# Filter — comma = AND, pipe = OR
users[].where(plan = "pro", active = true)
users[].where(plan = "pro" | plan = "team")
users[].where(city = "Delhi" | city = "Mumbai", age >= 18)

# Select fields
users[].give(name, email, balance)
users[].give(*)

# Sort
users[].sort(balance, DESC)
users[].sort(created_at, ASC)

# Limit
users[].cup(10)

# Full chain
users[]
  .where(plan = "pro", active = true, balance > 1000)
  .give(name, email, city, balance)
  .sort(balance, DESC)
  .cup(20)
```

### Aggregation

```coffeescript
# Group + aggregate
orders[]
  .blend(city)
  .give(city, COUNT() as total, SUM(total) as revenue, AVG(total) as avg)
  .sort(revenue, DESC)

# MongoDB too
products{}
  .blend(category)
  .give(category, COUNT() as count, AVG(price) as avg_price)
```

### Joins

```coffeescript
# Same database JOIN
users[]
  .mix(orders[] ON users[].id = orders[].user_id)
  .give(users[].name, orders[].total, orders[].status)
  .sort(orders[].total, DESC)
  .cup(10)

# Cross-adapter JOIN — PostgreSQL + MongoDB
users[]
  .mix(products{} ON users[].id = products{}.user_id)
  .give(users[].name, products{}.title, products{}.price)
  .cup(10)
```

### Writing data

```coffeescript
# Insert — .pour()
users[].pour({
  name:    "Rahul Sharma",
  email:   "rahul@example.com",
  city:    "Delhi",
  plan:    "pro",
  active:  true,
  balance: 2500.00
})

# Update — .refill()
users[]
  .where(email = "rahul@example.com")
  .refill({ plan: "team", balance: 5000.00 })

# Delete — .spill()
users[]
  .where(email = "rahul@example.com")
  .spill()
```

### Geo search

```coffeescript
# Find within radius — m, km, mi
cafes[]
  .where(location.near(28.6139, 77.2090, 5km))
  .give(name, city, rating)
  .sort(rating, DESC)
  .cup(10)

# Combine with other filters
stores{}
  .where(
    location.near(19.0760, 72.8777, 2mi),
    open = true,
    rating > 4.0
  )
  .give(name, address, rating)
```

### AI similarity search

```coffeescript
# Vector similarity — 0 to 1 threshold
products{}
  .where(embed.like("dark roast espresso machine").threshold(0.85))
  .give(name, price, brand)
  .cup(5)

# Combine with price filter
products{}
  .where(
    embed.like("budget coffee maker").threshold(0.80),
    price < 2000
  )
  .give(name, price)
```

### Time windows

```coffeescript
# Recency filter — s m h d w mo y
events{}.where(created_at.last(7d))
logs{}.where(ts.last(1h), level = "error")
orders[].where(placed_at.last(30d))
```

### Array contains

```coffeescript
products{}.where(tags.has("espresso"))
events{}.where(categories.has("workshop"), tags.has("beginner"))
```

---

## Adapter Reference

### PostgresAdapter

```js
import { PostgresAdapter } from 'coffeeql'

new PostgresAdapter({
  uri: 'postgresql://user:pass@localhost:5432/dbname'
})

// Or connection object
new PostgresAdapter({
  host:     'localhost',
  port:     5432,
  user:     'boba',
  password: 'coffee',
  database: 'mydb',
})
```

Works with: **Supabase, Neon, AWS RDS, Railway, local PostgreSQL**

---

### MongoAdapter

```js
import { MongoAdapter } from 'coffeeql'

new MongoAdapter({
  uri: 'mongodb://user:pass@localhost:27017',
  db:  'mydb'
})
```

Works with: **MongoDB Atlas, self-hosted, Docker**

---

### MySQLAdapter

```js
import { MySQLAdapter } from 'coffeeql'

new MySQLAdapter({
  uri: 'mysql://user:pass@localhost:3306/dbname'
})
```

Works with: **PlanetScale, AWS RDS MySQL, self-hosted MySQL 8.0**

> Note: CoffeeQL handles MySQL differences automatically — booleans as `1/0`, UUID as `VARCHAR(36)`, etc.

---

### RedisAdapter

```js
import { RedisAdapter } from 'coffeeql'

new RedisAdapter({
  uri: 'redis://:password@localhost:6379'
})
```

Uses the `namespace:key{}` pattern:

```coffeescript
session:sess_001{}.give(*)           # → HGETALL
cart:usr_003{}.give(items, total)    # → HMGET
session:old{}.spill()               # → DEL
leaderboard:top{}.give(*).cup(10)   # → ZREVRANGE
```

Works with: **Redis Cloud, Upstash, self-hosted Redis 7**

---

## Schema — grind & menu

```coffeescript
# Define structured collection
grind users[] (
  id        UUID      PRIMARY,
  name      TEXT      NOT NULL,
  email     TEXT      UNIQUE NOT NULL,
  age       INT,
  city      TEXT,
  plan      TEXT,
  balance   FLOAT,
  active    BOOL,
  joined_at DATETIME,
  location  GEOPOINT,
  embedding VECTOR
)

# Define unstructured — no schema needed
grind products{}
grind events{}

# Inspect
menu()              # list all collections
menu(users[])       # show schema of users
```

### Data types

| Type | PostgreSQL | MySQL | SQLite |
|---|---|---|---|
| UUID | UUID | VARCHAR(36) | TEXT |
| TEXT | TEXT | TEXT | TEXT |
| INT | INTEGER | INT | INTEGER |
| FLOAT | NUMERIC | DECIMAL(10,2) | REAL |
| BOOL | BOOLEAN | TINYINT(1) | INTEGER |
| DATETIME | TIMESTAMPTZ | DATETIME | INTEGER |
| GEOPOINT | POINT | POINT | TEXT |
| VECTOR | vector[] | JSON | TEXT |

---

## Transactions

```coffeescript
shot {
  users[].where(id = "usr_001").refill({ balance: 4500 })
  /
  orders[].pour({ user_id: "usr_001", total: 500, status: "pending" })
  /
  payments[].pour({ order_id: "ord_new", amount: 500, method: "UPI" })
}
```

All succeed or all rollback. No partial writes.

---

## Built-in functions

```coffeescript
# In .pour() — auto-generate values
users[].pour({
  id:        uuid(),    # UUID v4
  name:      "Rahul",
  joined_at: now(),     # Unix timestamp ms
  date:      today()    # "2026-05-22"
})

# In .give() after .blend() — aggregates
orders[]
  .blend(city)
  .give(
    city,
    COUNT()       as total_orders,
    SUM(total)    as revenue,
    AVG(total)    as avg_order,
    MAX(total)    as biggest,
    MIN(total)    as smallest
  )
```

---

## Parse-only mode (WASM)

If you only need query validation or planning without execution:

```js
import { coffeeql, isValid } from 'coffeeql'

// Validate
isValid('users[].where(age > 18).cup(10)')  // → true
isValid('users.where(age > 18)')             // → false (missing brackets)

// Parse + plan
const result = coffeeql('users[].where(plan = "pro").give(name).cup(10)')
result.success  // true
result.plan     // execution plan string
result.ast      // abstract syntax tree
result.error    // null or error message
```

Works in **Node.js, Bun, Deno, and the browser** — compiled to WebAssembly.

---

## All operators

| Operator | Meaning | Example |
|---|---|---|
| `=` | Equal | `plan = "pro"` |
| `!=` | Not equal | `status != "cancelled"` |
| `>` | Greater than | `balance > 1000` |
| `<` | Less than | `price < 500` |
| `>=` | Greater or equal | `age >= 18` |
| `<=` | Less or equal | `age <= 65` |
| `,` | AND | `active = true, age > 18` |
| `\|` | OR | `city = "Delhi" \| city = "Mumbai"` |
| `!` | NOT | `!active` |
| `EXISTS` | Field is not null | `brand EXISTS` |

---

## All keywords

| Keyword | What it does |
|---|---|
| `collection[]` | Structured collection — SQL |
| `collection{}` | Unstructured collection — document |
| `ns:key{}` | Redis key-value hash |
| `.where()` | Filter records |
| `.give()` | Select fields |
| `.sort()` | Order results |
| `.cup()` | Limit results |
| `.blend()` | Group by field |
| `.mix()` | Join two collections |
| `.pour()` | Insert record |
| `.refill()` | Update records |
| `.spill()` | Delete records |
| `grind` | Define collection schema |
| `menu()` | List / inspect collections |
| `shot{}` | Atomic transaction block |

---

## Error messages

CoffeeQL errors are readable:

```
☕ users needs a type!
   Hint: Use users[] for a table, users{} for a document

☕ Wrong chain order!
   .give() cannot come after .cup()
   Hint: Give comes before Cup — order matters.

☕ No adapter for 'products{}'.
   Register it in new CoffeeQL({ adapters: { 'products{}': new MongoAdapter(...) } })

☕ Too hot! Query took too long.
   Hint: Add .cup() to limit results, or add an index
```

---

## What's next

- [ ] Benchmarks — federation latency vs native queries
- [ ] Failure semantics — partial results, retries, timeouts
- [ ] SQLite adapter
- [ ] TypeScript types

---

## Roadmap

| Version | Status | What |
|---|---|---|
| v0.1.0 | ✅ Published | Parser, planner, WASM, parse-only mode |
| v0.2.0 | 🔨 Building | Real adapter execution, federation engine |
| v0.3.0 | 📋 Planned | Explain plans, benchmarks |
| v0.4.0 | 📋 Planned | Failure semantics, retries, timeouts |

---

## License

MIT

---

> CoffeeQL is in active development. v0.2.0 brings real execution, adapters that actually connect, query, and return data from live databases.

---

*Made with ☕ by [Khushvi Bamrolia](https://www.linkedin.com/in/Khusshvii)*
*Checkout [Khushvi's Github](https://github.com/KhushviB)*

## Star this REPO & FOLLOW for daily new sample backend codes. Starting → 25-05-2026 (Monday)
---
## Links

- 📖 **Docs** → [coffeeql.dev](https://coffeeql.dev)
- 📦 **npm** → [npmjs.com/package/coffeeql](https://npmjs.com/package/coffeeql)
- ⭐ **GitHub** → [github.com/KhushviB/coffeeql](https://github.com/KhushviB/coffeeql)

## License

MIT © CoffeeQL
