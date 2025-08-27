# Explanation
- Databases can have multiple tables. We will now see that database has many different tables inside it.
- We can use the following SQLite command to see all the tables in our database: `.tables`.
- This command returns the names of the tables in database.
- These tables have some relationships between them, and hence we call the database a **relational database**. Consider our first example. ![[author_book_example.jpg|600]]
- Just looking at these two columns, how can we tell who wrote which book? Even if we assume that every book is lined up next to its author, just looking at the `authors` table would give us no information about the books written by that author.
- Some possible ways to organize books and authors are:
	- **The honor system**: the first row in the `authors` table will always correspond to the first row in the `books` table. The problem with this system is that one may make a mistake (add a book but forget to add its corresponding author, or vice versa). Also, an author may have written more than one book or a book may be co-written by multiple authors.
	- **Going back to a one-table approach**: This approach could result in redundancy (duplication of data) if one author writes multiple books or if a book is co-written by multiple authors.
	 ![[one-table approach.jpg|600]]
- After considering these ideas, it seems like having two different tables is the most efficient approach. Let us look at some different ways in which tables can be related to each other in relational databases.
	- Consider this case, where each author writes only one book and each book is written by one author. This is called a **one-to-one relationship**.
	- On the other hand, if an author can write multiple books, the relationship is a **one-to-many relationship**.
	- Another situation where not only can one author write multiple books, but books can also be co-written by multiple authors. This is a **many-to-many relationship**.
##### Entity Relationship Diagrams
- It is possible to visualize the above relationships using an entity relationship (ER) diagram.
	 ![[ER.PNG|600]]
- Each table is an entity in our database. The relationships between the tables, or entities, are represented by the _verbs_ that mark the lines connecting entities.
- Each line is this diagram is in crow’s foot notation.
	- The first line with a circle looks like a 0 marked on the line. This line indicates that there are no relations.
	- The second line with a perpendicular line looks like a 1 marked on the line. An entity with this arrow has to have at least one row that relates to it in the other table.
	- The third line, which looks like a crow’s foot, has many branches. This line means that the entity is related to many rows from another table.
	 ![[Line_er_diagram.jpg|500]]
