# Explanation
- Here is a SQLite command (not an [[0x00_Querying#SQL|SQL]] keyword) that can shed more light on how a database was created: `.schema`.
- On running this, we see the SQL statement used to create a table. This shows us the columns inside the table and the types of data that each column is able to store.
- To see the schema for a specified table: `.schema books`.
##### Creating a Database Schema
- Now that we have seen the schema for an existing database, let us create our own! We are tasked with representing the subway system of the city of Boston through a database schema. This includes the subway stations, the different train lines, and the people who take the trains.
- To break down the question further, we need to decide…
	- what kinds of tables we will have in our Boston Subway database,
	- what columns each of the tables will have, and
	- what types of data we should put in each of those columns.
##### Normalizing
- Observe this initial attempt at creating a table to represent Boston Subway data. This table contains subway rider names, current stations the riders are at and the action performed at the station. 
- It also records the fares paid and balance amounts on their subway cards. This table also contains an ID for each rider “transaction”, which serves as the primary key.
 ![[initial_attempt.jpg|600]]
- What redundancies exist in this table?
	- We may choose to separate out rider names into a table of its own, to avoid having to duplicate the names so many times. We would need to give each rider an ID that can be used to relate the new table to this one.
	- We may similarly choose to move subway stations to a different table and give each subway station an ID to be used as a foreign key here.
- The process of separating our data in this manner is called **normalizing**. 
- When normalizing, we put each entity in its own table—as we did with riders and subway stations. Any information about a specific entity, for example a rider’s address, goes into the entity’s own table. Then, We now need to decide how our entities are related.
##### CREATE TABLE
- We run the following command to create the first table for riders:
```SQL
CREATE TABLE riders (
    "id",
    "name"
);
```
- Similarly, let us create a table for stations as well.
```SQL
CREATE TABLE stations (
    "id",
    "name",
    "line"
);
```
- Next, we will create a table to relate these two entities. These tables are often called **junction tables**, **associative entities** or **join tables**.
```SQL
CREATE TABLE visits (
    "rider_id",
    "station_id"
);
```
##### Data Types and Storage Classes
- SQLite has five storage classes:
    - **Null**: nothing, or empty value
    - **Integer**: numbers without decimal points
    - **Real**: decimal or floating point numbers
    - **Text**: characters or strings
    - **Blob**: Binary Large Object, for storing objects in binary (useful for images, audio etc.)
- A storage class can hold several data types. For example, these are the data types that fall under the umbrella of the Integer storage class.
	 ![[integer_storage_class.jpg|600]]
- SQLite takes care of storing the input value under the right data type. In other words, we as programmers only need to choose a storage class and SQLite will do the rest.
##### Type Affinities
- It is possible to specify the data type of a column while creating a table. However, columns in SQLite don’t always store one particular data type. 
- They are said to have **type affinities**, meaning that they try to convert an input value into the type they have an affinity for.
- The five type affinities in SQLite are: **Text**, **Numeric** (either integer or real values based on what the input value best converts to), **Integer**, **Real** and **Blob**.
- Consider a column with a type affinity for Integers. If we try to insert “25” (the number 25 but stored as text) into this column, it will be converted into an integer data type.
##### Adding Types to our Tables
- To drop (or delete) the existing tables use the following command: `DROP TABLE "riders";`
- let’s type out the schemas again, but with the affinity types this time.
```SQL
CREATE TABLE riders (
    "id" INTEGER,
    "name" TEXT
);

CREATE TABLE stations (
    "id" INTEGER,
    "name" TEXT,
    "line" TEXT
);

CREATE TABLE visits (
    "rider_id" INTEGER,
    "station_id" INTEGER
);
```
##### Table Constraints
- We can use table constraints to impose restrictions on certain values in our tables.
- For example, a [[0x01_Relating#^211dc1|primary key]] column must have unique values. The table constraint we use for this is `PRIMARY KEY`.
- Similarly, a constraint on a [[0x01_Relating#^9023b9|foreign key]] value is that it must be found in the primary key column of the related table! This table constraint is called, predictably, `FOREIGN KEY`.
- Let’s add primary and foreign key constraints to our `schema.sql` file.
```SQL
CREATE TABLE riders (
    "id" INTEGER,
    "name" TEXT,
    PRIMARY KEY("id")
);

CREATE TABLE stations (
    "id" INTEGER,
    "name" TEXT,
    "line" TEXT,
    PRIMARY KEY("id")
);

CREATE TABLE visits (
    "rider_id" INTEGER,
    "station_id" INTEGER,
    FOREIGN KEY("rider_id") REFERENCES "riders"("id"),
    FOREIGN KEY("station_id") REFERENCES "stations"("id")
);
```
- In the `visits` table, there is no primary key. However, SQLite gives every table a primary key by default, known as the **row ID**. Even though the row ID is implicit, it can be queried.
- It is also possible to create a primary key composed of two columns. For example, if we wanted to give `visits` a primary key composed of both the rider and stations IDs, we could use this syntax
```SQL
CREATE TABLE visits (
    "rider_id" INTEGER,
    "station_id" INTEGER,
    PRIMARY KEY("rider_id", "station_id")
);
```
##### Column Constraints
- A column constraint is a type of constraint that applies to a specified column in the table.
- SQLite has four column constraints:
    - `CHECK`: allows checking for a condition, like all values in the column must be greater than 0
    - `DEFAULT`: uses a default value if none is supplied for a row
    - `NOT NULL`: dictates that a null or empty value cannot be inserted into the column
    - `UNIQUE`: dictates that every value in this column must be unique
```SQL
CREATE TABLE riders (
    "id" INTEGER,
    "name" TEXT,
    PRIMARY KEY("id")
);

CREATE TABLE stations (
    "id" INTEGER,
    "name" TEXT NOT NULL UNIQUE,
    "line" TEXT NOT NULL,
    PRIMARY KEY("id")
);

CREATE TABLE visits (
    "rider_id" INTEGER,
    "station_id" INTEGER,
    FOREIGN KEY("rider_id") REFERENCES "riders"("id"),
    FOREIGN KEY("station_id") REFERENCES "stations"("id")
);
```
- Primary key columns and by extension, foreign key columns must always have unique values, so there is **no need** to explicitly specify the `NOT NULL` or `UNIQUE` column constraints. The table constraint `PRIMARY KEY` includes these column constraints.
##### Altering Tables
- We could alter the `visits` table in the following way:
```SQL
ALTER TABLE "visits"
RENAME TO "swipes";
```
- If We also need to add some columns:
```SQL
ALTER TABLE "swipes"
ADD COLUMN "swipetype" TEXT;
```
- We also have the ability to rename a column in an `ALTER TABLE` command. If we wanted to rename the column `"swipetype"` to make it less wordy, perhaps, we could try the following.
```SQL
ALTER TABLE "swipes"
RENAME COLUMN "swipetype" TO "type";
```
- Finally, we have the ability to drop (or remove) a column.
```SQL
ALTER TABLE "swipes"
DROP COLUMN "type";
```
- It is also possible to return to the schema file that we had originally and simply make these changes there instead of altering tables.
```SQL
CREATE TABLE "cards" (
    "id" INTEGER,
    PRIMARY KEY("id")
);

CREATE TABLE "stations" (
    "id" INTEGER,
    "name" TEXT NOT NULL UNIQUE,
    "line" TEXT NOT NULL,
    PRIMARY KEY("id")
);

CREATE TABLE "swipes" (
    "id" INTEGER,
    "card_id" INTEGER,
    "station_id" INTEGER,
    "type" TEXT NOT NULL CHECK("type" IN ('enter', 'exit', 'deposit')),
    "datetime" NUMERIC NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "amount" NUMERIC NOT NULL CHECK("amount" != 0),
    PRIMARY KEY("id"),
    FOREIGN KEY("station_id") REFERENCES "stations"("id"),
    FOREIGN KEY("card_id") REFERENCES "cards"("id")
);
```
- If we don’t specify a type affinity of a column in SQLite, what happens? The default type affinity is **numeric**, so the column would get assigned the numeric type affinity.
# Sources
- [CS50 SQL - Lecture 2 - Designing](https://cs50.harvard.edu/sql/2024/weeks/2/).