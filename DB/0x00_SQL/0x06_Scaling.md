# Explanation
- We will use different database management systems like MySQL and PostgreSQL which can be used to scale databases.
- SQLite is an **embedded** database, but MySQL and PostgreSQL are **database servers** — they often run on their **own dedicated hardware** that we can connect to over the internet to run our SQL queries. 
- This confers them the advantage of being able to store their data on RAM, resulting in faster queries.
##### MySQL
- On the terminal, let’s connect to a MySQL server using root:
```
mysql -u root -h 127.0.0.1 -P 3306 -p
```
- In this terminal command, `-u` indicates the user. We provide the user we want to connect to the database as — `root` (synonymous with database admin, in this case).
- `127.0.0.1` is the address of local host on the internet (our own computer).
- `3306` is the port we want to connect to, and this is the default port where MySQL is hosted.
- `-p` at the end of the command indicates that we want to be prompted for a password when connecting.
- Since this a full database server with potentially many databases inside it. To show all the existing ones, we use the following MySQL command.
```
SHOW DATABASES;
```
- This returns some default databases already in the server.
- We will perform some operations to set up the MBTA database.
- Creating a new database:
```
CREATE DATABASE `mbta`;
```
- Instead of quotation marks, we use backticks to identify the table name and other variables in our SQL statements.
- To change the current database to `mbta`:
```
USE `mbta`;
```
##### Creating the cards Table
- MySQL has more granularity with types than SQLite. For example, an integer could be `TINYINT`, `SMALLINT`, `MEDIUMINT`, `INT` or `BIGINT` based on the size of the number we want to store. 
- The following table shows us the size and range of numbers we can store in each of the integer types.
	 ![[mysql_int.jpg|600]]
- Let us now create the table `cards` using an `INT` data type for the ID column.
```SQL
CREATE TABLE `cards` (
    `id` INT AUTO_INCREMENT,
    PRIMARY KEY(`id`)
);
```
- Note that we use the keyword `AUTO_INCREMENT` with the ID so that MySQL automatically inserts the next number as the ID for a new row.
##### Creating the stations Table
- After creating the table, we can see a list of the existing tables by running: 
```
SHOW TABLES;
```
- For further details about a table, we can use the `DESCRIBE` command.
	 ![[describe_mysql.PNG|600]]
```
DESCRIBE `cards`;
```
- To handle text, MySQL provides many types. Two commonly used ones are `CHAR` — a fixed width string, and `VARCHAR` — a string of variable length. 
- MySQL also has a type `TEXT` but unlike in SQLite, this type is used for longer chunks of text like paragraphs, pages of books etc. 
- Based on the length of the text, it could be one of: `TINYTEXT`, `TEXT`, `MEDIUMTEXT` and `LONGTEXT`. Additionally, we have the `BLOB` type to store binary strings.
- MySQL also provides two other text types: `ENUM` and `SET`. 
	- `Enum` restricts a column to a single predefined option from a list of options we provide. 
		- For example, shirt sizes could be enumerated to M, L, XL and so on. 
	- A `set` allows for multiple options to be stored in a single cell, useful for scenarios like movie genres.
- Now, let us create the `stations` table in MySQL.
```SQL
CREATE TABLE `stations` (
    `id` INT AUTO_INCREMENT,
    `name` VARCHAR(32) NOT NULL UNIQUE,
    `line` ENUM('blue', 'green', 'orange', 'red') NOT NULL,
    PRIMARY KEY(`id`)
);
```
- On running the command to describe this table, we see a similar output that lists out each of the columns in the table. 
- Under the `Key` field, the primary key is recognized by `PRI` and any column with unique values is recognized by `UNI`. 
- The `NULL` field tells us which columns allow `NULL` values, which none of the columns do for the `stations` table.
	 ![[stations_mysql.PNG]]
- If we do not know how long a piece of text will be and use something like `VARCHAR(300)` to represent it, is that okay?
	- While this is okay, there is a trade-off here. We will lose 300 bytes of memory for every row of data inserted, which might not be worth it if we end up storing only very small strings. 
	- It might be better to start off with a smaller length and then alter the table to increase length if needed.
##### Creating the swipes Table
- MySQL provides us with some options for storing dates and times, while in SQLite they had to be stored using the numeric type.
- We could use `DATE`, `YEAR`, `TIME`, `DATETIME` and `TIMESTAMP` (for more precise times) to store our date and time values. 
- The last three allow an optional parameter to specify the precision with which we want to store the time.
- In SQLite, we had a `REAL` data type. Here, our options are `FLOAT` and `DOUBLE PRECISION`.
	 ![[real_mysql.jpg|500]]
