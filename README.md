

```sql
CREATE TABLE parent(
    id bigint auto_increment
    , CONSTRAINT parent_pk PRIMARY KEY (id)
);
```
Query OK, 0 rows affected (0.73 sec)


```sql
CREATE TABLE child(
    id bigint auto_increment
    , parent_id bigint
    , name varchar(254)
    , is_deleted varchar(1)
    , CONSTRAINT child_pk PRIMARY KEY (id)
);
```
Query OK, 0 rows affected (0.63 sec)

```sql
ALTER TABLE child ADD FULLTEXT INDEX ngram_name (name) WITH PARSER ngram;
```
Query OK, 0 rows affected, 1 warning (7.38 sec)
Records: 0  Duplicates: 0  Warnings: 1

```sql
INSERT INTO parent(id) VALUES
(1)
, (2)
, (3);
```
Query OK, 3 rows affected (0.15 sec)
Records: 3  Duplicates: 0  Warnings: 0

```sql
INSERT INTO child(id, parent_id, name, is_deleted) VALUES
(1, 1, 'john', '0')
, (2, 1, 'mike', '0')
, (3, 1, 'tom', '0')
, (4, 2, 'john', '1')
, (5, 2, 'mike', '1')
, (6, 2, 'tom', '1')
, (7, 2, 'john', '0')
, (8, 2, 'mike', '0')
, (9, 2, 'tom', '0')
, (10, 3, 'john', '0')
, (11, 3, 'mike', '0')
, (12, 3, 'tom', '0');
```
Query OK, 12 rows affected (0.24 sec)
Records: 12  Duplicates: 0  Warnings: 0

```sql
SELECT * FROM parent;
```
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
+----+
3 rows in set (0.00 sec)



```sql
SELECT * FROM child;
```
+----+-----------+------+------------+
| id | parent_id | name | is_deleted |
+----+-----------+------+------------+
|  1 |         1 | john | 0          |
|  2 |         1 | mike | 0          |
|  3 |         1 | tom  | 0          |
|  4 |         2 | john | 1          |
|  5 |         2 | mike | 1          |
|  6 |         2 | tom  | 1          |
|  7 |         2 | john | 0          |
|  8 |         2 | mike | 0          |
|  9 |         2 | tom  | 0          |
| 10 |         3 | john | 0          |
| 11 |         3 | mike | 0          |
| 12 |         3 | tom  | 0          |
+----+-----------+------+------------+
12 rows in set (0.00 sec)


```sql
SELECT
    *
FROM
    parent
    INNER JOIN child
        ON parent.id = child.parent_id
        AND child.is_deleted = '0'
WHERE
    (
        EXISTS (
            SELECT
                *
            FROM
                child child_2
            WHERE
                child_2.parent_id = parent.id
                AND child_2.is_deleted = '0'
                AND MATCH (child_2.name) against('mike' IN BOOLEAN mode)
        )
    )
;
```
+----+----+-----------+------+------------+
| id | id | parent_id | name | is_deleted |
+----+----+-----------+------+------------+
|  1 |  1 |         1 | john | 0          |
|  1 |  2 |         1 | mike | 0          |
|  1 |  3 |         1 | tom  | 0          |
|  2 |  7 |         2 | john | 0          |
|  2 |  8 |         2 | mike | 0          |
|  2 |  9 |         2 | tom  | 0          |
|  3 | 10 |         3 | john | 0          |
|  3 | 11 |         3 | mike | 0          |
|  3 | 12 |         3 | tom  | 0          |
+----+----+-----------+------+------------+
9 rows in set (0.00 sec)

```sql
SELECT
    *
FROM
    parent
    INNER JOIN child
        ON parent.id = child.parent_id
        AND child.is_deleted = '0'
WHERE
    (
        1 = 2
        OR EXISTS (
            SELECT
                *
            FROM
                child child_2
            WHERE
                child_2.parent_id = parent.id
                AND child_2.is_deleted = '0'
                AND child_2.name like 'mike'
        )
    )
;
```
+----+----+-----------+------+------------+
| id | id | parent_id | name | is_deleted |
+----+----+-----------+------+------------+
|  1 |  1 |         1 | john | 0          |
|  1 |  2 |         1 | mike | 0          |
|  1 |  3 |         1 | tom  | 0          |
|  2 |  7 |         2 | john | 0          |
|  2 |  8 |         2 | mike | 0          |
|  2 |  9 |         2 | tom  | 0          |
|  3 | 10 |         3 | john | 0          |
|  3 | 11 |         3 | mike | 0          |
|  3 | 12 |         3 | tom  | 0          |
+----+----+-----------+------+------------+
9 rows in set (0.00 sec)

```sql

SELECT
    *
FROM
    parent
    INNER JOIN child
        ON parent.id = child.parent_id
        AND child.is_deleted = '0'
WHERE
    (
        1 = 2
        OR EXISTS (
            SELECT
                *
            FROM
                child child_2
            WHERE
                child_2.parent_id = parent.id
                AND child_2.is_deleted = '0'
                AND MATCH (child_2.name) against('mike' IN BOOLEAN mode)
        )
    )
;
```

+----+----+-----------+------+------------+
| id | id | parent_id | name | is_deleted |
+----+----+-----------+------+------------+
|  1 |  1 |         1 | john | 0          |
|  1 |  3 |         1 | tom  | 0          |
|  2 |  7 |         2 | john | 0          |
|  2 |  9 |         2 | tom  | 0          |
|  3 | 10 |         3 | john | 0          |
|  3 | 12 |         3 | tom  | 0          |
+----+----+-----------+------+------------+
6 rows in set (0.01 sec)