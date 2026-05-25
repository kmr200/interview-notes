# JDBC

## What is JDBC?

JDBC (Java Database Connectivity) is an industry-standard API for Java applications to interact with various database management systems (DBMS). It is implemented as the `java.sql` package, included in Java SE.

JDBC is based on the concept of drivers, which enable connections to a database using a specially formatted URL. When a driver is loaded, it registers itself in the system and is automatically invoked whenever a program requests a URL with the protocol that the driver supports.

### Advantages of JDBC

- **Ease of development** - Developers do not need to know the specifics of the database they are working with.
- **Minimal code changes when switching databases** - The amount of required modifications depends only on differences in SQL dialects.
- **No need for additional client programs** - JDBC allows direct communication with databases without installing separate database clients.
- **Simple connection process** - Any database can be accessed via an easily described JDBC URL.

---

## JDBC URL

A JDBC URL consists of the following components:

- `<protocol>` - Always starts with `jdbc:`.
- `<subprotocol>` - The driver name or connection mechanism. For example, `odbc` is a common subprotocol for ODBC data sources.
- `<subname>` - The database identifier, which varies depending on the subprotocol. It provides all necessary information for locating the database.

If the database is on the internet, the JDBC URL must include the network address:

```
jdbc:<subprotocol>://<hostname>:<port>/<subname>
```

Example - connecting to a MySQL database named `Test` on localhost via port 3306:

```
jdbc:mysql://localhost:3306/Test
```

---

## JDBC Components

JDBC consists of two main parts:

- **JDBC API** - A set of classes and interfaces that define database access, declared in `java.sql` and `javax.sql`.
- **JDBC Driver** - A database-specific component that translates API calls into native database commands.

### Key Classes and Interfaces

| Class / Interface                    | Description                                                                                |
|--------------------------------------|--------------------------------------------------------------------------------------------|
| `java.sql.DriverManager`             | Loads and registers the required JDBC driver and establishes a connection to the database. |
| `javax.sql.DataSource`               | An alternative to `DriverManager`, offering a more flexible way to obtain connections.     |
| `javax.sql.ConnectionPoolDataSource` | Extends `DataSource` to support connection pooling.                                        |
| `javax.sql.XADataSource`             | Extends `DataSource` to support distributed transactions.                                  |
| `java.sql.Connection`                | Manages queries and transactions.                                                          |
| `javax.sql.PooledConnection`         | Supports connection pooling.                                                               |
| `javax.sql.XAConnection`             | Supports distributed transactions.                                                         |
| `java.sql.Statement`                 | Executes simple SQL queries.                                                               |
| `java.sql.PreparedStatement`         | Executes parameterized SQL queries.                                                        |
| `java.sql.CallableStatement`         | Executes stored procedures.                                                                |
| `java.sql.ResultSet`                 | Reads and navigates through query result sets.                                             |
| `java.sql.ResultSetMetaData`         | Retrieves metadata about a result set.                                                     |
| `java.sql.DatabaseMetaData`          | Provides metadata about the database.                                                      |

### JDBC Data Types and Java Mappings

| JDBC Type       | Java Type               |
|-----------------|-------------------------|
| `CHAR`          | `String`                |
| `VARCHAR`       | `String`                |
| `LONGVARCHAR`   | `String`                |
| `NUMERIC`       | `java.math.BigDecimal`  |
| `DECIMAL`       | `java.math.BigDecimal`  |
| `BIT`           | `Boolean`               |
| `TINYINT`       | `Integer`               |
| `SMALLINT`      | `Integer`               |
| `INTEGER`       | `Integer`               |
| `BIGINT`        | `Long`                  |
| `REAL`          | `Float`                 |
| `FLOAT`         | `Double`                |
| `DOUBLE`        | `Double`                |
| `BINARY`        | `byte[]`                |
| `VARBINARY`     | `byte[]`                |
| `LONGVARBINARY` | `byte[]`                |
| `DATE`          | `java.sql.Date`         |
| `TIME`          | `java.sql.Time`         |
| `TIMESTAMP`     | `java.sql.Timestamp`    |
| `CLOB`          | `Clob`                  |
| `BLOB`          | `Blob`                  |
| `ARRAY`         | `Array`                 |
| `STRUCT`        | `Struct`                |
| `REF`           | `Ref`                   |
| `DISTINCT`      | Corresponding base type |
| `JAVA_OBJECT`   | Any Java base class     |

