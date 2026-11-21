# Database (WIA2001)

> How to store, organise, query, and protect data reliably. (Based on Coronel & Morris, *Database Systems*, using the "Tiny College" examples.)

**Topics:** [Why Databases?](#1-why-databases) · [Database Systems vs File Systems](#2-database-systems-vs-file-systems) · [Data Models](#3-data-models) · [Relational Model](#4-the-relational-model) · [Relational Algebra](#5-relational-algebra) · [ER Modelling](#6-entity-relationship-er-modelling) · [Normalization](#7-normalization) · [Database Design](#8-database-design-process) · [SQL Basics](#9-sql-basics) · [Advanced SQL](#10-advanced-sql) · [Transactions & Concurrency](#11-transactions--concurrency-control) · [Coupling & Cohesion](#12-coupling--cohesion)

---

## 1. Why Databases?

- **Efficient storage** and fast retrieval of large amounts of data
- **Data integrity and consistency:** rules keep data correct
- **Data sharing:** many users and apps at once
- **Security:** control who sees what
- Powers **data-driven applications**: e-commerce (products, orders), banking, hospitals, social media, university systems

> 📚 **Analogy:** a database is a well-run library with a catalogue system, librarians enforcing the rules, and many readers at once. A pile of spreadsheets on different laptops is a messy bookshelf in everyone's bedroom.

---

## 2. Database Systems vs File Systems

**Data** = raw facts ("85"). **Information** = processed data with meaning ("Ali scored 85 in DB, top of the class").

- **Database:** a shared, integrated collection of related data plus **metadata** (data about data).
- **DBMS** (Database Management System): the software that manages the database and sits between users/apps and the data (MySQL, Oracle, PostgreSQL, SQL Server).

### Problems with old file systems
Each department kept its own files (CUSTOMER.txt in sales, another copy in billing…):
| Problem | Meaning |
|---|---|
| **Data redundancy** | the same data stored in many places |
| **Data inconsistency** | copies disagree (an address updated in one file, not another) |
| **Structural & data dependence** | change a file's format → must rewrite every program using it |
| Lengthy development, hard ad-hoc queries | every report needs a new program |
| Poor security, difficult sharing | |
| **Data anomalies** | see below |

**Anomalies** (from redundancy):
- **Update anomaly:** must change the same fact in many rows.
- **Insertion anomaly:** can't add data without unrelated data (can't add a new course until a student enrols).
- **Deletion anomaly:** deleting one thing accidentally deletes another (delete the last student → lose the course info).

### DBMS functions
Data dictionary management, storage management, transformation and presentation, **security**, multi-user access control, backup and recovery, data integrity, query language (SQL), communication interfaces.

### Database system environment
Hardware, software (DBMS, OS, apps), **people** (DBA, designers, analysts, programmers, end users), procedures, and data.

### Types of databases
- By users: **single-user** (desktop), **multiuser** (workgroup, enterprise).
- By location: centralised vs **distributed**.
- By use: **operational/OLTP** (day-to-day transactions) vs **analytical** (data warehouses, OLAP, business intelligence).
- By structure: structured (tables), semi-structured (XML/JSON), unstructured.

---

## 3. Data Models

A **data model** is a simple representation (usually graphical) of real-world data structures. It's a communication tool between designers, programmers, and users.

**Building blocks:**
| Block | Meaning | Example |
|---|---|---|
| **Entity** | a thing we store data about | STUDENT, COURSE |
| **Attribute** | a characteristic of an entity | name, matric_no |
| **Relationship** | an association between entities | a student *enrols in* a course |
| **Constraint** | a rule on the data | GPA between 0.00 and 4.00 |

**Relationship types:** **1:1**, **1:M**, **M:N**.

**Business rules** → data model: nouns become entities, verbs become relationships.
*"A customer may place many orders; each order is placed by exactly one customer."* → CUSTOMER 1:M ORDER.

### Evolution of data models
| Model | Idea | Notes |
|---|---|---|
| **Hierarchical** | tree (parent–child) | 1960s IBM IMS; only 1:M |
| **Network** | graph; records with multiple parents | M:N possible but complex |
| **Relational** | **tables** (relations) | E.F. Codd, 1970; the dominant model; SQL |
| **Entity Relationship** | graphical ER diagrams | Peter Chen, 1976; for design |
| **Object-oriented** | objects with attributes **and** methods, classes, inheritance | |
| Object/relational | relational + OO features | |
| **NoSQL** | key-value, document, column-family, graph | **Big Data**: volume, velocity, variety; scales horizontally |

