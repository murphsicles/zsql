# ZSQL

[![Zeta](https://img.shields.io/badge/Zeta-Language-%23FF6B6B?style=flat-square)](https://zeta-lang.org)
[![zorbs.io](https://img.shields.io/badge/zorbs.io-@db/zsql-%2300C4FF?style=flat-square)](https://zorbs.io/@db/zsql)
[![GitHub](https://img.shields.io/badge/GitHub-murphsicles/zsql-%23181717?style=flat-square)](https://github.com/murphsicles/zsql)

Lightweight embedded SQL database for Zeta. ZSQL wraps SQLite3 — giving you a zero-config, serverless SQL engine backed by the most widely deployed database engine in the world.

ZSQL is a companion to [Zenith](https://zorbs.io/@db/zenith) — use Zenith for fast in-memory key-value workloads and ZSQL for persistent SQL querying.

## Quick Start

Add to your `zorb.toml`:

```toml
[dependencies]
@db/zsql = "1.0"
```

Open a database and execute queries:

```zeta
use @db/zsql::Connection;

fn main() {
    let conn = Connection::open("my_data.db").unwrap();

    conn.execute("CREATE TABLE IF NOT EXISTS users (
        id    INTEGER PRIMARY KEY,
        name  TEXT NOT NULL,
        age   INTEGER
    )", []).unwrap();

    conn.execute("INSERT INTO users (name, age) VALUES (?1, ?2)",
        params_from_iter(["Alice".into(), "30".into()])).unwrap();

    let mut stmt = conn.prepare("SELECT id, name, age FROM users WHERE age > ?1").unwrap();
    let rows = stmt.query_map([25], |row| {
        (row.get::<i32>(0).unwrap(), row.get::<String>(1).unwrap(), row.get::<i32>(2).unwrap())
    }).unwrap();

    for row in rows {
        let (id, name, age) = row.unwrap();
        print("User {}: {} (age {})\n", id, name, age);
    }
}
```

## In-Memory Database

```zeta
let conn = Connection::open_in_memory().unwrap();
conn.execute_batch("CREATE TABLE t (x INTEGER); INSERT INTO t VALUES (1), (2), (3);").unwrap();
```

## Transactions

```zeta
let tx = conn.transaction_with_behavior(TransactionBehavior::Immediate, true).unwrap();
tx.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1", []).unwrap();
tx.commit().unwrap();
```

## API

| Function / Type | Description |
|----------------|-------------|
| `Connection::open(path)` | Open or create a SQLite database file |
| `Connection::open_in_memory()` | Create an in-memory database |
| `conn.execute(sql, params)` | Execute a query |
| `conn.prepare(sql)` | Prepare a statement |
| `stmt.query_map(params, f)` | Execute and map rows |
| `conn.transaction()` | Begin a transaction |
| `Transaction::commit()` / `.rollback()` | Commit or roll back |

## License

MIT
