# Explanation
- A view is a **virtual table** defined by a **query**.
- Say we wrote a query to join three tables and then select the relevant columns. The new table created by this query can be saved as a view, to be further queried later on.
- Views are useful for:
    - **Simplifying**: putting together data from different tables to be queried more simply.
    - **Aggregating**: running aggregate functions, like finding the sum, and storing the results.
    - **Partitioning**: dividing data into logical pieces.
    - **Securing**: hiding columns that should be kept secure.
##### Simplifying
- To save the virtual table created as a view:
```SQL
CREATE VIEW "longlist" AS
SELECT "name", "title" FROM "authors"
JOIN "authored" ON "authors"."id" = "authored"."author_id"
JOIN "books" ON "books"."id" = "authored"."book_id";
```
- The view created here is called `longlist`. This view can now be used exactly as we would use a table in SQL.
- Let us write a query to see all the data within this view.
```SQL
SELECT * FROM "longlist";
```
- A view, being a virtual table, does not consume much more disk space to create. The data within a view is still stored in the underlying tables, but still accessible through this simplified view.
##### Aggregating
- In `longlist.db` we have a table containing individual ratings given to each book. To find the average rating of every book, rounded to 2 decimal places.
```SQL
SELECT "book_id", "title", "year", ROUND(AVG("rating"), 2) AS "rating" 
FROM "ratings"
JOIN "books" ON "ratings"."book_id" = "books"."id"
GROUP BY "book_id";
```
- This **aggregated** data can be stored in a view.
```SQL
CREATE VIEW "average_book_ratings" AS
SELECT "book_id" AS "id", "title", "year", ROUND(AVG("rating"), 2) AS "rating" 
FROM "ratings"
JOIN "books" ON "ratings"."book_id" = "books"."id"
GROUP BY "book_id";
```
- To create temporary views that are not stored in the database schema, we can use `CREATE TEMPORARY VIEW`. This command creates a view that exists only for the duration of our connection with the database.
- To find the average rating of books _per year_, we can use the view we already created.
```SQL
SELECT "year", ROUND(AVG("rating"), 2) AS "rating" 
FROM "average_book_ratings" 
GROUP BY "year";
```
- We can store the results in a temporary view.
```SQL
CREATE TEMPORARY VIEW "average_ratings_by_year" AS
SELECT "year", ROUND(AVG("rating"), 2) AS "rating" FROM "average_book_ratings" 
GROUP BY "year";
```
##### Common Table Expression (CTE)
- A **regular** view exists **forever** in our database schema. A **temporary** view exists for the **duration of our connection** with the database. A **CTE** is a view that exists for a **single query** alone.
- Let us recreate the view containing average book ratings per year using a CTE instead of a temporary view. First, we need to drop the existing temporary view so that we can reuse the name `average_book_ratings`.
```SQL
DROP VIEW "average_book_ratings";
```
- Next, we create a CTE containing the average ratings _per book_. We then use the average ratings per book to calculate the average ratings _per year_, in much the same way as we did before.
```SQL
WITH "average_book_ratings" AS (
    SELECT "book_id", "title", "year", ROUND(AVG("rating"), 2) AS "rating" FROM "ratings"
    JOIN "books" ON "ratings"."book_id" = "books"."id"
    GROUP BY "book_id"
)
SELECT "year" ROUND(AVG("rating"), 2) AS "rating" FROM "average_book_ratings"
GROUP BY "year";
```
##### Partitioning
- Views can be used to partition data, or to break it into smaller pieces that will be useful to us or an application. 
- For example, the website for the International Booker Prize has a page of longlisted books for each year the prize was awarded. 
- However, our database stores all the longlisted books in a single table. For the sake of creating the website, or a different purpose, it might be useful to have a different table (or view) of books for each year.
- Can views be updated? No, because views do not have any data in the way that tables do. Views actually pull data from the underlying tables each time they are queried. 
- This means that when an underlying table is updated, the next time the view is queried, it will display updated data from the table.
##### Securing
- Views can be used to enhance database security by limiting access to certain data.
- Consider a rideshare company’s database with a table `rides` that looks like the following.
	 ![[riders.jpg]]
- If we were to give this data to an analyst, whose job is to find the most popular ride routes, it would be irrelevant and indeed, not secure to give them the names of individual riders. 
- Rider names are likely categorized as Personally Identifiable Information (PII) which companies are not allowed to share indiscriminately.
- Views can be handy in this situation — we can share with the analyst a view containing the origin and destination of rides, but not the rider names.
```SQL
CREATE VIEW "analysis" AS
SELECT "id", "origin", "destination", 'Anonymous' AS "rider" 
FROM "rides";
```
- Although we can create a view that anonymizes data, SQLite does not allow access control. This means that our analyst could simply query the original `rides` table and see all the rider names we went to great lengths to omit in the `analysis` view.
##### Soft Deletions
- A soft deletion involves marking a row as deleted instead of removing it from the table.
- We already know that it is not possible to insert data into or delete data from a view. However, we can set up a [[0x03_Writing#Triggers|trigger]] that inserts into or deletes from the underlying table using `INSTEAD OF` trigger.
```SQL
CREATE TRIGGER "delete"
INSTEAD OF DELETE ON "current_collections"
FOR EACH ROW
BEGIN
    UPDATE "collections" SET "deleted" = 1 
    WHERE "id" = OLD."id";
END;
```
- Every time we try to delete rows from the view, this trigger will instead update the `deleted` column of the row in the underlying table `collections`, thus completing the soft deletion.
- We use the keyword `OLD` within our update clause to indicate that the ID of the row updated in `collections` should be the same as the ID of the row we are trying to delete from `current_collections`.
- Now, we can delete a row from the `current_collections` view.
```SQL
DELETE FROM "current_collections" 
WHERE "title" = 'Imaginative landscape';
```
- Similarly, we can create a trigger that inserts data into the underlying table when we try to insert it into a view.
- There are two situations to consider here. We could be trying to insert into a view a row that already exists in the underlying table, but was soft deleted. We can write the following trigger to handle this situation.
```SQL
CREATE TRIGGER "insert_when_exists"
INSTEAD OF INSERT ON "current_collections"
FOR EACH ROW 
WHEN NEW."accession_number" IN (
    SELECT "accession_number" FROM "collections"
)
BEGIN
    UPDATE "collections" 
    SET "deleted" = 0 
    WHERE "accession_number" = NEW."accession_number";
END;
```
- The `WHEN` keyword is used to check if the accession number of the artwork already exists in the `collections` table.
- The second situation occurs when we are trying to insert a row that does not exist in the underlying table. The following trigger handles this situation.
```SQL
CREATE TRIGGER "insert_when_new"
INSTEAD OF INSERT ON "current_collections"
FOR EACH ROW
WHEN NEW."accession_number" NOT IN (
    SELECT "accession_number" FROM "collections"
)
BEGIN
    INSERT INTO "collections" ("title", "accession_number", "acquired")
    VALUES (NEW."title", NEW."accession_number", NEW."acquired");
END;
```
# Sources
- [CS50 SQL - Lecture 4 - Viewing](https://cs50.harvard.edu/sql/2024/weeks/4/).