- On observing the lines connecting the Book and Translator entities, we can say that books don’t _need_ to have a translator. They could have zero to many translators. However, a translator in the database translates at least one book, and possibly many.
##### Keys
- Keys help relate tables in [[0x00_Querying#SQL|SQL]]. There are two types of keys: ^211dc1
	- **_Primary Keys_**
		- In the case of books, every book has a unique identifier called an ISBN. In other words, if you search for a book by its ISBN, only one book will be found. 
		- In database terms, the ISBN is a primary key — an identifier that is unique for every item in a table.
		 ![[primary_key.jpg|600]]
		- Inspired by this idea of an ISBN, we can imagine assigning unique IDs to our publishers, authors and translators. 
		- Each of these IDs would be the **primary key of the table it belongs to**.
	- **_Foreign Keys_** ^9023b9
		- A foreign key is a **primary key** taken from a **different table**. By referencing the primary key of a different table, it helps **relate** the tables by forming a **link** between them.
		 ![[Foreign_Keys.jpg|600]]
		- Notice how the primary key of the `books` table is now a column in the `ratings` table. This helps form a one-to-many relationship between the two tables — a book with a title (found in the `books` table) can have multiple ratings (found in the `ratings` table).
		-  To implement the many-to-many relationship between the `authors` and `books` entities, a table called `authored` maps the primary key of `books` to the primary key of `authors`.
		 ![[many_to_many.jpg|600]]
##### Subqueries
- A subquery is a query **inside** another query. These are also called **nested queries**.
- To find out the books published by Fitzcarraldo Editions, we would need two queries — one to find out the `publisher_id` of Fitzcarraldo Editions from the `publishers` table and the second, to use this `publisher_id` to find all the books published by Fitzcarraldo Editions. These two queries can be combined into one using the idea of a subquery.
```SQL
SELECT "title" FROM "books"
WHERE "publisher_id" = (
    SELECT "id"
    FROM "publishers"
    WHERE "publisher" = 'Fitzcarraldo Editions'
);
```
- The subquery is in parentheses. The query that is furthest inside parantheses will be run first, followed by outer queries.
- The inner query is indented. This is done as per style conventions for subqueries, to increase readability.
- To find the author(s) who wrote the book Flights, three tables would need to be queried: `books`, `authors` and `authored`.
```SQL
SELECT "name" FROM "authors"
WHERE "id" = (
    SELECT "author_id" FROM "authored"
    WHERE "book_id" = (
      SELECT "id" FROM "books"
      WHERE "title" = 'Flights'
    )
);
```
##### IN
- This keyword is used to check whether the desired value is **in** a given list or set of values.
- The relationship between authors and books is many-to-many. This means that it is possible a given author has written more than one book. 
- To find the names of all books in the database written by Fernanda Melchor, we would use the `IN` keyword as follows.
```SQL
SELECT "title" FROM "books"
WHERE "id" IN (
    SELECT "book_id" FROM "authored"
    WHERE "author_id" = (
        SELECT "id" FROM "authors"
        WHERE "name" = 'Fernanda Melchor'
    )
);
```
- If the value of an inner query is not found, the inner query would return nothing, prompting the outer query to also return nothing.
##### JOIN
- This keyword allows us to combine two or more tables together. To understand how `JOIN` works, consider a database of sea lions and their migration patterns. Here is a snapshot of the database.
 ![[database of sea lions.jpg]]
- To find out how far the sea lion Spot travelled, or answer similar questions about each sea lion, we could use nested queries. Alternately, we could join the tables `sea lions` and `migrations` together such that each sea lion also has its corresponding information as an extension of the same row.
- We can join the tables on the sea lion ID (the common factor between the two tables) to ensure that the correct rows are lined up against each other.
```SQL
SELECT * FROM "sea_lions"
JOIN "migrations" ON "migrations"."id" = "sea_lions"."id";
```
- The `ON` keyword is used to specify which values match between the tables being joined. It is not possible to join tables without matching values.
- If there are any IDs in one table not present in the other, this row will **not be present** in the joined table. This kind of join is called an **INNER JOIN**.
- Some other ways of joining tables that allow us to retain certain unmatched IDs are **LEFT JOIN**, **RIGHT JOIN** and **FULL JOIN**. Each of these is a kind of **OUTER JOIN**.
- A LEFT JOIN prioritizes the data in the left (or first) table.
```SQL
SELECT * FROM "sea_lions"
LEFT JOIN "migrations" ON "migrations"."id" = "sea_lions"."id";
```
- This query would retain all sea lion data from the `sea_lions` table — the left one. Some rows in the joined table could be partially blank. 
- This would happen if the right table didn’t have data for a particular ID. OUTER JOIN could lead to empty or `NULL` values in the joined table.
- Similarly, a RIGHT JOIN retains all the rows from the right (or second) table. A FULL JOIN allows us to see the entirety of all tables.
- Both tables in the sea lions database have the column `id`. Since the value on which we are joining the tables has the same column name in both tables, we can actually omit the `ON` section of the query while joining.
```SQL
SELECT * FROM "sea_lions"
NATURAL JOIN "migrations";
```
- This join works similarly to an INNER JOIN.
##### Sets
- In our database of books, we have authors and translators. A person could be either an author or a translator. If the two sets have an intersection, it is also a possible that a person could be both an author and a translator of books. We can use the `INTERSECT` operator to find this set.
```SQL
SELECT "name" FROM "translators"
INTERSECT
SELECT "name" FROM "authors";
```
- If a person is either an author or a translator, or both, they belong to the union of the two sets. In other words, this set is formed by combining the author and translator sets.
```SQL
SELECT "name" FROM "translators"
UNION
SELECT "name" FROM "authors";
```
- Notice that every author and every translator is included in this result set, but only once.
- Everyone who is an author and _only_ an author is included in the following set. The `EXCEPT` keyword can be used to find such a set. In other words, the set of translators is subtracted from the set of authors to form this one.
```SQL
SELECT "name" FROM "authors"
EXCEPT
SELECT "name" FROM "translators";
```
- How can we find this set of people who are either authors or translators but not both?
- These operators could be useful to answer many different questions. For example, we can find the books that Sophie Hughes and Margaret Jull Costa have translated together.
```SQL
SELECT "book_id" FROM "translated"
WHERE "translator_id" = (
    SELECT "id" from "translators"
    WHERE "name" = 'Sophie Hughes'
)
INTERSECT
SELECT "book_id" FROM "translated"
WHERE "translator_id" = (
    SELECT "id" from "translators"
    WHERE "name" = 'Margaret Jull Costa'
);
```
- Each of the nested queries here finds the IDs of the books for one translator. The `INTERSECT` keyword is used to intersect the resulting sets and give us the books they have collaborated on.
##### Groups
- Consider the `ratings` table. For each book, we want to find the average rating of the book. To do this, we would first need to group ratings together by book and then average the ratings out for each book (each group).
```SQL
SELECT "book_id", AVG("rating") AS "average rating"
FROM "ratings"
GROUP BY "book_id";
```
- In this query, the `GROUP BY` keyword was used to create groups for each book and then collapse the ratings of the group into an average rating.
- Now, we only want to see the books that are well-rated, with an average rating of over 4.
```SQL
SELECT "book_id", ROUND(AVG("rating"), 2) AS "average rating"
FROM "ratings"
GROUP BY "book_id"
HAVING "average rating" > 4.0;
```
- Note that the `HAVING` keyword is used here to specify a condition for the groups, instead of `WHERE` (which can only be used to specify conditions for individual rows).
# Sources
- [CS50 SQL - Lecture 1 - Relating](https://cs50.harvard.edu/sql/2024/weeks/1/).