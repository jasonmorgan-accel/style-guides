---
title: SQL Style Guide
category: SQL
order: 10
nav: true
---
## Overview
This the style guide for SQL Development.  This is primarily target to SQL Server (TSQL), RedShift/postgres (PSQL) and mysql although most concepts will directy translate to other targets.

## Purpose

Maintaining reproducibility and transparency is a core value of any successful data team, and a SQL style guide can help achieve that goal. Additionally, adhering to the basic rules in this style guide will improve our ability to share, maintain, and extend our reach when working with SQL.

This document is written as a manual for anyone working on SQL, but also as a guide for anyone at the company who would like to write clean and clear code that is meant to be shared.

The individual tips in this guide are based on a composite of knowledge we've gleaned from experience and our roles at previous jobs.

## Principles

* We take a disciplined and practical approach to writing code.
* We regularly check-in code to Github
* We believe consistency in style is very important.
* We demonstrate intent explicitly in code, via clear structure and comments where needed.

## Rules

### General stuff

* No spaces. 1 tab per indent.
* No trailing whitespace.
* Always capitalize SQL keywords (e.g., `SELECT` or `AS`)
* Variable names should be underscore separated:

  __GOOD__:
  `SELECT COUNT(*) AS backers_count`

  __BAD__:
  `SELECT COUNT(*) AS backersCount`

* Comments should go near the top of your query, or at least near the closest `SELECT`
* Try to only comment on things that aren't obvious about the query (e.g., why a particular ID is hardcoded, etc.)
* Don't use single letter variable names be as descriptive as possible given the context:

  __GOOD__:
  `SELECT ksr.backings AS backings_with_creators`

  __BAD__:
  `SELECT ksr.backings AS b`