---

## Main Steps of Working with a Database via JDBC

1. Register JDBC Driver
2. Establish a Connection
3. Create SQL Query
4. Execute SQL Query
5. Process Results
6. Close the Connection

---

## 1. Registering a JDBC Driver

**Option 1 - Using `DriverManager.registerDriver()`:**

```java
DriverManager.registerDriver(new com.mysql.cj.jdbc.Driver());
```

**Option 2 - Using `Class.forName()` with `newInstance()`:**

```java
Class.forName("com.mysql.cj.jdbc.Driver").newInstance();
```

**Option 3 - Using `Class.forName()` (recommended):**

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

---

## 2. Establishing a Database Connection

JDBC uses `DriverManager.getConnection()` to establish a connection. When successful, it returns a `java.sql.Connection` object representing a session with the database.

**Using a database URL only:**

```java
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/Test");
```

**Using a URL with a `Properties` object:**

```java
Properties props = new Properties();
props.setProperty("user", "root");
props.setProperty("password", "password");
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/Test", props);
```

**Using a URL with username and password directly:**

```java
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/Test", "root", "password");
```

---

## Transaction Isolation Levels

Transaction isolation defines the level of visibility of uncommitted changes between concurrent transactions.

| Isolation Level                | Prevents Dirty Read?    | Prevents Non-Repeatable Read? | Prevents Phantom Read? |
|--------------------------------|-------------------------|-------------------------------|------------------------|
| `TRANSACTION_NONE`             | **-** (No transactions) | **-**                         | **-**                  |
| `TRANSACTION_READ_UNCOMMITTED` | **-**                   | **-**                         | **-**                  |
| `TRANSACTION_READ_COMMITTED`   | **+**                   | **-**                         | **-**                  |
| `TRANSACTION_REPEATABLE_READ`  | **+**                   | **+**                         | **-**                  |
| `TRANSACTION_SERIALIZABLE`     | **+**                   | **+**                         | **+**                  |

**Setting the isolation level:**

```java
conn.setTransactionIsolation(Connection.TRANSACTION_SERIALIZABLE);
```

**Getting the current isolation level:**

```java
int level = conn.getTransactionIsolation();
```

> Not all databases support all isolation levels. Check supported levels using `DatabaseMetaData`:
>
> ```java
> DatabaseMetaData metaData = conn.getMetaData();
> boolean supported = metaData.supportsTransactionIsolationLevel(Connection.TRANSACTION_SERIALIZABLE);
> System.out.println("Supports TRANSACTION_SERIALIZABLE: " + supported);
> ```

---

## 3 & 4. Executing Queries

JDBC provides three interfaces for executing SQL:

- `Statement` - for simple SQL statements without parameters.
- `PreparedStatement` - for parameterized queries and frequently executed statements.
- `CallableStatement` - for executing stored procedures.

### Creating Query Objects

```java
Statement stmt         = conn.createStatement();
PreparedStatement pstmt = conn.prepareStatement(sql);
CallableStatement cstmt = conn.prepareCall(sql);
```

### Statement vs. PreparedStatement

| Feature     | `Statement`                   | `PreparedStatement`              |
|-------------|-------------------------------|----------------------------------|
| Query type  | Simple SQL without parameters | Parameterized SQL                |
| Compilation | Compiled on every execution   | Precompiled once, reused         |
| Performance | Lower for repeated execution  | Higher due to query plan caching |
| Security    | Vulnerable to SQL Injection   | Prevents SQL Injection           |

