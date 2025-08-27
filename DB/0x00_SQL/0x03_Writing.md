# Explanation
##### Inserting Data
- The [[0x00_Querying#SQL|SQL]] statement `INSERT INTO` is used to insert a row of data into a given table.
```SQL
INSERT INTO "collections" ("id", "title", "accession_number", "acquired")
VALUES (1, 'Profusion of flowers', '56.257', '1956-04-12');
```
- We can see that this command requires the list of columns in the table that will receive new data and the values to be added to each column, in the same order.
- We can add more rows to the database by inserting multiple times. However, typing out the value of the primary key manually (as 1, 2, 3 etc.) might result in errors. 
- Thankfully, SQLite can fill out the primary key values automatically. To make use of this functionality, we omit the ID column altogether while inserting a row.
```SQL
INSERT INTO "collections" ("title", "accession_number", "acquired")
VALUES ('Farmers working at dawn', '11.6152', '1911-08-03');
```
- Notice that the way SQLite fills out the primary key values is by incrementing the previous primary key.
- If we delete a row with the primary key 1, will SQLite automatically assign a primary key of 1 to the next inserted row? No, SQLite actually selects the **highest primary key value** in the table and increments it to generate the next primary key value.
##### Other Constraints
- A schema for the database could be like the following:
```SQL
CREATE TABLE "collections" (
    "id" INTEGER,
    "title" TEXT NOT NULL,
    "accession_number" TEXT NOT NULL UNIQUE,
    "acquired" NUMERIC,
    PRIMARY KEY("id")
);
```
- It is specified that the accession number is unique. If we try to insert a row with a repeated accession number, we will trigger a error that looks like `Runtime error: UNIQUE constraint failed: collections.accession_number (19)`.
- This error informs us that the row we are trying to insert violates a constraint in the schema—specifically the `UNIQUE` constraint in this scenario.
- Similarly, we can try to add a row with a `NULL` title, violating the `NOT NULL` constraint.
```SQL
INSERT INTO "collections" ("title", "accession_number", "acquired")
VALUES(NULL, NULL, '1900-01-10');
```
- On running this, we will again see an error that looks like `Runtime error: NOT NULL constraint failed: collections.title (19)`.
- In this manner, the schema constraints are guardrails that protect us from adding rows that do not follow the schema of our database.
##### Inserting Multiple Rows
- We may need to insert more than one row at a time while writing into a database. One way to do this is to separate out the rows using commas in the `INSERT INTO` command.
```SQL
INSERT INTO "collections" ("title", "accession_number", "acquired") 
VALUES 
('Imaginative landscape', '56.496', NULL),
('Peonies and butterfly', '06.1899', '1906-01-01');
```
- SQLite makes it possible to import a CSV file directly into our database. On opening up CSV file, we can note that the first row contains the column names, which match exactly with the column names of our table `collections` as per the schema.
	 ![[csv.jpg]]
- We can import the CSV by running a SQLite command.
```SQL
.import --csv --skip 1 file.csv table
```
- The first argument, `--csv` indicates to SQLite that we are importing a CSV file. The second argument indicates that the first row of the CSV file (the header row) needs to be skipped, or not inserted into the table.
- The CSV file we just inserted contained primary key values (1, 2, 3 etc.) for each row of data. However, it is more likely that CSV files we work with will not contain the ID or primary key values. How can we have SQLite insert them automatically?
- To try this out, let’s open up the CSV file and delete the `id` column from the header row, along with the values in each column.
- Now, we want to import this CSV file into a table. However, the `collections` table (as per our schema) must have four columns in every row. 
- This new CSV file contains only three columns for every row. Hence, we cannot proceed to import in the same way we did before.
- To successfully import the CSV file without ID values, we will to use a temporary table:
```SQL
.import --csv file.csv temp
```
- Notice how we don’t use the argument `--skip 1` with this command. This is because SQLite is capable of recognizing the very first row of CSV data as the header row, and converts those into the column names of the new `temp` table.
- Next, we will select the data (without primary keys) from `temp` and move it to `collections`. We can use the following command to achieve this.
```SQL
INSERT INTO "collections" ("title", "accession_number", "acquired") 
SELECT "title", "accession_number", "acquired" FROM "temp";
```
- In this process, SQLite will automatically add the primary key values in the `id` column.
- What happens if one of the multiple rows we are trying to insert violates a table constraint? While trying to insert multiple rows into a table, if even one of them violates a constraint, the insertion command will result in an error and none of the rows will be inserted.
##### Deleting Data
- To delete all the rows that are already within a table, we use:
```SQL
DELETE FROM "collections";
```
- We can also delete rows that match specific conditions. we can run:
```SQL
DELETE FROM "collections"
WHERE "title" = 'Spring outing';
```
- To delete rows pertaining to paintings older than 1909, we can run
```SQL
DELETE FROM "collections"
WHERE "acquired" < '1909-01-01';
```
- There might be cases where deleting some data could impact the integrity of a database. [[0x01_Relating#^9023b9|Foreign key]] constraints are a good example. A foreign key column references the primary key of a different table. If we were to delete the primary key, the foreign key column would have nothing to reference!
- Consider now an updated schema our database, containing information not just about artwork but also artists. 
- The two entities Artist and Collection have a many-to-many relationship—a painting can be created by many artists and a single artist can also create many pieces of artwork.
	 ![[update_database.jpg]]
- The `artists` and `collections` tables have primary keys—the ID columns. The `created` table references these IDs in its two foreign key columns.
- Now, we can try to delete from the `artists` table.
```SQL
DELETE FROM "artists"
WHERE "name" = 'Unidentified artist';
```
- On running this, we get an error very similar to ones we have seen before in this class: `Runtime error: FOREIGN KEY constraint failed (19)`. This error notifies us that deleting this data would violate the foreign key constraint set up in the `created` table.
- How do we ensure that the constraint is not violated? One possibility is to delete the corresponding rows from the `created` table before deleting from the `artists` table.
```SQL
DELETE FROM "created"
WHERE "artist_id" = (
    SELECT "id" FROM "artists"
    WHERE "name" = 'Unidentified artist'
);
```
- This query effectively deletes the artist’s _affiliation_ with their work. Once the affiliation no longer exists, we can delete the artist’s data without violating the foreign key constraint. To do this, we can run the query before the above one again.
- In another possibility, we can specify the action to be taken when an ID referenced by a foreign key is deleted. To do this, we use the keyword `ON DELETE` followed by the action to be taken.
	- `ON DELETE RESTRICT`: This restricts us from deleting IDs when the foreign key constraint is violated.
	- `ON DELETE NO ACTION`: This allows the deletion of IDs that are referenced by a foreign key and nothing happens.
	- `ON DELETE SET NULL`: This allows the deletion of IDs that are referenced by a foreign key and sets the foreign key references to `NULL`.
	- `ON DELETE SET DEFAULT`: This does the same as the previous, but allows us to set a default value instead of `NULL`.
	- `ON DELETE CASCADE`: This allows the deletion of IDs that are referenced by a foreign key and also proceeds to cascadingly delete the referencing foreign key rows. For example, if we used this to delete an artist ID, all the artist’s affiliations with the artwork would also be deleted from the `created` table.
- Is there any way to make the next inserted row have an ID of a deleted row? we can use the `AUTOINCREMENT` keyword while creating a column to indicate that any deleted ID should be repurposed for a new row being inserted into the table.
##### Updating Data
- We can use the `update` command to make changes to data in a database.
```SQL
UPDATE "created"
SET "artist_id" = (
    SELECT "id" FROM "artists"
    WHERE "name" = 'Li Yin'
)
WHERE "collection_id" = (
    SELECT "id" FROM "collections"
    WHERE "title" = 'Farmers working at dawn'
);
```
##### Triggers
- A trigger is a named database object that is associated with a table, and that activates when a particular event occurs for the table.
- Triggers are useful for automating repetitive tasks. You can just set up a trigger to do some calculation after every specific database action. You can also set up a trigger to perform data validation tasks on a table.
- A trigger gets fired when an `INSERT`, `UPDATE` or `DELETE` operation happens on a database table. 
- A trigger is fired per row, so if multiple rows of data are being inserted or deleted, each one still fires the action setup by the trigger. A trigger can be set to fire before or after an action.
- To create a new trigger, use the `CREATE TRIGGER` command. This command has the following structure:
```SQL
CREATE TRIGGER trigger_name
trigger_time trigger_event ON table_name
FOR EACH ROW
BEGIN
trigger_body;
END;
```
- The `trigger_time` is a variable value that can only be either `BEFORE` or `AFTER`. This determines whether the trigger will fire before or after the event has happened.
- The `trigger_event` is another variable that has a limited number of possible options. This variable cannot be any value other than `INSERT`, `UPDATE`, or `DELETE`. It specifies what event to listen for.
```SQL
CREATE TRIGGER "sell"
BEFORE DELETE ON "collections" 
FOR EACH ROW
BEGIN
	INSERT INTO "transactions" ("title", "action")
	VALUES("OLD.title", 'sold');
END;
```
- The trigger actions may access elements of the row being inserted, deleted or updated using references of the form `NEW.column-name` and `OLD.column-name`, where _column-name_ is the name of a column from the table that the trigger is associated with. 
- `OLD` and `NEW` references may only be used in triggers on events for which they are relevant, as follows:
	- **_INSERT:_** `NEW`references are valid.
	- **UPDATE:_** `NEW` and `OLD` references are valid.
	- **_DELETE:_** `OLD` references are valid.
- To drop the trigger, use the `DROP TRIGGER` command. The command only requires the name of the trigger. You can use the command like this:
```sql
DROP TRIGGER password_hasher;
```
# Sources
- [CS50 SQL - Lecture 3 - Writing](https://cs50.harvard.edu/sql/2024/weeks/3/).