* Use [Common Table Expressions](http://www.postgresql.org/docs/8.4/static/queries-with.html) (CTEs) early and often, and name them well.
* `HAVING` isn't supported in Redshift, so use CTEs instead. If you don't know what this means, ask a friendly Data Team member.


### Do

* Use consistent and descriptive identifiers and names.
* Make judicious use of white space and indentation to make code easier to read.
* Store [ISO-8601][iso-8601] compliant time and date information
  (`YYYY-MM-DD HH:MM:SS.SSSSS`).
* Try to use only standard SQL functions instead of vendor specific functions for
  reasons of portability.
* Keep code succinct and devoid of redundant SQL—such as unnecessary quoting or
  parentheses or `WHERE` clauses that can otherwise be derived.
* Include comments in SQL code where necessary. Use the C style opening `/*` and
  closing `*/` where possible otherwise preceed comments with `--` and finish
  them with a new line.

```sql
SELECT file_hash  -- stored ssdeep hash
FROM file_system
WHERE file_name = '.vimrc';
```
```sql
/* Updating the file record after writing to the file */
UPDATE file_system
SET 
     file_modified_date = '1980-02-22 13:19:01.00000'
    ,file_size = 209732
WHERE file_name = '.vimrc';
```

### Avoid

* CamelCase—it is difficult to scan quickly.
* Descriptive prefixes or Hungarian notation such as `sp_` or `tbl`.
* Plurals—use the more natural collective term where possible instead. For example
  `staff` instead of `employees` or `people` instead of `individuals`.
* Quoted identifiers—if you must use them then stick to SQL92 double quotes for
  portability (you may need to configure your SQL server to support this depending
  on vendor).
* Object oriented design principles should not be applied to SQL or database
  structures.

## Naming conventions

### General

* Ensure the name is unique and does not exist as a
  [reserved keyword][reserved-keywords].
* Keep the length to a maximum of 30 bytes—in practice this is 30 characters
  unless you are using multi-byte character set.
* Names must begin with a letter and may not end with an underscore.
* Only use letters, numbers and underscores in names.
* Avoid the use of multiple consecutive underscores—these can be hard to read.
* Use underscores where you would naturally include a space in the name (first
  name becomes `first_name`).
* Avoid abbreviations and if you have to use them make sure they are commonly
  understood.

```sql
SELECT first_name
FROM staff;
```

### Tables

* Use a collective name or, less ideally, a plural form. For example (in order of
  preference) `staff` and `employees`.
* Do not prefix with `tbl` or any other such descriptive prefix or Hungarian
  notation.
* Never give a table the same name as one of its columns and vice versa.
* Avoid, where possible, concatenating two table names together to create the name
  of a relationship table. Rather than `cars_mechanics` prefer `servicers`.

### Columns

* Always use the singular name.
* Where possible avoid simply using `id` as the primary identifier for the table.
* Do not add a column with the same name as its table and vice versa.
* Always use lowercase.

### Aliasing or correlations

* Should relate in some way to the object or expression they are aliasing.
* As a rule of thumb the correlation name should be a descriptive label.  When it makes sense using the singular for the 
  the table name can make the SQL read quite logically ie. staff -> mentor and students -> student in the example below.
  When this does not make sence the first letter of each word in the object's name should be used.
* If there is already a correlation with the same name then append a number.
* Always include the `AS` keyword—makes it easier to read as it is explicit.
* For computed data (`SUM()` or `AVG()`) use the name you would give it were it
  a column defined in the schema.
* Always use an alias for tables even if it is the same as the table name.  Alway qualify table and column name.  This 
  aids in automation and SQL template creations as table and column names are easily machine replacable and predicates are
  insulated from changes in the table naming that may occure during deployment
* 
```sql
-- GOOD
SELECT first_name, mentor.first_name as mentor_first_name
FROM staff AS mentor
  INNER JOIN students AS student
    ON student.mentor_id = mentor.staff_num;

-- BAD   
SELECT first_name fn, mentor.first_name as mfn 
FROM staff AS s1
  INNER JOIN students AS s2
    ON s2.mentor_id = s1.staff_num;
```

```sql
SELECT SUM(s.monitor_tally) AS monitor_total
FROM staff AS staff_member;
```

The following example if from an installer application that is 
```sql
WITH SEQ  ([CODE], [VALUE]) AS (
  SELECT 
          CAST([N2DMA] as INT),
          CONCAT('PLEASE UPDATE ', CAST("N2DMA" as nvarchar(255)))
          
  FROM    "DBO"."TMP_SYNC_AUDIENCE" AS SYNC WITH(NOLOCK)
  
  WHERE   NOT EXISTS(SELECT 1 FROM [DBO].[OPTION_N2DMA] AS OPTION WHERE OPTION."CODE" = SYNC."N2DMA"   )
          AND COALESCE(SYNC.[N2DMA]   ,'') <>''
          
  GROUP BY SYNC.[N2DMA]   
)
INSERT  "DBO"."OPTION_N2DMA"( -- "N2DMA" <-- this 
  CODE, 
  EN
)
SELECT
      SEQ.[CODE]
     ,SEQ.[VALUE]
FROM  SEQ; 
```

### Stored procedures

* The name must contain a verb.
* Do not prefix with `sp_` or any other such descriptive prefix or Hungarian
  notation.

### Uniform suffixes

The following suffixes have a universal meaning ensuring the columns can be read
and understood easily from SQL code. Use the correct suffix where appropriate.

* `_id`—a unique identifier such as a column that is a primary key.
* `_status`—flag value or some other status of any type such as
  `publication_status`.
* `_total`—the total or sum of a collection of values.
* `_num`—denotes the field contains any kind of number.
* `_name`—signifies a name such as `first_name`.
* `_seq`—contains a contiguous sequence of values.
* `_date`—denotes a column that contains the date of something.
* `_tally`—a count.
* `_size`—the size of something such as a file size or clothing.
* `_addr`—an address for the record could be physical or intangible such as
  `ip_addr`.


## `SELECT`

Align all columns to the first column on their own line:

```sql
SELECT
  projects.name,
  users.email,
  projects.country,
  COUNT(backings.id) AS backings_count
FROM ...
```

`SELECT` goes on its own line:

```sql
SELECT
  name,
  ...
```

Always rename aggregates and function-wrapped columns:

```sql
SELECT
  name,
  SUM(amount) AS sum_amount
FROM ...
```

Always rename all columns when selecting with table aliases:

```sql
SELECT
  projects.name AS project_name,
  COUNT(backings.id) AS backings_count
FROM ksr.backings AS backings
INNER JOIN ksr.projects AS projects ON ...
```

Always use `AS` to rename columns:

__GOOD__:

```sql
SELECT
  projects.name AS project_name,
  COUNT(backings.id) AS backings_count
...
```

__BAD__:

```sql
SELECT
  projects.name project_name,
  COUNT(backings.id) backings_count
...
```

Long Window functions should be split across multiple lines: one for the `PARTITION`, `ORDER` and frame clauses, aligned to the `PARTITION` keyword. Partition keys should be one-per-line, aligned to the first, with aligned commas. Order (`ASC`, `DESC`) should always be explicit. All window functions should be aliased.

```sql
SUM(1) OVER (PARTITION BY category_id,
                          year
             ORDER BY pledged DESC
             ROWS UNBOUNDED PRECEDING) AS category_year
```

## `FROM`

Only one table should be in the `FROM`. Never use `FROM`-joins:

__GOOD__:

```sql
SELECT
  projects.name AS project_name,
  COUNT(backings.id) AS backings_count
FROM ksr.projects AS projects
INNER JOIN ksr.backings AS backings ON backings.project_id = projects.id
...
```

__BAD__:

```sql
SELECT
  projects.name AS project_name,
  COUNT(backings.id) AS backings_count
FROM ksr.projects AS projects, ksr.backings AS backings
WHERE
  backings.project_id = projects.id
...
```

## `JOIN`

Explicitly use `INNER JOIN` not just `JOIN`, making multiple lines of `INNER JOIN`s easier to scan:

__GOOD__:

```sql
SELECT
  projects.name AS project_name,
  COUNT(backings.id) AS backings_count
FROM ksr.projects AS projects
INNER JOIN ksr.backings AS backings ON ...
INNER JOIN ...
LEFT JOIN ksr.backer_rewards AS backer_rewards ON ...
LEFT JOIN ...
```

__BAD__:

```sql
SELECT
  projects.name AS project_name,
  COUNT(backings.id) AS backings_count
FROM ksr.projects AS projects
JOIN ksr.backings AS backings ON ...
LEFT JOIN ksr.backer_rewards AS backer_rewards ON ...
LEFT JOIN ...
```

Additional filters in the `INNER JOIN` go on new indented lines:

```sql
SELECT
  projects.name AS project_name,
  COUNT(backings.id) AS backings_count
FROM ksr.projects AS projects
INNER JOIN ksr.backings AS backings ON projects.id = backings.project_id
  AND backings.project_country != 'US'
...
```

The `ON` keyword and condition goes on the `INNER JOIN` line:

```sql
SELECT
  projects.name AS project_name,
  COUNT(backings.id) AS backings_count
FROM ksr.projects AS projects
INNER JOIN ksr.backings AS backings ON projects.id = backings.project_id
...
```

Begin with `INNER JOIN`s and then list `LEFT JOIN`s, order them semantically, and do not intermingle `LEFT JOIN`s with `INNER JOIN`s unless necessary:

__GOOD__:

```sql
INNER JOIN ksr.backings AS backings ON ...
INNER JOIN ksr.users AS users ON ...
INNER JOIN ksr.locations AS locations ON ...
LEFT JOIN ksr.backer_rewards AS backer_rewards ON ...
LEFT JOIN ...
```

__BAD__:

```sql
LEFT JOIN ksr.backer_rewards AS backer_rewards ON backings
INNER JOIN ksr.users AS users ON ...
LEFT JOIN ...
INNER JOIN ksr.locations AS locations ON ...
```

## `WHERE`

Multiple `WHERE` clauses should go on different lines and begin with the SQL operator:

```sql
SELECT
  name,
  goal
FROM ksr.projects AS projects
WHERE
  country = 'US'
  AND deadline >= '2015-01-01'
...
```

## `CASE`

`CASE` statements aren't always easy to format but try to align `WHEN`, `THEN`, and `ELSE` together inside `CASE` and `END`:

```sql
CASE WHEN category = 'Art'
     THEN backer_id
     ELSE NULL
END
```

## Common Table Expressions (CTEs)

[From AWS](http://docs.aws.amazon.com/redshift/latest/dg/r_WITH_clause.html):

>`WITH` clause subqueries are an efficient way of defining tables that can be used throughout the execution of a single query. In all cases, the same results can be achieved by using subqueries in the main body of the `SELECT` statement, but `WITH` clause subqueries may be simpler to write and read.

The body of a CTE must be one indent further than the `WITH` keyword. Open them at the end of a line and close them on a new line:

```sql
WITH backings_per_category AS (
  SELECT
    category_id,
    deadline,
    ...
)
```

Multiple CTEs should be formatted accordingly:

```sql
WITH backings_per_category AS (
  SELECT
    ...
), backers AS (
  SELECT
    ...
), backers_and_creators AS (
  ...
)
SELECT * FROM backers;
```

If possible, `JOIN` CTEs inside subsequent CTEs, not in the main clause:

__GOOD__:

```sql
WITH backings_per_category AS (
  SELECT
    ...
), backers AS (
  SELECT
    backer_id,
    COUNT(backings_per_category.id) AS projects_backed_per_category
  INNER JOIN ksr.users AS users ON users.id = backings_per_category.backer_id
), backers_and_creators AS (
  ...
)
SELECT * FROM backers_and_creators;
```

__BAD__:

```sql
WITH backings_per_category AS (
  SELECT
    ...
), backers AS (
  SELECT
    backer_id,
    COUNT(backings_per_category.id) AS projects_backed_per_category
), backers_and_creators AS (
  ...
)
SELECT * FROM backers_and_creators
INNER JOIN backers ON backers_and_creators ON backers.backer_id = backers_and_creators.backer_id
```

Always use CTEs over inlined subqueries.


## Appendix

### Reserved keyword reference

A list of ANSI SQL (92, 99 and 2003), MySQL 3 to 5.x, PostgreSQL 8.1, MS SQL Server 2000, MS ODBC and Oracle 10.2 reserved keywords.

```sql
A
ABORT
ABS
ABSOLUTE
ACCESS
ACTION
ADA
ADD
ADMIN
AFTER
AGGREGATE
ALIAS
ALL
ALLOCATE
ALSO
ALTER
ALWAYS
ANALYSE
ANALYZE
AND
ANY
ARE
ARRAY
AS
ASC
ASENSITIVE
ASSERTION
ASSIGNMENT
ASYMMETRIC
AT
ATOMIC
ATTRIBUTE
ATTRIBUTES
AUDIT
AUTHORIZATION
AUTO_INCREMENT
AVG
AVG_ROW_LENGTH
BACKUP
BACKWARD
BEFORE
BEGIN
BERNOULLI
BETWEEN
BIGINT
BINARY
BIT
BIT_LENGTH
BITVAR
BLOB
BOOL
BOOLEAN
BOTH
BREADTH
BREAK
BROWSE
BULK
BY
C
CACHE
CALL
CALLED
CARDINALITY
CASCADE
CASCADED
CASE
CAST
CATALOG
CATALOG_NAME
CEIL
CEILING
CHAIN
CHANGE
CHAR
CHAR_LENGTH
CHARACTER
CHARACTER_LENGTH
CHARACTER_SET_CATALOG
CHARACTER_SET_NAME
CHARACTER_SET_SCHEMA
CHARACTERISTICS
CHARACTERS
CHECK
CHECKED
CHECKPOINT
CHECKSUM
CLASS
CLASS_ORIGIN
CLOB
CLOSE
CLUSTER
CLUSTERED
COALESCE
COBOL
COLLATE
COLLATION
COLLATION_CATALOG
COLLATION_NAME
COLLATION_SCHEMA
COLLECT
COLUMN
COLUMN_NAME
COLUMNS
COMMAND_FUNCTION
COMMAND_FUNCTION_CODE
COMMENT
COMMIT
COMMITTED
COMPLETION
COMPRESS
COMPUTE
CONDITION
CONDITION_NUMBER
CONNECT
CONNECTION
CONNECTION_NAME
CONSTRAINT
CONSTRAINT_CATALOG
CONSTRAINT_NAME
CONSTRAINT_SCHEMA
CONSTRAINTS
CONSTRUCTOR
CONTAINS
CONTAINSTABLE
CONTINUE
CONVERSION
CONVERT
COPY
CORR
CORRESPONDING
COUNT
COVAR_POP
COVAR_SAMP
CREATE
CREATEDB
CREATEROLE
CREATEUSER
CROSS
CSV
CUBE
CUME_DIST
CURRENT
CURRENT_DATE
CURRENT_DEFAULT_TRANSFORM_GROUP
CURRENT_PATH
CURRENT_ROLE
CURRENT_TIME
CURRENT_TIMESTAMP
CURRENT_TRANSFORM_GROUP_FOR_TYPE
CURRENT_USER
CURSOR
CURSOR_NAME
CYCLE
DATA
DATABASE
DATABASES
DATE
DATETIME
DATETIME_INTERVAL_CODE
DATETIME_INTERVAL_PRECISION
DAY
DAY_HOUR
DAY_MICROSECOND
DAY_MINUTE
DAY_SECOND
DAYOFMONTH
DAYOFWEEK
DAYOFYEAR
DBCC
DEALLOCATE
DEC
DECIMAL
DECLARE
DEFAULT
DEFAULTS
DEFERRABLE
DEFERRED
DEFINED
DEFINER
DEGREE
DELAY_KEY_WRITE
DELAYED
DELETE
DELIMITER
DELIMITERS
DENSE_RANK
DENY
DEPTH
DEREF
DERIVED
DESC
DESCRIBE
DESCRIPTOR
DESTROY
DESTRUCTOR
DETERMINISTIC
DIAGNOSTICS
DICTIONARY
DISABLE
DISCONNECT
DISK
DISPATCH
DISTINCT
DISTINCTROW
DISTRIBUTED
DIV
DO
DOMAIN
DOUBLE
DROP
DUAL
DUMMY
DUMP
DYNAMIC
DYNAMIC_FUNCTION
DYNAMIC_FUNCTION_CODE
EACH
ELEMENT
ELSE
ELSEIF
ENABLE
ENCLOSED
ENCODING
ENCRYPTED
END
END-EXEC
ENUM
EQUALS
ERRLVL
ESCAPE
ESCAPED
EVERY
EXCEPT
EXCEPTION
EXCLUDE
EXCLUDING
EXCLUSIVE
EXEC
EXECUTE
EXISTING
EXISTS
EXIT
EXP
EXPLAIN
EXTERNAL
EXTRACT
FALSE
FETCH
FIELDS
FILE
FILLFACTOR
FILTER
FINAL
FIRST
FLOAT
FLOAT4
FLOAT8
FLOOR
FLUSH
FOLLOWING
FOR
FORCE
FOREIGN
FORTRAN
FORWARD
FOUND
FREE
FREETEXT
FREETEXTTABLE
FREEZE
FROM
FULL
FULLTEXT
FUNCTION
FUSION
G
GENERAL
GENERATED
GET
GLOBAL
GO
GOTO
GRANT
GRANTED
GRANTS
GREATEST
GROUP
GROUPING
HANDLER
HAVING
HEADER
HEAP
HIERARCHY
HIGH_PRIORITY
HOLD
HOLDLOCK
HOST
HOSTS
HOUR
HOUR_MICROSECOND
HOUR_MINUTE
HOUR_SECOND
IDENTIFIED
IDENTITY
IDENTITY_INSERT
IDENTITYCOL
IF
IGNORE
ILIKE
IMMEDIATE
IMMUTABLE
IMPLEMENTATION
IMPLICIT
IN
INCLUDE
INCLUDING
INCREMENT
INDEX
INDICATOR
INFILE
INFIX
INHERIT
INHERITS
INITIAL
INITIALIZE
INITIALLY
INNER
INOUT
INPUT
INSENSITIVE
INSERT
INSERT_ID
INSTANCE
INSTANTIABLE
INSTEAD
INT
INT1
INT2
INT3
INT4
INT8
INTEGER
INTERSECT
INTERSECTION
INTERVAL
INTO
INVOKER
IS
ISAM
ISNULL
ISOLATION
ITERATE
JOIN
K
KEY
KEY_MEMBER
KEY_TYPE
KEYS
KILL
LANCOMPILER
LANGUAGE
LARGE
LAST
LAST_INSERT_ID
LATERAL
LEADING
LEAST
LEAVE
LEFT
LENGTH
LESS
LEVEL
LIKE
LIMIT
LINENO
LINES
LISTEN
LN
LOAD
LOCAL
LOCALTIME
LOCALTIMESTAMP
LOCATION
LOCATOR
LOCK
LOGIN
LOGS
LONG
LONGBLOB
LONGTEXT
LOOP
LOW_PRIORITY
LOWER
M
MAP
MATCH
MATCHED
MAX
MAX_ROWS
MAXEXTENTS
MAXVALUE
MEDIUMBLOB
MEDIUMINT
MEDIUMTEXT
MEMBER
MERGE
MESSAGE_LENGTH
MESSAGE_OCTET_LENGTH
MESSAGE_TEXT
METHOD
MIDDLEINT
MIN
MIN_ROWS
MINUS
MINUTE
MINUTE_MICROSECOND
MINUTE_SECOND
MINVALUE
MLSLABEL
MOD
MODE
MODIFIES
MODIFY
MODULE
MONTH
MONTHNAME
MORE
MOVE
MULTISET
MUMPS
MYISAM
NAME
NAMES
NATIONAL
NATURAL
NCHAR
NCLOB
NESTING
NEW
NEXT
NO
NO_WRITE_TO_BINLOG
NOAUDIT
NOCHECK
NOCOMPRESS
NOCREATEDB
NOCREATEROLE
NOCREATEUSER
NOINHERIT
NOLOGIN
NONCLUSTERED
NONE
NORMALIZE
NORMALIZED
NOSUPERUSER
NOT
NOTHING
NOTIFY
NOTNULL
NOWAIT
NULL
NULLABLE
NULLIF
NULLS
NUMBER
NUMERIC
OBJECT
OCTET_LENGTH
OCTETS
OF
OFF
OFFLINE
OFFSET
OFFSETS
OIDS
OLD
ON
ONLINE
ONLY
OPEN
OPENDATASOURCE
OPENQUERY
OPENROWSET
OPENXML
OPERATION
OPERATOR
OPTIMIZE
OPTION
OPTIONALLY
OPTIONS
OR
ORDER
ORDERING
ORDINALITY
OTHERS
OUT
OUTER
OUTFILE
OUTPUT
OVER
OVERLAPS
OVERLAY
OVERRIDING
OWNER
PACK_KEYS
PAD
PARAMETER
PARAMETER_MODE
PARAMETER_NAME
PARAMETER_ORDINAL_POSITION
PARAMETER_SPECIFIC_CATALOG
PARAMETER_SPECIFIC_NAME
PARAMETER_SPECIFIC_SCHEMA
PARAMETERS
PARTIAL
PARTITION
PASCAL
PASSWORD
PATH
PCTFREE
PERCENT
PERCENT_RANK
PERCENTILE_CONT
PERCENTILE_DISC
PLACING
PLAN
PLI
POSITION
POSTFIX
POWER
PRECEDING
PRECISION
PREFIX
PREORDER
PREPARE
PREPARED
PRESERVE
PRIMARY
PRINT
PRIOR
PRIVILEGES
PROC
PROCEDURAL
PROCEDURE
PROCESS
PROCESSLIST
PUBLIC
PURGE
QUOTE
RAID0
RAISERROR
RANGE
RANK
RAW
READ
READS
READTEXT
REAL
RECHECK
RECONFIGURE
RECURSIVE
REF
REFERENCES
REFERENCING
REGEXP
REGR_AVGX
REGR_AVGY
REGR_COUNT
REGR_INTERCEPT
REGR_R2
REGR_SLOPE
REGR_SXX
REGR_SXY
REGR_SYY
REINDEX
RELATIVE
RELEASE
RELOAD
RENAME
REPEAT
REPEATABLE
REPLACE
REPLICATION
REQUIRE
RESET
RESIGNAL
RESOURCE
RESTART
RESTORE
RESTRICT
RESULT
RETURN
RETURNED_CARDINALITY
RETURNED_LENGTH
RETURNED_OCTET_LENGTH
RETURNED_SQLSTATE
RETURNS
REVOKE
RIGHT
RLIKE
ROLE
ROLLBACK
ROLLUP
ROUTINE
ROUTINE_CATALOG
ROUTINE_NAME
ROUTINE_SCHEMA
ROW
ROW_COUNT
ROW_NUMBER
ROWCOUNT
ROWGUIDCOL
ROWID
ROWNUM
ROWS
RULE
SAVE
SAVEPOINT
SCALE
SCHEMA
SCHEMA_NAME
SCHEMAS
SCOPE
SCOPE_CATALOG
SCOPE_NAME
SCOPE_SCHEMA
SCROLL
SEARCH
SECOND
SECOND_MICROSECOND
SECTION
SECURITY
SELECT
SELF
SENSITIVE
SEPARATOR
SEQUENCE
SERIALIZABLE
SERVER_NAME
SESSION
SESSION_USER
SET
SETOF
SETS
SETUSER
SHARE
SHOW
SHUTDOWN
SIGNAL
SIMILAR
SIMPLE
SIZE
SMALLINT
SOME
SONAME
SOURCE
SPACE
SPATIAL
SPECIFIC
SPECIFIC_NAME
SPECIFICTYPE
SQL
SQL_BIG_RESULT
SQL_BIG_SELECTS
SQL_BIG_TABLES
SQL_CALC_FOUND_ROWS
SQL_LOG_OFF
SQL_LOG_UPDATE
SQL_LOW_PRIORITY_UPDATES
SQL_SELECT_LIMIT
SQL_SMALL_RESULT
SQL_WARNINGS
SQLCA
SQLCODE
SQLERROR
SQLEXCEPTION
SQLSTATE
SQLWARNING
SQRT
SSL
STABLE
START
STARTING
STATE
STATEMENT
STATIC
STATISTICS
STATUS
STDDEV_POP
STDDEV_SAMP
STDIN
STDOUT
STORAGE
STRAIGHT_JOIN
STRICT
STRING
STRUCTURE
STYLE
SUBCLASS_ORIGIN
SUBLIST
SUBMULTISET
SUBSTRING
SUCCESSFUL
SUM
SUPERUSER
SYMMETRIC
SYNONYM
SYSDATE
SYSID
SYSTEM
SYSTEM_USER
TABLE
TABLE_NAME
TABLES
TABLESAMPLE
TABLESPACE
TEMP
TEMPLATE
TEMPORARY
TERMINATE
TERMINATED
TEXT
TEXTSIZE
THAN
THEN
TIES
TIME
TIMESTAMP
TIMEZONE_HOUR
TIMEZONE_MINUTE
TINYBLOB
TINYINT
TINYTEXT
TO
TOAST
TOP
TOP_LEVEL_COUNT
TRAILING
TRAN
TRANSACTION
TRANSACTION_ACTIVE
TRANSACTIONS_COMMITTED
TRANSACTIONS_ROLLED_BACK
TRANSFORM
TRANSFORMS
TRANSLATE
TRANSLATION
TREAT
TRIGGER
TRIGGER_CATALOG
TRIGGER_NAME
TRIGGER_SCHEMA
TRIM
TRUE
TRUNCATE
TRUSTED
TSEQUAL
TYPE
UESCAPE
UID
UNBOUNDED
UNCOMMITTED
UNDER
UNDO
UNENCRYPTED
UNION
UNIQUE
UNKNOWN
UNLISTEN
UNLOCK
UNNAMED
UNNEST
UNSIGNED
UNTIL
UPDATE
UPDATETEXT
UPPER
USAGE
USE
USER
USER_DEFINED_TYPE_CATALOG
USER_DEFINED_TYPE_CODE
USER_DEFINED_TYPE_NAME
USER_DEFINED_TYPE_SCHEMA
USING
UTC_DATE
UTC_TIME
UTC_TIMESTAMP
VACUUM
VALID
VALIDATE
VALIDATOR
VALUE
VALUES
VAR_POP
VAR_SAMP
VARBINARY
VARCHAR
VARCHAR2
VARCHARACTER
VARIABLE
VARIABLES
VARYING
VERBOSE
VIEW
VOLATILE
WAITFOR
WHEN
WHENEVER
WHERE
WHILE
WIDTH_BUCKET
WINDOW
WITH
WITHIN
WITHOUT
WORK
WRITE
WRITETEXT
X509
XOR
YEAR
YEAR_MONTH
ZEROFILL
ZONE
```