```java
// Statement (avoid for user input - SQL Injection risk)
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM users WHERE id = " + userId);

// PreparedStatement (recommended)
PreparedStatement pstmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
pstmt.setInt(1, userId);
ResultSet rs = pstmt.executeQuery();
```

### Execution Methods

| Method            | Usage                                                         |
|-------------------|---------------------------------------------------------------|
| `executeQuery()`  | For `SELECT` queries - returns a `ResultSet`                  |
| `executeUpdate()` | For `INSERT`, `UPDATE`, `DELETE` - returns affected row count |
| `execute()`       | For dynamic queries that may return multiple results          |

```java
// SELECT
ResultSet rs = stmt.executeQuery("SELECT * FROM employees");
while (rs.next()) {
    System.out.println(rs.getString("name"));
}

// INSERT
int rows = stmt.executeUpdate("INSERT INTO employees (name, salary) VALUES ('John', 50000)");
System.out.println("Rows affected: " + rows);
```

---

## 5. Processing ResultSet

- `next()` - moves the cursor to the next row.
- `getString()`, `getInt()`, `getFloat()` - retrieve column values by name or index.

```java
while (rs.next()) {
    int id = rs.getInt("id");
    String name = rs.getString("name");
    System.out.println(id + " - " + name);
}
```

---

## Calling Stored Procedures

### Choosing the Right Interface

| Scenario                       | Interface           |
|--------------------------------|---------------------|
| No parameters                  | `Statement`         |
| Input parameters only          | `PreparedStatement` |
| Input and/or output parameters | `CallableStatement` |

```java
public void runStoredProcedure(Connection conn) throws SQLException {
    String procedure = "{ call procedureExample(?, ?, ?) }";

    CallableStatement cs = conn.prepareCall(procedure);

    // Set input parameters
    cs.setString(1, "abcd");
    cs.setBoolean(2, true);

    // Register output parameter
    cs.registerOutParameter(3, java.sql.Types.INTEGER);

    // Execute
    cs.execute();

    // Retrieve output
    int result = cs.getInt(3);
    System.out.println("Procedure result: " + result);

    cs.close();
}
```

---

## 6. Closing the Connection

**Manual close:**

```java
conn.close();
```

**Try-with-resources (recommended - Java 7+):**

```java
try (Connection conn = DriverManager.getConnection(url, user, password)) {
    // Database operations
} catch (SQLException e) {
    e.printStackTrace();
}
```

> Always close `ResultSet`, `Statement`, and `PreparedStatement` before closing the `Connection`.

---

## Transaction Propagation

Transaction propagation defines how transactional boundaries are managed when a transactional method calls another - determining whether the inner method joins the existing transaction or starts a new one. In Spring, it is configured via `@Transactional(propagation = ...)`.

| Type                   | Behavior                                                                                                                                                            |
|------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `REQUIRED` *(default)* | Joins an existing transaction if present; otherwise creates a new one.                                                                                              |
| `REQUIRES_NEW`         | Suspends the current transaction and always starts a new, independent one. Useful for logging or auditing that must commit even if the main transaction rolls back. |
| `NESTED`               | Executes within a nested transaction (using savepoints) if an existing transaction exists, allowing partial rollback.                                               |
| `SUPPORTS`             | Runs within a transaction if one exists; otherwise runs non-transactionally.                                                                                        |
| `MANDATORY`            | Requires an active transaction; throws an exception if none exists.                                                                                                 |
| `NOT_SUPPORTED`        | Always runs non-transactionally, suspending any existing transaction.                                                                                               |
| `NEVER`                | Throws an exception if a transaction already exists.                                                                                                                |

### Important Behaviors

- **Logical vs. Physical Transactions:** `REQUIRED` can map multiple nested logical transactions into one physical transaction.
- **`UnexpectedRollbackException`:** Occurs when an inner transaction (e.g., `REQUIRED`) marks the transaction as "rollback-only," causing the outer transaction to fail on commit.
- **Suspension:** With `REQUIRES_NEW`, the parent transaction is paused until the new inner transaction completes.