**Degrees of abstraction (ANSI/SPARC):** external (users' views) → conceptual (whole-organisation ER model) → internal (DBMS-specific schema) → physical (files and disks).

---

## 4. The Relational Model

Data is stored in **relations** (tables).

> 📊 **Analogy:** a set of spreadsheets that are carefully linked by ID columns.

### Characteristics of a relational table
- 2D structure of **rows (tuples/records)** and **columns (attributes/fields)**
- Each row/column intersection holds **one single value** (atomic)
- Each column has a **domain** (allowed values) and a unique name
- Row and column order don't matter
- Each row is **uniquely identified** by a primary key

### Keys
**Determination / functional dependency:** A → B means "knowing A determines B" (matric_no → name).

| Key | Meaning | Example |
|---|---|---|
| **Superkey** | any attribute set that uniquely identifies a row | {matric_no}, {matric_no, name} |
| **Candidate key** | a *minimal* superkey (no extra attributes) | {matric_no}, {IC_no} |
| **Primary key (PK)** | the candidate key chosen; **unique and not null** | matric_no |
| **Foreign key (FK)** | an attribute that refers to a PK in another table | ENROLMENT.matric_no → STUDENT.matric_no |
| **Composite key** | a key made of more than one attribute | (matric_no, course_code) |
| **Secondary key** | used for retrieval, not necessarily unique | last_name |

### Integrity rules
- **Entity integrity:** the PK is unique and **never null**.
- **Referential integrity:** an FK must match an existing PK value **or** be null.

> 🔗 **Analogy:** referential integrity means you can't enrol a student who doesn't exist, like you can't book a seat on a flight that doesn't exist.

**Handling nulls:** avoid them where possible; use flags or default values.

**Data dictionary / system catalog:** metadata about all tables, attributes, types, and constraints.

**Index:** a sorted structure (key + pointer) that speeds up searches, like a book's index.

### Implementing relationships
- **1:1:** put the FK on either side (usually the optional side).
- **1:M:** put the FK on the **"many"** side (ORDER has customer_id).
- **M:N:** can't be implemented directly → create a **bridge (composite/associative) entity**:
  ```
  STUDENT ─< ENROLMENT >─ COURSE
  ENROLMENT(matric_no FK, course_code FK, grade)   PK = (matric_no, course_code)
  ```

---

## 5. Relational Algebra

The theoretical operations behind SQL. Each takes relations and returns a new relation (**closure**).

| Operator | Symbol | What it does | SQL equivalent |
|---|---|---|---|
| **SELECT** (restrict) | σ | picks **rows** matching a condition | `WHERE` |
| **PROJECT** | π | picks **columns** | `SELECT col1, col2` |
| **UNION** | ∪ | rows in either table (union-compatible) | `UNION` |
| **INTERSECT** | ∩ | rows in both | `INTERSECT` |
| **DIFFERENCE** | − | rows in A but not B | `EXCEPT` / `MINUS` |
| **PRODUCT** | × | every row of A with every row of B | `CROSS JOIN` |
| **JOIN** | ⋈ | combine related rows | `JOIN` |
| **DIVIDE** | ÷ | rows of A associated with **all** rows of B | (complex) |

> ✂️ **Analogy:** SELECT cuts the table horizontally (keeps some rows); PROJECT cuts it vertically (keeps some columns).

**Union-compatible:** same number of columns with compatible domains.

### Types of joins
| Join | Result |
|---|---|
| **Natural join** | joins on columns with the same name; keeps one copy |
| **Equijoin** | join on equality of specified columns |
| Theta join | join on any comparison (`<`, `>`, …) |
| **Inner join** | only matching rows |
| **Left outer join** | all rows from the left + matches (nulls when there's no match) |
| **Right outer join** | all rows from the right + matches |
| Full outer join | all rows from both |

*Example:* "List all customers and their orders, including customers with no orders" → **LEFT OUTER JOIN**.

---

## 6. Entity Relationship (ER) Modelling

**ERD** = the diagram of entities, attributes, and relationships. Common notations: **Chen** and **Crow's Foot** (used in the course).

### Attributes
| Type | Meaning | Example |
|---|---|---|
| **Required / optional** | must have a value / may be empty | name / middle_name |
| **Simple / composite** | can't be split / can be split | gender / address (street, city, postcode) |
| **Single-valued / multivalued** | one value / many values | IC_no / phone_numbers |
| **Derived** | calculated from others | age (from date_of_birth) |
| Identifier | the PK | matric_no |

**Multivalued attributes** in a relational DB: create a new entity (STUDENT_PHONE) or several columns.
**Derived attributes:** store them (faster reads, risk of going stale) or calculate them on demand (always current, more processing).

### Relationships
- **Connectivity:** 1:1, 1:M, M:N.
- **Cardinality:** min and max occurrences, e.g. (1,4): "a professor teaches at least 1 and at most 4 classes".
- **Participation:**
  - **Optional:** an entity can exist without the relationship (a COURSE may have no CLASS yet). Shown with a **circle** O.
  - **Mandatory:** must participate. Shown with a **bar** |.
- **Degree:** **unary/recursive** (an EMPLOYEE *manages* an EMPLOYEE), **binary** (most common), **ternary** (three entities).

### Crow's Foot symbols
```
──O<   zero or many          ──|<   one or many
──O|   zero or one           ──||   exactly one
```

### Strong vs weak
- **Existence-dependent:** the entity can't exist without another (DEPENDENT needs EMPLOYEE).
- **Weak (non-identifying) relationship:** the child's PK does **not** include the parent's PK (dashed line).
- **Strong (identifying) relationship:** the child's PK **includes** the parent's PK (solid line).
- **Weak entity:** existence-dependent **and** its PK is partly derived from the parent: `DEPENDENT(emp_no, dep_no, name)`.

> 🏠 **Analogy:** a hotel room (weak entity) is identified as "Room 205 of Hotel Hilton". "Room 205" alone isn't unique; it needs the hotel's identity.

### Tiny College example
```
SCHOOL 1──< DEPARTMENT 1──< PROFESSOR
DEPARTMENT 1──< COURSE 1──< CLASS >──1 PROFESSOR (teaches)
STUDENT 1──< ENROLL >──1 CLASS        (ENROLL is the bridge entity for the M:N)
CLASS >──1 ROOM >──1 BUILDING
```

---

## 7. Normalization

**Normalization** = organising tables to **reduce redundancy and anomalies**, by applying a series of **normal forms**.

> 🧹 **Analogy:** tidying a messy room. Put each type of thing in its own drawer (table), and label each drawer so you can find things (keys).

### Functional dependencies
- **Full FD:** the attribute depends on the **whole** composite key.
- **Partial dependency:** depends on **part** of a composite key. `(student_id, course_id) → student_name`, but student_id alone determines the name.
- **Transitive dependency:** a non-key attribute depends on **another non-key attribute**. `emp_id → dept_id → dept_name`.

### The normal forms
| Form | Rule | Fix |
|---|---|---|
| **1NF** | no repeating groups; all values atomic; a PK is identified | split repeating groups into rows; identify the PK and dependencies |
| **2NF** | 1NF + **no partial dependencies** | move partially dependent attributes to a new table with that part of the key |
| **3NF** | 2NF + **no transitive dependencies** | move transitively dependent attributes to a new table |
| BCNF | every determinant is a candidate key | (stricter 3NF) |
| 4NF | no multivalued dependencies | |

💡 *Memory trick:* every non-key attribute must depend on "**the key** (1NF), **the whole key** (2NF), and **nothing but the key** (3NF), so help me Codd."

### Worked example
**Unnormalized:** `PROJECT(proj_num, proj_name, emp_num, emp_name, job_class, chg_hour, hours)` with repeating employees per project.

**1NF:** one row per project–employee; PK = `(proj_num, emp_num)`.
Dependencies:
- `proj_num → proj_name` (partial)
- `emp_num → emp_name, job_class, chg_hour` (partial)
- `job_class → chg_hour` (transitive)
- `(proj_num, emp_num) → hours` (full)

**2NF:**
```
PROJECT(proj_num, proj_name)
EMPLOYEE(emp_num, emp_name, job_class, chg_hour)
ASSIGNMENT(proj_num, emp_num, hours)
```
**3NF** (remove `job_class → chg_hour`):
```
PROJECT(proj_num, proj_name)
EMPLOYEE(emp_num, emp_name, job_class)
JOB(job_class, chg_hour)
ASSIGNMENT(proj_num, emp_num, hours)
```

**Surrogate key:** an artificial PK (auto-number) used when the natural key is too long, composite, or might change.

**Denormalization:** sometimes deliberately reintroducing redundancy for **performance** (fewer joins), e.g. in reporting/data warehouses. It's a trade-off.

---

## 8. Database Design Process

### Information system & SDLC
A database is part of an **information system** (people, hardware, software, procedures, data). It's built using the **Systems Development Life Cycle** (planning → analysis → design → implementation → maintenance); see [SAD](SAD-systems-analysis-and-design.md).

### Database Life Cycle (DBLC)
1. **Database initial study:** analyse the company situation, define problems and constraints, objectives, scope, and boundaries.
2. **Database design** (the most important stage):
   - **Conceptual design:** an ER model, independent of any DBMS. Data analysis and requirements → ER modelling and normalization → data model verification.
   - **DBMS selection.**
   - **Logical design:** map the ER model to tables, keys, and constraints for the chosen model (relational).
   - **Physical design:** storage, indexes, access methods, performance.
3. **Implementation and loading:** create the database, load data, set up security (physical, passwords, access rights, audit trails, encryption).
4. **Testing and evaluation:** test integrity, performance, backup and recovery.
5. **Operation.**
6. **Maintenance and evolution.**

**Backups:** full, **differential** (changes since the last full backup), incremental/transaction log (changes since the last backup of any kind).
**Sources of failure:** software, hardware, programming exceptions, transactions, external (fire, flood).

**Design strategies:** **top-down** (start with entities, then attributes) vs **bottom-up** (start with attributes, then group them). **Centralised** vs **decentralised** design (split into modules, then aggregate).

> 🏗️ **Analogy:** conceptual design = an architect's sketch of the house, logical = detailed floor plans, physical = the construction specs (materials, plumbing).

---

## 9. SQL Basics

**SQL** (Structured Query Language) has two main parts:
- **DDL** (Data Definition Language): create and change structure (`CREATE`, `ALTER`, `DROP`)
- **DML** (Data Manipulation Language): work with the data (`SELECT`, `INSERT`, `UPDATE`, `DELETE`, `COMMIT`, `ROLLBACK`)

### Creating tables
```sql
CREATE TABLE VENDOR (
    V_CODE      INTEGER      PRIMARY KEY,
    V_NAME      VARCHAR(35)  NOT NULL,
    V_STATE     CHAR(2),
    V_ORDER     CHAR(1)      DEFAULT 'N' CHECK (V_ORDER IN ('Y','N'))
);

CREATE TABLE PRODUCT (
    P_CODE      VARCHAR(10)  PRIMARY KEY,
    P_DESCRIPT  VARCHAR(35)  NOT NULL,
    P_INDATE    DATE         NOT NULL,
    P_QOH       SMALLINT     NOT NULL,       -- quantity on hand
    P_MIN       SMALLINT     NOT NULL,
    P_PRICE     NUMERIC(8,2) NOT NULL,
    P_DISCOUNT  NUMERIC(5,2) DEFAULT 0,
    V_CODE      INTEGER,
    FOREIGN KEY (V_CODE) REFERENCES VENDOR (V_CODE) ON UPDATE CASCADE
);

CREATE INDEX P_INDATEX ON PRODUCT (P_INDATE);
```
**Common data types:** `INTEGER`, `SMALLINT`, `NUMERIC(p,s)`/`DECIMAL`, `CHAR(n)` (fixed length), `VARCHAR(n)` (variable length), `DATE`.
**Constraints:** `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL`, `UNIQUE`, `DEFAULT`, `CHECK`.

### Inserting, updating, deleting
```sql
INSERT INTO VENDOR VALUES (21225, 'Bryson Inc.', 'TN', 'Y');
INSERT INTO PRODUCT (P_CODE, P_DESCRIPT, P_INDATE, P_QOH, P_MIN, P_PRICE)
       VALUES ('BRT-345', 'Titanium drill bit', '2025-10-18', 75, 10, 4.50);

UPDATE PRODUCT SET P_PRICE = P_PRICE * 1.10 WHERE V_CODE = 21225;
DELETE FROM PRODUCT WHERE P_QOH = 0;

COMMIT;     -- make the changes permanent
ROLLBACK;   -- undo changes since the last COMMIT
```

### Querying
```sql
SELECT P_CODE, P_DESCRIPT, P_PRICE
FROM   PRODUCT
WHERE  P_PRICE > 10 AND V_CODE = 21344
ORDER BY P_PRICE DESC;
```
| Operator | Example |
|---|---|
| Comparison | `=  <  >  <=  >=  <>` |
| Arithmetic | `SELECT P_QOH * P_PRICE AS TOTAL_VALUE` |
| Logical | `AND`, `OR`, `NOT` |
| `BETWEEN` | `WHERE P_PRICE BETWEEN 50 AND 100` |
| `IN` | `WHERE V_CODE IN (21344, 24288)` |
| `LIKE` | `WHERE V_NAME LIKE 'Sm%'` (`%` = any characters, `_` = exactly one) |
| `IS NULL` | `WHERE V_CODE IS NULL` |
| `EXISTS` | `WHERE EXISTS (subquery)` |
| `DISTINCT` | `SELECT DISTINCT V_CODE FROM PRODUCT` |

### Aggregates and grouping
```sql
SELECT V_CODE, COUNT(*) AS NUM_PRODUCTS, AVG(P_PRICE) AS AVG_PRICE
FROM   PRODUCT
GROUP BY V_CODE
HAVING AVG(P_PRICE) > 10;       -- HAVING filters GROUPS; WHERE filters ROWS
```
Aggregates: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.

> 🧮 **WHERE vs HAVING:** WHERE picks which students enter the exam hall; HAVING picks which *classes* (groups) get an award based on their average.

**Logical order of execution:** `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY`.

### Altering structure
```sql
ALTER TABLE PRODUCT ADD P_SALECODE CHAR(1);
ALTER TABLE PRODUCT MODIFY (P_PRICE DECIMAL(9,2));     -- Oracle syntax
ALTER TABLE PRODUCT DROP COLUMN P_SALECODE;
DROP TABLE PART;
CREATE TABLE PART AS SELECT P_CODE, P_DESCRIPT FROM PRODUCT;   -- copy
```

---

## 10. Advanced SQL

### Joins
```sql
-- Inner join (old style and modern style)
SELECT P_DESCRIPT, V_NAME FROM PRODUCT, VENDOR WHERE PRODUCT.V_CODE = VENDOR.V_CODE;
SELECT P_DESCRIPT, V_NAME FROM PRODUCT JOIN VENDOR ON PRODUCT.V_CODE = VENDOR.V_CODE;
SELECT P_DESCRIPT, V_NAME FROM PRODUCT NATURAL JOIN VENDOR;
SELECT P_DESCRIPT, V_NAME FROM PRODUCT JOIN VENDOR USING (V_CODE);

-- Outer join: all vendors, even those with no products
SELECT V_NAME, P_DESCRIPT FROM VENDOR LEFT JOIN PRODUCT ON VENDOR.V_CODE = PRODUCT.V_CODE;
```

### Subqueries
```sql
-- WHERE subquery: products priced above average
SELECT P_CODE, P_PRICE FROM PRODUCT WHERE P_PRICE > (SELECT AVG(P_PRICE) FROM PRODUCT);

-- IN subquery: customers who bought hammers
SELECT DISTINCT CUS_CODE FROM INVOICE JOIN LINE USING (INV_NUMBER)
WHERE P_CODE IN (SELECT P_CODE FROM PRODUCT WHERE P_DESCRIPT LIKE '%hammer%');

-- HAVING subquery
SELECT P_CODE, SUM(LINE_UNITS) FROM LINE GROUP BY P_CODE
HAVING SUM(LINE_UNITS) > (SELECT AVG(LINE_UNITS) FROM LINE);

-- Multirow operators: ALL / ANY
SELECT P_CODE FROM PRODUCT WHERE P_PRICE > ALL (SELECT P_PRICE FROM PRODUCT WHERE V_CODE = 21344);

-- Attribute list subquery
SELECT P_CODE, P_PRICE, P_PRICE - (SELECT AVG(P_PRICE) FROM PRODUCT) AS DIFF FROM PRODUCT;

-- Correlated subquery: runs once per outer row
SELECT INV_NUMBER, P_CODE, LINE_UNITS FROM LINE LS
WHERE LS.LINE_UNITS > (SELECT AVG(LINE_UNITS) FROM LINE LA WHERE LA.P_CODE = LS.P_CODE);
```
> 🔁 **Correlated subquery analogy:** for each student, compare their mark with the average **of their own class**, so the inner query reruns for each student.

### Functions
Date (`SYSDATE`, `TO_CHAR`, `ADD_MONTHS`), numeric (`ROUND`, `ABS`, `CEIL`, `FLOOR`), string (`UPPER`, `LOWER`, `SUBSTR`, `LENGTH`, `||` concatenation), conversion (`TO_DATE`, `CAST`).

### Set operators
`UNION` (removes duplicates), `UNION ALL` (keeps them), `INTERSECT`, `EXCEPT`/`MINUS`. Queries must be union-compatible.

### Views
A **view** is a saved query that acts like a virtual table.
```sql
CREATE VIEW PRICEGT50 AS SELECT P_DESCRIPT, P_QOH, P_PRICE FROM PRODUCT WHERE P_PRICE > 50;
SELECT * FROM PRICEGT50;
```
Uses: security (hide columns), simplify complex queries. **Updatable views** have restrictions (no GROUP BY, aggregates, DISTINCT…).

### Sequences (Oracle)
```sql
CREATE SEQUENCE CUS_CODE_SEQ START WITH 20010 NOCACHE;
INSERT INTO CUSTOMER VALUES (CUS_CODE_SEQ.NEXTVAL, 'Lim', ...);
```

### Procedural SQL (PL/SQL)
```sql
-- Trigger: automatically runs on INSERT/UPDATE/DELETE
CREATE OR REPLACE TRIGGER TRG_PRODUCT_REORDER
AFTER INSERT OR UPDATE OF P_QOH, P_MIN ON PRODUCT
BEGIN
    UPDATE PRODUCT SET P_REORDER = 1 WHERE P_QOH <= P_MIN;
END;

-- Stored procedure: named block you call
CREATE OR REPLACE PROCEDURE PRC_PROD_DISCOUNT AS
BEGIN
    UPDATE PRODUCT SET P_DISCOUNT = P_DISCOUNT + .05 WHERE P_QOH >= P_MIN * 2;
    DBMS_OUTPUT.PUT_LINE('* * Update finished * *');
END;
EXEC PRC_PROD_DISCOUNT;

-- Cursor: process query results row by row
DECLARE
    CURSOR PROD_CURSOR IS SELECT P_CODE, P_QOH FROM PRODUCT;
    W_P_CODE PRODUCT.P_CODE%TYPE;  W_P_QOH PRODUCT.P_QOH%TYPE;
BEGIN
    OPEN PROD_CURSOR;
    LOOP
        FETCH PROD_CURSOR INTO W_P_CODE, W_P_QOH;
        EXIT WHEN PROD_CURSOR%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(W_P_CODE || ' ' || W_P_QOH);
    END LOOP;
    CLOSE PROD_CURSOR;
END;
```
Cursor attributes: `%ROWCOUNT`, `%FOUND`, `%NOTFOUND`, `%ISOPEN`.
**Stored functions** return a value and can be used inside SQL.
**Embedded SQL:** SQL inside a host language (Java, C, COBOL) with status variables like `SQLCODE`.

> ⚙️ **Trigger analogy:** a motion-sensor light. You don't switch it on; an event (someone walks past, a row is updated) fires it automatically.

---

## 11. Transactions & Concurrency Control

A **transaction** is a logical unit of work that must be **completed entirely or not at all**.

*Example:* transfer RM100 from A to B = `UPDATE A -100` + `UPDATE B +100`. If the system crashes between the two, money vanishes, so both must succeed or both must be undone.

> ✈️ **Analogy:** booking a flight = reserve a seat + charge the card + issue a ticket. You don't want to be charged without a seat.

### ACID properties
| Property | Meaning |
|---|---|
| **Atomicity** | all or nothing |
| **Consistency** | takes the DB from one valid state to another (rules preserved) |
| **Isolation** | concurrent transactions don't see each other's partial work |
| **Durability** | once committed, changes survive crashes |

(Plus **serializability**: concurrent execution gives the same result as some serial order.)

```sql
BEGIN TRANSACTION;          -- (implicit in Oracle)
UPDATE ACCOUNT SET BAL = BAL - 100 WHERE ACC = 'A';
UPDATE ACCOUNT SET BAL = BAL + 100 WHERE ACC = 'B';
COMMIT;                     -- or ROLLBACK on error
```

**Transaction log:** records every change (before and after values) so the DBMS can **roll back** incomplete transactions or **roll forward** committed ones after a crash.

### Concurrency problems
| Problem | What happens | Example |
|---|---|---|
| **Lost update** | two transactions update the same data; one overwrites the other | T1 and T2 both read stock = 35; T1 adds 100 → 135; T2 subtracts 30 → 5; T1's update is lost |
| **Uncommitted data (dirty read)** | T2 reads T1's change, then T1 rolls back | T2 used a value that never officially existed |
| **Inconsistent retrieval** | T1 calculates a summary while T2 updates some of the rows | totals are wrong |

The **scheduler** orders operations so they're serializable.

### Locking
**Lock granularity:** database → table → page → **row** → field. Finer = more concurrency, more overhead.

| Lock type | Meaning |
|---|---|
| **Binary** | locked/unlocked |
| **Shared (S)** | for reading; many transactions can share it |
| **Exclusive (X)** | for writing; no one else can read or write |

> 📖 **Analogy:** a shared lock = many people can read the same library book in the reading room. An exclusive lock = one person takes it home to write notes in it.

**Two-Phase Locking (2PL):** guarantees serializability.
1. **Growing phase:** acquire locks, release none.
2. **Shrinking phase:** release locks, acquire none.

**Deadlock:** T1 waits for T2's lock while T2 waits for T1's. Handled by **prevention** (abort if a deadlock is possible), **detection** (periodically check, abort a victim), or **avoidance** (get all locks in advance).

### Time stamping
Each transaction gets a unique timestamp; conflicts are resolved by age.
| Scheme | Older T requests a lock held by younger | Younger T requests a lock held by older |
|---|---|---|
| **Wait/Die** | older **waits** | younger **dies** (rolled back, restarted with the same timestamp) |
| **Wound/Wait** | older **wounds** (preempts) the younger | younger **waits** |

### Optimistic concurrency
Assume conflicts are rare. Phases: **read** (work on private copies) → **validation** (check for conflicts) → **write** (apply if valid, otherwise restart).

### Isolation levels (ANSI)
| Level | Dirty read | Non-repeatable read | Phantom read |
|---|---|---|---|
| Read Uncommitted | possible | possible | possible |
| Read Committed | ❌ | possible | possible |
| Repeatable Read | ❌ | ❌ | possible |
| Serializable | ❌ | ❌ | ❌ |

### Database recovery
- **Write-ahead log protocol:** write the log **before** the data.
- **Redundant transaction logs**, buffers, **checkpoints**.
- **Deferred write:** the DB is updated only after commit → recovery needs only redo.
- **Write-through:** the DB is updated immediately → recovery needs undo (for uncommitted) and redo.

---

## 12. Coupling & Cohesion

Design principles for modules (also apply to programs and system design):
- **Cohesion** = how closely related the tasks *inside* one module are. **Want HIGH cohesion** (each module does one well-defined job).
- **Coupling** = how much modules *depend on each other*. **Want LOW coupling** (change one without breaking others).

> 🍳 **Analogy:** a well-organised kitchen. The baking station only bakes (high cohesion) and doesn't need to borrow tools from the grill station (low coupling).

**Cohesion types (best → worst):** functional, sequential, communicational, procedural, temporal, logical, coincidental.
**Coupling types (best → worst):** data, stamp, control, external, common, content.

---

### ✅ Quick self-check
1. What's the difference between a candidate key and a primary key? *(the PK is the candidate key chosen)*
2. How do you implement an M:N relationship? *(a bridge/associative table)*
3. A table has a transitive dependency. Which normal form does it violate? *(3NF)*
4. WHERE vs HAVING? *(rows vs groups)*
5. What does the "A" in ACID mean? *(atomicity: all or nothing)*
6. What two phases does 2PL have? *(growing, shrinking)*
