# Explanation
- Here is the [[0x01_Relating#Entity Relationship Diagrams|ER Diagram]] detailing the entities and their relationships of our database.
	 ![[imdb.jpg|800]]
##### Index
- To find the information pertaining to the movie Cars, we would run the following query.
```SQL
SELECT * FROM "movies"
WHERE "title" = 'Cars';
```
- Say we want to find how long it took for this query to run. SQLite has a command `.timer on` that enables us to time our queries.
- On running the above query to find Cars again, we can see three different time measurements displayed along with the results.
- `real` time indicates the **stopwatch** time, or the time between executing the query and obtaining the results. The time taken to execute this query during lecture was roughly a tenth of a second.
- Under the hood, when the query to find Cars was run, we triggered a **scan** of the table `movies` — that is, the table `movies` was scanned top to bottom, one row at a time, to find all the rows with the title Cars.
- We can optimize this query to be more efficient than a scan. In the same way that textbooks often have an index, databases tables can have an index as well. 
- An index, in database terminology, is a **structure used to speed up** the retrieval of rows from a table.
- We can use the following command to create an index for the `"title"` column in the `movies` table.
```SQL
CREATE INDEX "title_index" ON "movies" ("title");
```
- After creating this index, we run the query to find the movie titled Cars again. On this run, the time taken is significantly shorter.
- In the previous example, once the index was created, we just assumed that SQL would use it to find a movie. However, we can also explicitly see this by using a SQLite command `EXPLAIN QUERY PLAN` before any query.
- To remove the index we just created, run: `DROP INDEX "title_index";`.
- Do databases not have implicit algorithms to optimize searching? They do, for some columns. 
- In SQLite and most other database management systems, if we specify that a column is a [[0x01_Relating#^211dc1|primary key]], an index will automatically be created via which we can search for the primary key. 
- However, for regular columns like `"title"`, there would be no automatic optimization.
##### Index across Multiple Tables
- We would run the following query to find all the movies Tom Hanks starred in.
```SQL
SELECT "title" FROM "movies"
WHERE "id" IN (
    SELECT "movie_id" FROM "stars"
    WHERE "person_id" = (
        SELECT "id" FROM "people"
        WHERE "name" = 'Tom Hanks'
    )
);
```
- To understand what kind of index could help speed this query up, we can run `EXPLAIN QUERY PLAN` ahead of this query again. This shows us that the query requires two scans — of `people` and `stars`. 
- The table `movies` is not scanned because we are searching `movies` by its ID, for which an index is automatically created by SQLite.
	 ![[query_plan.PNG]]
- Let us create the two indexes to speed this query up.
```SQL
CREATE INDEX "person_index" ON "stars" ("person_id");
CREATE INDEX "name_index" ON "people" ("name");
```
- Now, we run `EXPLAIN QUERY PLAN` with the same nested query. We can observe that the search on the table `people` uses something called a `COVERING INDEX`.
	 ![[covering index.PNG]]
- A **covering index** means that **all the information needed** for the query can be found within the **index itself**. Instead of two steps:
	1. looking up relevant information in the index.
	2. using the index to then search the table, a covering index means that we do our search in one step (just the first one).
- To have our search on the table `stars` also use a covering index, we can add `"movie_id"` to the index we created for `stars`. 
- This will ensure that the information being looked up (`"movie_id"`) and the value being searched on (`"person_id"`) are both be in the index.
```SQL
CREATE INDEX "person_index" ON "stars" ("person_id", "movie_id");
```
The query now runs a _lot_ faster than it did without indexes.
##### Space Trade-off
- Indexes seem incredibly helpful, but there are trade-offs associated — they occupy additional space in the database, so while we gain query speed, we do lose space.
- An index is stored in a database as a data structure called a B-Tree, or [[Binary Search Trees (BST)|balanced tree]]. A tree data structure looks something like:
	 ![[b_tree.jpg|500]]
- Let us consider how an index is created for the `"title"` column of the table `movies`. If the movie titles were sorted alphabetically, it would be a lot easier to find a particular movie by using binary search.
- In this case, a copy is made of the `"titles"` column. This copy is sorted and then linked back to the original rows within the `movies` table by pointing to the movie IDs.
	 ![[copy.jpg|500]]
- While this helps us visualize the index for this column easily, in reality, the index is not a single column but is broken up into many nodes. 
- This is because if the database has a lot of data, like IMDb, storing one column all together in memory might not be feasible.
- If we have multiple nodes containing sections of the index, however, we also need nodes to navigate to the right sections. For example, consider the following nodes. The left-hand node directs us to the right section of the index based on whether the movie title comes before Frozen, between Frozen and Soul, or after Soul alphabetically.
	 ![[broken_up_nodes.jpg|600]]
##### Time Trade-off
- Similar to the space trade-off we discussed earlier, it also takes longer to insert data into a column and then add it to an index. Each time a value is added to the index, the B-tree needs to be traversed to figure out where the value should be added.
##### Partial Index
- This is an index that includes only a **subset of rows from a table**, allowing us to save some space that a full index would occupy.
- This is especially useful when we know that users query only a subset of rows from the table. 
- In the case of IMDb, it may be that the users are more likely to query a movie that was just released as opposed to a movie that is 15 years old. Let’s try to create a partial index that stores the titles of movies released in 2023.
```SQL
CREATE INDEX "recents" ON "movies" ("titles")
WHERE "year" = 2023;
```
##### Vacuum
- There are ways to delete unused space in our database. SQLite allows us to `vacuum` data — this cleans up previously deleted data (that is actually not deleted, but just marked as space being available for the next `INSERT`).
- To find the size of `movies.db` on the terminal, we can use a Unix command 
```bash
du -b movies.db
```
- We can now connect to our database and drop an index we previously created.
- Now, if we run the Unix command again, we see that the size of the database has not decreased! To actually clean up the deleted space, we need to vacuum it. We can run the following command in SQLite: `VACUUM;`.
- On running the Unix command to check the size of the database again, we can should see a smaller size.
##### Concurrency
- Concurrency is the **simultaneous** handling of multiple queries or interactions by the database. Imagine a database for a financial service, that gets a lot of traffic at the same time.
- For example, consider a bank’s database. One transaction could be sending money from one account to the other. For example, Alice is trying to send $10 to Bob.
- To complete this transaction, we would need to add $10 to Bob’s account and also subtract $10 from Alice’s account. 
- If someone sees the status of the `accounts` database after the first update to Bob’s account but before the second update to Alice’s account, they could get an incorrect understanding of the total amount of money held by the bank.
- To an outside observer, it should seem like the different parts of a transaction happen all at once. 
- In database terminology, a transaction is an individual unit of work — something that cannot be broken down into smaller pieces.
- Transactions have some properties, which can be remembered using the acronym ACID:
	- **Atomicity**: can’t be broken down into smaller pieces.
	- **Consistency**: should not violate a database constraint.
	- **Isolation**: if multiple users access a database, their transactions cannot interfere with each other.
	- **Durability**: in case of any failure within the database, all data changed by transactions will remain.
- To move $10 from Alice’s account to Bob’s, we can write the following transaction.
```SQL
BEGIN TRANSACTION;
UPDATE "accounts" SET "balance" = "balance" + 10 WHERE "id" = 2;
UPDATE "accounts" SET "balance" = "balance" - 10 WHERE "id" = 1;
COMMIT;
```
- If we execute the query after writing the `UPDATE` statements, but without committing, neither of the two `UPDATE` statements will be run! This helps keep the transaction **atomic**. By updating our table in this way, we are unable to see the intermediate steps.
- If we tried to run the above transaction again — Alice tries to pay Bob another $10 — it should fail to run because Alice’s account balance is at 0. (The `"balance"` column in `accounts` has a check constraint to ensure that it has a non-negative value.)
- The way we implement reverting the transaction is using `ROLLBACK`. Once we begin a transaction and write some SQL statements, if any of them fail, we can end it with a `ROLLBACK` to revert all values to their pre-transaction state. This helps keep transactions **consistent**.
```SQL
BEGIN TRANSACTION;
UPDATE "accounts" SET "balance" = "balance" + 10 WHERE "id" = 2;
UPDATE "accounts" SET "balance" = "balance" - 10 WHERE "id" = 1; -- Invokes constraint error
ROLLBACK;
```
- Transactions can help guard against [[0x17_Intro to Concurrency#The Heart Of The Problem Uncontrolled Scheduling|race conditions]].
- A race condition occurs when multiple entities simultaneously access and make decisions based on a **shared value**, potentially causing inconsistencies in the database.
- Transactions are processed in **isolation** to avoid the inconsistencies in the first place. Each transaction dealing with similar data from our database will be processed sequentially. This helps prevent the inconsistencies that an adversarial attack can exploit.
- To make transactions sequential, SQLite and other database management systems use **[[0x18_Locks|locks]]** on databases. A table in a database could be in a few different states:

	- **UNLOCKED**: this is the default state when no user is accessing the database.
	- **SHARED**: when a transaction is reading data from the database, it obtains shared lock that allows other transactions to read simultaneously from the database.
	- **EXCLUSIVE**: if a transaction needs to write or update data, it obtains an exclusive lock on the database that does not allow other transactions to occur at the same time (not even a read).
- Do we lock a database, a table or a row of a table? This depends on the DBMS. In SQLite, we can actually do this by running an exclusive transaction as below:
```SQL
BEGIN EXCLUSIVE TRANSACTION;
```
- If we do not complete this transaction now, and try to connect to the database through a different terminal to read from the table, we will get an error that the database is locked. This, of course, is a very coarse way of locking because it locks the entire database.
- Because SQLite is coarse in this manner, it has a module for prioritizing transactions and making sure an exclusive lock is obtained only for the shortest necessary duration.
# Sources
- [CS50 SQL - Lecture 5 - Optimizing](https://cs50.harvard.edu/sql/2024/weeks/5/).