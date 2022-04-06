# Chapter 3: Complex Modeling and Querying
## Modeling: Data Redundancy
### Problems of Redundancy
1. data size
2. data integrity
### Types of Redundancy
1. duplicate rows
2. duplicate column values
3. subkeys
### Subkey Redundancy
- a **subkey** is a set of columns whose values are a **functional dependency** of another set of columns.
- *Z* is functionally dependent on *Y* if knowing *Y* tells you exactly what *Z* is.
- **there is never a reason to store Z in a table when Y is already stored**
### Subkey Elimination
1. if table is already associated with another table where the subkey can be found: eliminate subkey from original subkey
2. if the subkey is internal to a table (*Z* is functionally dependent on *Y*): make *Z* a new table T' and copy *Y* to T' as it's PK as well as the FK of T (T' becomes one-to-many parent)
### Repeated-Value and Multi-Valued Columns
1. **repeated**: a collection of values that aren't shared between instances
2. **multi**: a list
solution: pretty much the same as before-- use a **junction table**/**association class**.
### Enumerated Domains
- you can enumerate a domain within a column (common mappings) or within a new table (large mappings) 
## Modeling: Subclasses
### Inheritance
- open hollow headed arrow in uml
- two dimensions:
	1. **complete** or **incomplete** inheritance
		- **complete**: no instances of base type (abstract)
		- **incomplete**: there are instances of base type
	2. **disjoint** or **overlapping** inheritance
		- **disjoint**: an object can only be of one derived type
		- **overlapping**: an object can be an instance of many of the derived types
- a **joined-subclass** inheritance has a FK of its parents PK that is **also its PK** 
	+ use `INNER JOIN` or `LEFT OUTER JOIN` on the base class to select all or some objects of derived classes 
		* an alternative to this is just putting derived classes in a single table (**single-table inheritance**)
### Aggregation
- ownership: child class forms some *integral part* of its parent; parent *owns* child
- aggregation: *A* owns *B* but *B* can exist without *A* (open diamond/0..1 on parent class in UML)
- composition: aggregation but child objects have no purpose/don't exist without the parent 
### Recursion
- an association that connects one class to itself
## Modeling: Normalization
- applying rules to improve a relational model by removing redundancy and integrity issues
## Querying: Functions and Aggregates
### Aggregate Functions
- they take a **collection of rows** and produce a **single value** as output
- `SELECT` cannot include columns that are not in the column list (only with `GROUP BY`)
### Grouping
- `GROUP BY` organizes selected rows into groups based on their column value, then applies an aggregate within just that group
	+ allows us to find many rows related to the aggregate
### Filtering
- since `WHERE` happens before `SELECT` an aggregate cannot  be used as a filter
- must use `HAVING` after `GROUP BY` instead
	+ aggregate must be repeated in `HAVING` verbatim
### Subqueries
- essentially filtering using the **result of a query**, can almost go anywhere in `SELECT`, including `HAVING`

**scalar-valued subqueries**: a `SELECT` that only returns a single value (aggregates can be used here)
```
SELECT *
FROM CUSTOMERS
WHERE SALESREPEMPLOYEENUMBER = (
	SELECT EMPLOYEENUMBER
	FROM EMPLOYEES
	WHERE FIRSTNAME = 'Julie'
)  
```
**vector-valued subqueries**: a subquery that returns many scalar values
-	can't be used in comparisons
-	can only test for containment or membership
```
SELECT CUSTOMERNAME
FROM CUSTOMERS
WHERE STATE IN (
	SELECT STATE // finds states with 2+ customers
	FROM CUSTOMERS
	GROUP BY STATE
	HAVING COUNT(*) > 1
)
```
**table-valued subqueries**: returns a collection of rows rather than individual values
```
SELECT *
FROM ( // creating a "fake" table
	SELECT NAME, SINGLES + DOUBLES + TRIPLES + HOME_RUNS AS TOTAL_HITS
	FROM PLAYERS
) AS BIG_HITTERS
WHERE TOTAL_HITS > 100
```
## Querying: Triggers
- constraints that cannot be expressed in the relational model are expressed using **triggers**, an RDBM function that executes when some action occurs on a table
- runs `BEFORE`/`AFTER` an `INSERT`/`UPDATE`/`DELETE` `FOR EACH ROW`/`FOR STATEMENT`

**triggers** are paired with **functions**

```
CREATE TRIGGER employees_insert_check
BEFORE INSERT on employees
FOR EACH ROW EXECUTE PROCEDURE check_employee_reportsto();
```

trigger function:
```
CREATE OR REPLACE FUNCTION check_employee_reportsto()
 RETURNS TRIGGER
 LANGUAGE PLPGSQL
 AS
$$
BEGIN
  IF NEW.reportsto = NEW.employeenumber THEN
  	  RAISE EXCEPTION 'An Employee cannot report to themself.';
  END IF;
  
  RETURN NEW;
END;
```