- There is also a way in MySQL to use a `decimal` (fixed precision) type. With this, we would specify the number of digits in the number to be represented, and the number of digits after the decimal point.
```SQL
CREATE TABLE `swipes` (
    `id` INT AUTO_INCREMENT,
    `card_id` INT,
    `station_id` INT,
    `type` ENUM('enter', 'exit', 'deposit') NOT NULL,
    `datetime` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `amount` DECIMAL(5,2) NOT NULL CHECK(`amount` != 0),
    PRIMARY KEY(`id`),
    FOREIGN KEY(`station_id`) REFERENCES `stations`(`id`),
    FOREIGN KEY(`card_id`) REFERENCES `cards`(`id`)
);
```
- Notice the use of `DEFAULT CURRENT_TIMESTAMP` to indicate that the timestamp should be auto-filled to store the current time if no value is provided.
- Does MySQL have [[0x02_Designing#Type Affinities|type affinities]]? Not exactly. MySQL does have data types, like `INT` and `VARCHAR` but unlike SQLite, it will not **allow us** to enter data of a different type and try to convert it.
##### Altering Tables
- If we wanted to add a silver line to the possible lines a station could be on, we can do the following.
```SQL
ALTER TABLE `stations` 
MODIFY `line` ENUM('blue', 'green', 'orange', 'red', 'silver') NOT NULL;
```
##### Stored Procedures
- Stored procedures are a way to **automate** SQL statements and run them repeatedly.
- Before we create a stored procedure, we need to change the delimited from `;` to something else. 
- Unlike SQLite, where we could type in multiple statements between a `BEGIN` and `END`  and end them with a `;`, MySQL prematurely ends the statement when it encounters a `;`.
```
delimiter //
```
- Now, we write the stored procedure.
```SQL
CREATE PROCEDURE `current_collection`()
BEGIN
    SELECT `title`, `accession_number`, `acquired` 
    FROM `collections` 
    WHERE `deleted` = 0;
END//
```
- After creating this, we must reset the delimited to `;`.
```
delimiter ;
```
- Let us try calling this procedure to see the current collections.
```SQL
CALL current_collection();
```
##### Stored Procedures with Parameters
- Now, if a piece of artwork is deleted from `collections` because it is being sold, we would also like to update this in the `transactions` table. 
- Usually, this would be two different queries but with a stored procedure, we can give this sequence one name.
```SQL
delimiter //
CREATE PROCEDURE `sell`(IN `sold_id` INT)
BEGIN
    UPDATE `collections` SET `deleted` = 1 
    WHERE `id` = `sold_id`;
    INSERT INTO `transactions` (`title`, `action`)
    VALUES ((SELECT `title` FROM `collections` WHERE `id` = `sold_id`), 'sold');
END//
delimiter ;
```
- We can now call the procedure to sell a particular item.
```SQL
CALL `sell`(2);
```
- What happens if I call `sell` on the same ID more than once? There is a danger of it being added multiple times to the `transactions` table. 
- Stored procedures can be considerably improved in logic and complexity by using some regular old programming constructs. The following list contains some popular constructs available in MySQL.
	 ![[logic_mysql.jpg|500]]
##### PostgreSQL
- Let’s see what data types are available to us in PostgreSQL.
	- **Integers**
		 ![[int_postgresql.jpg|600]]
	- **Serial**
		- Serials are also integers, but they are serial numbers, usually used for primary keys.
- Let us connect to the database server by opening PSQL — the command line interface for PostgreSQL.
```
psql postgresql://postgres@127.0.0.1:5432/postgres
```
- To view all the databases, we can run `\l` and it pulls up a list.
- To create the MBTA database, we can run:
```
CREATE DATABASE "mbta";
```
- To connect to this specific database, we can run `\c "mbta"`.
- To list out all the tables in the database, we can run `\dt`.
- Finally, we can create the `cards` table, as proposed. We use a `SERIAL` data type for the ID column.
```SQL
CREATE TABLE "cards" (
    "id" SERIAL,
    PRIMARY KEY("id")
);
```
- To describe a table in PostgreSQL, we can use a command like `\d "cards"`.
##### Creating PostgreSQL Tables
- The `stations` table is created in a similar manner to MySQL.
```SQL
CREATE TABLE "stations" (
    "id" SERIAL,
    "name" VARCHAR(32) NOT NULL UNIQUE,
    "line" VARCHAR(32) NOT NULL,
    PRIMARY KEY("id")
);
```
- We want to create the `swipes` table next. Recall that the swipe type can mark entry, exit or deposit of funds in the card. 
- Similar to MySQL, we can use an `ENUM` to capture these options, but do not include it in the column definition. Instead, we create our own type.
```SQL
CREATE TYPE "swipe_type" AS ENUM('enter', 'exit', 'deposit');
```
- PostgreSQL has types `TIMESTAMP`, `DATE`, `TIME` and `INTERVAL` to represent date and time values. 
- `INTERVAL` is used to capture how long something took, or the distance between times. Similar to MySQL, we can specify the precision with these types.
- A key difference with real number types in PostgreSQL is that the `DECIMAL` type is called `NUMERIC`.
- We can now go ahead and create the `swipes` table as the following.
```SQL
CREATE TABLE "swipes" (
    "id" SERIAL,
    "card_id" INT,
    "station_id" INT,
    "type" "swipe_type" NOT NULL,
    "datetime" TIMESTAMP NOT NULL DEFAULT now(),
    "amount" NUMERIC(5,2) NOT NULL CHECK("amount" != 0),
    PRIMARY KEY("id"),
    FOREIGN KEY("station_id") REFERENCES "stations"("id"),
    FOREIGN KEY("card_id") REFERENCES "cards"("id")
);
```
- For the default timestamp, we use a function provided to us by PostgreSQL called `now()` that gives us the current timestamp.
- To exit PostgreSQL, we use the command `\q`.
##### Scaling with MySQL
- Consider a database server for an application growing in demand. As the number of reads and writes coming in from the application begin to increase, the wait time for the queries to be processed by the server increases also.
- One approach here is to scale the database **vertically**. Scaling vertically is increasing **capacity** by increasing the **computing power** of the database server.
- Another approach is to scale **horizontally**. This means increasing **capacity** by **distributing load across multiple servers**. When we scale horizontally, we keep copies of our database on multiple servers (**replication**).
- There are three main models of replication: 
	- **Single-leader** replication involves a single database server handling incoming writes and then copying those changes into other servers. 
	- **Multi-leader** replication involves multiple servers receiving updates, leading to increased complexity. 
	- **Leaderless** replication uses a different approach that does not require leaders in this sense.
- In single-leader model of replication model, the follower database server is a read replica: a copy of the database from which data may only be read. The leader server is designated to process writes to the database.
- Once the leader processes a write request, it could wait for the followers to replicate changes before doing anything else. This is called **synchronous replication**. 
- While this ensures the database is always consistent, it may be too slow in responding to queries. 
- In applications like finance or healthcare, where data consistency is extremely important, we might choose this kind of communication despite the disadvantages.
- Another kind is **asynchronous replication**, wherein the leader communicates with follower databases asynchronously to ensure changes are replicated. 
- This method could be used in social media applications, where speed of response is extremely important.
- Another popular way of scaling is called **sharding**. This involves splitting the database into shards across multiple database servers. 
- A word of caution with sharding: we want to avoid having a database hotspot, or a database server that becomes more frequently accessed than others. This could create an overload on that server.
- Another problem arises when we use sharding without replication. In this case, if one of the servers goes down, we will have an incomplete database. This creates a **single point of failure**: if one system goes down, our entire system is not usable.
##### Access Controls
- Previously, we logged into MySQL using the root user. However, we can also create more users and give them some kind of access to the database.
- Let’s create a new user called Carter:
```SQL
CREATE USER 'carter' IDENTIFIED BY 'password';
```
- When we create this new user, by default it has very few privileges with which new user can access some of the default databases in the server.
- If we wanted to share the `analysis` view with the user we just created, we would do the following while logged in as the root user.
```SQL
GRANT SELECT ON `rideshare`.`analysis` TO 'carter';
```
- If we wanted to undo the previous step, we would do the following while logged in as the root user.
```SQL
REVOKE SELECT ON `rideshare`.`analysis` TO 'carter';
```
##### SQL Injection Attacks
- As the name indicates, this involves a malicious user injecting some SQL phrases to complete an existing query within our application in an **undesirable way**.
- For example, a website that asks a user to log in with their username and password may be running a query like this on the database.
```SQL
SELECT `id` FROM `users`
WHERE `user` = 'Carter' AND `password` = 'password';
```
- In the above example, the user Carter entered their username and password as per usual. However, a malicious user could enter something different, like the string “password’ OR ‘1’ = 1” as their password. In this case, they are trying to gain access to the entire database of users and passwords.
```SQL
SELECT `id` FROM `users`
WHERE `user` = 'Carter' AND `password` = 'password' OR '1' = '1';
```
- In MySQL, we can use **prepared statements** to prevent SQL injection attacks.
- An example of an SQL injection attack that can be run to display all user accounts from the `accounts` table is this.
```SQL
SELECT * FROM `accounts`
WHERE `id` = 1 UNION SELECT * FROM `accounts`;
```
- A prepared statement is a statement in SQL that we **can later insert values into**. For the above query, we can write a prepared statement.
```SQL
PREPARE `balance_check`
FROM 'SELECT * FROM `accounts`
WHERE `id` = ?';
```
- To actually run this statement now and check someone’s balance, we accept user input as a variable and then plug it into the prepared statement.
```SQL
SET @id = 1;
EXECUTE `balance_check` USING @id;
```
 ![[prepared_statments.PNG]]
- The prepared statement cleans up input to ensure that no malicious SQL code is injected. Let’s try to run the same statements as above but with a malicious ID.
```SQL
SET @id = '1 UNION SELECT * FROM `accounts`';
EXECUTE `balance_check` USING @id;
```
- This also gives us the same results as the previous code — it shows us the balance of the user with ID 1 and nothing else.
# Sources
- [CS50 SQL - Lecture 6 - Scaling](https://cs50.harvard.edu/sql/2024/weeks/6/).