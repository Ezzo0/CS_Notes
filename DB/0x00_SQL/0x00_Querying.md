# Explanation
- In today’s information age, we can store our tables using software like Google Sheets instead of paper or stone tablets. However, in this course we will talk about databases and not spreadsheets.
- Three reasons to move beyond spreadsheets to databases are
    - **Scale**: Databases can store not just items numbering to tens of thousands but even millions and billions.
    - **Update Capacity**: Databases are able to handle multiple updates of data in a second.
    - **Speed**: Databases allow faster look-up of information. This is because databases provide us with access to different algorithms to retrieve information. In contrast, spreadsheets that merely allow the use of `Ctrl+F` to go through hits one at a time.
##### What is a Database?
- A database is a way of organizing data such that you can perform four operations on it
    - create
    - read
    - update
    - delete
- A database management system (DBMS) is a way to interact with a database using a graphical interface or textual language.
- Examples of DBMS: MySQL, Oracle, PostgreSQL, SQLite, Microsoft Access, MongoDB etc.
##### SQL
- SQL stands for Structured Query Language. It is a language used to interact with databases, via which you can create, read, update, and delete data in a database.
- Most DBMS support some subset of the SQL language. So for SQLite, for example, we’re using a subset of SQL that is supported by SQLite. 
- If we wanted to port our code to a different system like MySQL, it is likely we would have to change some of the syntax.
##### SELECT
- What data is actually in our database? To answer this, we will use our first SQL keyword, `SELECT`, which allows us to select some (or all) rows from a table inside the database.
- This selects all the rows from the table called `longlist`: `SELECT * FROM "longlist";`.
- The output we get contains all the columns of all the rows in this table, which is a lot of data. We can simplify it by selecting a particular column, say the title, from the table. Let’s try: `SELECT "title" FROM "longlist";`.
- Now, we see a list of the titles in this table. But what if we want to see titles and authors in our search results? For this, we run: `SELECT "title", "author" FROM longlist;`.
- It is good practice to use double quotes around table and column names, which are called SQL identifiers. SQL also has strings and we use single quotes around strings to differentiate them from identifiers.
- SQL keywords can be written in small letters, but they are written in capital letters because this is especially useful in improving the readability of longer queries. Table and column names are in lowercase.
##### LIMIT
- If a database had millions of rows, it might not make sense to select all of its rows. Instead, we might want to merely take a peek at the data it contains. 
- We use the SQL keyword `LIMIT` to specify the number of rows in the query output.
- This query gives us the first 10 titles in the database: 
```SQL
SELECT "title" 
FROM "longlist" 
LIMIT 10;
```
- The titles are ordered the same way in the output of this query as they are in the database.
##### WHERE
- The keyword `WHERE` is used to select rows based on a condition; it will output the rows for which the specified condition is true.
```SQL
SELECT "title", "author" 
FROM "longlist" 
WHERE "year" = 2023;
```
- This gives us the titles and authors for the books longlisted in 2023. Note that `2023` is not in quotes because it is an integer, not a string or identifier.
- The operators that can be used to specify conditions in SQL are `=` (“equal to”), `!=` (“not equal to”) and `<>` (also “not equal to”).
- To select the books that are not hardcovers, we can run the query
```SQL
SELECT "title", "format" 
FROM "longlist" 
WHERE "format" != 'hardcover';
```
- Yet another way to get the same results is to use the SQL keyword `NOT`. The modified query would be
```SQL
SELECT "title", "format" 
FROM "longlist" 
WHERE NOT "format" = 'hardcover';
```
- To combine conditions, we can use the SQL keywords `AND` and `OR`. We can also use parentheses to indicate how to combine the conditions in a compound conditional statement.
```SQL
SELECT "title", "format" 
FROM "longlist" 
WHERE ("year" = 2022 OR "year" = 2023) AND "format" != 'hardcover';
```
##### LIKE
- This keyword is used to select data that roughly matches the specified string. For example, `LIKE` could be used to select books that have a certain word or phrase in their title.
- `LIKE` is combined with the operators `%` (matches any characters around a given string) and `_` (matches a single character).
- To select the books with the word “love” in their titles, we can run
```SQL
SELECT "title"
FROM "longlist"
WHERE "title" LIKE '%love%';
```
- Given that there is a book in the table whose name is either “Pyre” or “Pire”, we can select it by running
```SQL
SELECT "title" 
FROM "longlist" 
WHERE "title" LIKE 'P_re';
```
- In SQLite, comparison of strings with `LIKE` is by default case-_insensitive_, whereas comparison of strings with `=` is case-sensitive. (Note that, in other DBMS’s, the configuration of your database can change this!)
##### Ranges
- We can also use the operators `<`, `>`, `<=` and `>=` in our conditions to match a range of values. For example, to select all the books longlisted between the years 2019 and 2022 (inclusive), we can run
```SQL
SELECT "title", "author" 
FROM "longlist" 
WHERE "year" >= 2019 AND "year" <= 2022;
```
- Another way to get the same results is using the keywords `BETWEEN` and `AND` to specify inclusive ranges. We can run
```SQL
SELECT "title", "author" 
FROM "longlist" 
WHERE "year" BETWEEN 2019 AND 2022;
```
- Ranges keywords can be used with date that is stored as string like the following: `YY-MM-DD`, or use `strftime('%Y', column_name) = 'value'`.
##### ORDER BY
- The `ORDER BY` keyword allows us to organize the returned rows in some specified order.
- The use of the SQL keyword `DESC` to specify the descending order. `ASC` can be used to explicitly specify ascending order.
- To select the top 10 books by rating and also include number of votes as a tie-break, we can run
```SQL
SELECT "title", "rating", "votes" 
FROM "longlist"
ORDER BY "rating" DESC, "votes" DESC 
LIMIT 10;
```
##### Aggregate Functions
- `COUNT`, `AVG`, `MIN`, `MAX`, and `SUM` are called aggregate functions and allow us to perform the corresponding operations over multiple rows of data. 
- By their very nature, each of the following aggregate functions will return only a single output—the aggregated value.
- To find the average rating of all books in the database, round the average rating to 2 decimal points, and rename the column in which the results are displayed
```SQL
SELECT ROUND(AVG("rating"), 2) AS "average rating" 
FROM "longlist";
```
- To select the maximum rating in the database: 
```SQL
SELECT MAX("rating")
FROM "longlist";
```
- To select the minimum rating in the database:
```SQL
SELECT MIN("rating") 
FROM "longlist";
```
- To count the total number of votes in the database
```SQL
SELECT SUM("votes") 
FROM "longlist";
```
- To count the number of translators in our database
```SQL
SELECT COUNT("translator") 
FROM "longlist";
```
- Note that, the `COUNT` function does not count `NULL` values.
- However, this may include duplicates. Another SQL keyword, `DISTINCT`, can be used to ensure that only distinct values are counted.
```SQL
SELECT COUNT(DISTINCT "publisher") 
FROM "longlist";
```
# Sources
- [CS50 SQL - Lecture 0 - Querying](https://cs50.harvard.edu/sql/2024/weeks/0/).