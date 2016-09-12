---
title: Using Database
permalink: /en/apv/walkthrough/database/
---

* TOC
{:toc}

In previous chapters, you learned how to create a static HTML page, then how to 
add dynamic content generated in a PHP script to that page. You then learned how
to organized the PHP and HTML code using templates, so that it doesn't become a
complete mess.

So far it's probably still a bit boring, because the data that the application displays
are still somewhat hardcoded in the PHP script. We would like to make the application
interactive, so that a user can permanently store and retrieve some arbitrary data.
The answer to this is that we need a [database server](todo).

In this walkthrough I will be using a [relational database](todo), which means that
all of the data are organized into [tables](todo). Specifically, I will be using
[PostgreSQL database](todo), which again is a rather arbitrary choice, because for
what we need, any relation database could be used.

## Getting started
Before you get started, you need a [database server](todo) and credentials to it.
You also need an access to some kind of administration interface, preferably
a graphical one, so that you don't get completely lost at the beginning.

To get you started as quickly as possible, I prepared a sample database for you.
It contains the structure of the [project](todo) and also some sample data, 
which are useful for testing your application. First you need to import the database,
then you can start playing with it using the [SQL language](todo).

### Import Database
I prepared files that you can import database, there are two files, first one contains
definition of what tables do exist in the database and second one contains sample data.
Download both files (use 'Save Link As') and import both files using the procedure
outlined below. 

- [Structure of database](/en/apv/walkthrough/database/pgsql-structure.sql) 
- [Sample Data](/en/apv/walkthrough/database/pgsql-data.sql) 

To import the file using the [Adminer](todo) client. Click on **Import**, select the 
file you downloaded, and click **Execute**. Repeat the process for both files:

![Screenshot -- Import data](/en/apv/walkthrough/database/adminer-import-1.png)   

Once you have the both the structure of the database and the data imported, you
should see, content in the Adminer

![Screenshot -- Imported data](/en/apv/walkthrough/database/adminer-import-2.png)

### SQL Language
SQL is a language which is used for communicating with most of the widely used database servers.
If you have been following this Walkthrough from the [beginning](todo), then this is
fourth language you have learned so far (after HTML, PHP, Latte Macro Language).

SQL language has only four everyday-use commands (there are dozens of others, which are
rarely used and you can always look them up):

- INSERT for inserting data into database
- UPDATE for modifying data already in the database 
- DELETE for removing data from the database
- SELECT for obtaining data from the database

The first three commands are really simple, as you will see in the following exercise. 
The SELECT command is incredibly complicated and it usually takes years to learn all the 
intricacies connected with it (so I'll start with some simple uses and leave the 
more difficult ones to later).

SQL language is a programming language, but instead of writing lengthy pieces of programming 
code as in PHP, we only write queries. An SQL query is self-contained piece of code -- **statement**
-- instructing the database server to do something (e.g. delete some data). The query may 
contain other statements inside.

When working with database, it is really important to know the structure of the database.
It is shown in the following graphic, which shows each table, the columns in the table and
their data types. It also shows relationships between tables (which we won't use much in this
chapter). The design of database structure is called **Database Schema**. Keep it handy.

![Database schema](/en/apv/schema.svg)

As we are working with [relational database system](todo), all data are stored in tables.
Each table has **columns** (with some name and datatype -- as defined in the database schema)
and **rows** which contain the actual data. Most operations in SQL work on full rows -- sometimes
called **records**.

{: .note}
In SQL, the [keywords](todo-zakladniclanekoprogramovani) are usually written ALL CAPS, this 
is just a convention to improve readability. It has no effect on the query itself. SQL statements
are delimited by semicolon `;`. Because in most cases, only a single SQL statement is used, the 
semicolon at the and of it may safely omitted (it is a delimiter, not terminator).  

### INSERT
The general syntax of the INSERT command is:

{% highlight sql %}
INSERT INTO *table_name* (*list_of_columns*) VALUES (*list_of_values*)
{% endhighlight %}

The INSERT statement inserts **rows** into a table. 
When you look at the schema, you should see that the `location` table has columns:
`id_location`, `city`, `street_name`, `street_number`, `zip`, `country`, `location`, `latitude`, `longitude`.
The column types of `latitude` and `longitude` is `numeric` which is a decimal number (GPS coordinates) and
the type of `id_location` and `street_number` is `integer`. All other columns have type `character varying` which 
is another name for `string`. All of the columns except `id_location` allow [`NULL`s](todo) and
the column `id_location` has [default value](todo), which means that the all of the columns are optional.
Columns with [automatically generated IDs](todo) -- such as `id_location` in this table -- are 
almost never inserted.

Let's insert some data into the `location` table:

{% highlight sql %}
INSERT INTO location (name, city, street_name) VALUES ('La Tour Eiffel', 'Paris', 'Avenue Gustave Eiffel')
{% endhighlight %}

The order of inserted values must match the order of column names, this (obviously?) means that 
there must be same number of items in the column list and value list.  
Try to run the query yourself in Adminer:

![Screenshot -- Run SQL query](/en/apv/walkthrough/database/run-query.png)

Important: Unlike in HTML and PHP, in SQL, strings must always be enclosed in single 
quotes `'`. Double quotes are **not allowed**. 

You should see the result

    Query executed OK, 1 row affected.

Click on **select** next to the `location` table to verify, that the row really got inserted into the table.
That's -- once you know the structure of the destination table, writing an SQL INSERT statement is easy.
Notice that there are two keywords used at the beginning of the query `INSERT INTO`. If you have some 
programming background, this may look unusual to you. It is supposed to improve the readability ot the
SQL query. In SQL it is quite common, so you better get used to it.

THe list of the columns is not required, so the above query may also be shortened to:

{% highlight sql %}
INSERT INTO location VALUES (DEFAULT, 'Paris', 'Avenue Gustave Eiffel', NULL, NULL, NULL, 'La Tour Eiffel')
{% endhighlight %}

I highly discourage you from using this form of INSERT query in your application. It breaks as 
soon as you make some modifications to the table. Also just from reading it, it is very hard 
to understand what gets inserted where. It is better to be **explicit and verbose**.

#### DEFAULT and NULL
They keyword `DEFAULT` means that a default value of the column should be used (in the above case,
it is a [sequence](todo) value -- `nextval('location_id_location_seq'::regclass)`). The keyword
`NULL` means that no value will be inserted into that column. 

If you don't list a column in the 
INSERT query, then it is the same as if you listed it with the `DEFAULT` keyword. 
Then it depends on the definition of the column:

- If the column has a default value, than that value is inserted.
- If the column has not a default value then:
    - If the column allows NULLs, no value is inserted (NULL).
    - If the column requires a value (defined as `NOT NULL`), an error is raised -- a required value 
    for the table column was not provided in the INSERT query

#### Working with Dates
Although dates are printed as string, the database server stores them in a 
[timestamp format](todo). When inserting a date, you must make sure 
that the server understands it correctly by either:

- Supplying the date in the format expected by the server (usually `YYYY-MM-DD`).
- Explicitly converting the date with conversion function.

The first option depends on the configuration of the database server, but
it is fairly common, that this query works:

{% highlight sql %}
INSERT INTO person (first_name, last_name, nickname, birth_day)
VALUES ('John', 'Doe', 'Johnnie', '2010-12-24')
{% endhighlight %}

Using a conversion function is safer, but slightly more complicated. There are
a lot of differences between various database servers. However, the general principle 
is that you supply date in an arbitrary format and a description of that format:

{% highlight sql %}
INSERT INTO person (first_name, last_name, nickname, birth_day)
VALUES ('John', 'Doe', 'Johnnie', TO_TIMESTAMP('24.12.2010', 'DD.MM.YYYY'),)
{% endhighlight %}

In the above query, I used the [TO_TIMESTAMP function](todo) of the PostgreSQL
database server. The first argument to the function is the date (in any format).
The second argument to that function is the description of the format 
with a [formatting string](todo). This way you tell the server that the date 
starts with two digits representing a date and followed by a dot. 

{. :note}
The date conversion functions are specific to each database server. While they share
the same principle, they may have different names and parameters. Always consult the
manual of the database server you are using (look for section 'Date conversion functions').

### UPDATE
The general syntax of the UPDATE command is:

{% highlight sql %}
UPDATE *table_name* SET *list_of_modifications* WHERE *search_condition*
{% endhighlight %}

The UPDATE command updates existing values in existing *rows* in the table. The *list_of_modifications* 
is a list of assignments to be made. The *search_condition* is an expression which is evaluated for 
each row and should yield boolean (true/false). If the *search_condition* is true, the corresponding row
is updated, otherwise it is skipped. 

Let's update a row in the `location` table:

{% highlight sql %}
UPDATE location SET name = 'Arc de Triomphe', street_name = 'Place Charles de Gaulle' 
    WHERE name = 'La Tour Eiffel'
{% endhighlight %}

{: .note}
Note that the same character `=` is used for both comparison (`WHERE` clause) and 
assignment (`SET` clause). In SQL the actual function of equal sign is determined by 
context. There is no `==` in SQL. 

Search condition can be composed of multiple statements joined using boolean operators `AND`, `OR` and `NOT`.
Notice that you can reference a column in both the search condition and in assignment (`name` in the 
above example). Also the new value of the column may be an expression, so it's valid to write e.g.:
`... SET age = age + 1 ...`. 

#### Search condition
Important: the `WHERE` condition is not required, if you forget it, 
**all rows in the table will be updated**. Also notice that `NULL`s need to be [treated specially](todo). 
Typically you use [key columns](todo) in the search condition to update a single record. Be careful
about this -- if you are using a [compound key](todo), then all columns of that key must be used
in the condition. Consider this query:

{% highlight sql %}
UPDATE person SET height = '135' WHERE first_name = 'John' and last_name = 'Doe'
{% endhighlight %}

There is a compound key on columns `first_name`, `last_name`, `nickname` defined on
the `persons` table (`UNIQUE (first_name, last_name, nickname)`). This guarantees that
the combination of first name, last name and nickname of a person is unique 
(in your database that is). However the combination of `first_name` and `last_name`
is **not guaranteed** to be unique and therefore the query **may** update more than 
one person and therefore it is **wrong**. There are two correct options: 

{% highlight sql %}
UPDATE person SET height = '135' 
WHERE first_name = 'John' and last_name = 'Doe' AND nickname = 'Johnnie'
{% endhighlight %}

{% highlight sql %}
UPDATE person SET height = '135' WHERE id_person='42'
{% endhighlight %}

{: .note}
Either of the above two solutions is fine, but any other solutions are wrong. When
you are modifying data in database, you must **make sure** that there is no possibility
of changing unwanted rows.

### DELETE
The general syntax of the DELETE command is:

{% highlight sql %}
DELETE FROM *table_name* WHERE *search_condition*
{% endhighlight %}

`DELETE` command removes **entire rows** from the table. Same rules apply for the *search_condition* as in
the `UPDATE` statement. Including that if you forget the `WHERE` statement, then 
**all rows of the table will be deleted!**.

Let's delete the row, we just inserted:

{% highlight sql %}
DELETE FROM location WHERE id_location > 50
{% endhighlight %}
 
### SELECT
The general syntax of a *simple* SELECT command is:

{% highlight sql %}
SELECT *column_list* FROM *table_name* WHERE *search_condition* ORDER BY *list_of_orderings* 
{% endhighlight %}

The `SELECT` command returns rows of an existing table (or tables) and returns 
them in form of a table. The *column_list* is either a list of column names or
asterisk `*` to list all columns of the table.  
The same rules apply to the *search_condition* as in previous commands, including that
if you omit the `WHERE` statement altogether, all rows of a table will be returned. 

{% highlight sql %}
SELECT * FROM location WHERE id_location > 40 
{% endhighlight %}

There are some [useful features](todo) which you can use in the `WHERE` clause. You should
definitely be aware of the existence of `LIKE` and `IN` operators.

`LIKE` operator is useful for selecting string by a partial match. Use the percent sign `%` as
a placeholder for any number of characters: 

{% highlight sql %}
SELECT * FROM location WHERE city LIKE 'B%' 
{% endhighlight %}

The above example selects all locations where `city` starts with letter `B`. 

`IN` operator is useful for matching against set of values. The following will select
all location whose city is one of those mentioned in the set (defined in the parenthesis):

{% highlight sql %}
SELECT * FROM location WHERE city IN ('Praha', 'Paris', 'Bremen') 
{% endhighlight %}

The set can be either enumerated as in the above example or it can be defined by another
[`SELECT` query](todo). Ordering of the results is possible by multiple criteria:

{% highlight sql %}
SELECT * FROM location ORDER BY city 
{% endhighlight %}

Compare the result of the above query with the result of the below query. Look at multiple
rows with the same city - e.g Brno

{% highlight sql %}
SELECT * FROM location ORDER BY city, street_name 
{% endhighlight %}

Each column ordering can be further modified by direction keyword either `ASC` (ascending sorting -- A-Z) 
or `DESC` (descending sorting -- Z-A). If no direction is specified, then `ASC` is assumed.
Note that the direction must be specified for each column individually, so the 
query 

{% highlight sql %}
SELECT * FROM location ORDER BY city, street_name DESC 
{% endhighlight %}

Will not fully reverse the table, but the following query will:

{% highlight sql %}
SELECT * FROM location ORDER BY city DESC, street_name DESC 
{% endhighlight %}


## Task -- Insert
Insert a person named *Karl Oshiro*, with nickname *hiromi*, gender = 1 and height = 180.

{: .solution}
{% highlight sql %}
INSERT INTO person (first_name, last_name, nickname, gender, height) VALUES 
    ('Karl', 'Oshiro', 'hiromi', 1, 180)
{% endhighlight %}

## Task -- Select
Select id_person of the *Karl Oshiro*, you have just inserted, make sure to return only a 
single row.

{: .solution}
{% highlight sql %}
SELECT id_person FROM person 
    WHERE first_name = 'Karl' AND last_name = 'Oshiro' AND nickname = 'hiromi'
{% endhighlight %}

## Task -- Update
Change the name of the *Karl Oshiro* you just inserted to *Carl Sohiro*, make sure 
to update only a single row.

{: .solution}
{% highlight sql %}
UPDATE person SET first_name = 'Carl', last_name = 'Oshiro' WHERE 
    id_person = 49
{% endhighlight %}

{: .solution}
{% highlight sql %}
UPDATE person SET first_name = 'Carl', last_name = 'Oshiro' WHERE 
    WHERE first_name = 'Karl' AND last_name = 'Oshiro' AND nickname = 'hiromi'
{% endhighlight %}

## Task -- Remove a value
Remove the value of `height` for *Carl Sohiro*.

{: .solution}
{% highlight sql %}
Hint: you must set the value to `NULL`.
{% endhighlight %}

{: .solution}
{% highlight sql %}
UPDATE person SET height = NULL WHERE id_person = 49
{% endhighlight %}

You cannot use a `DELETE` command, because that deletes entire rows. Also you 
mustn't set the value to 0, because there is a difference between 0 height and 
unknown height.

## Task -- Combined Search condition
Find all persons whose last name begins with **C** or **K** and their height
is more than 170 centimeters.

{: .solution}
{% highlight sql %}
Hint: You'll need an `OR`, an `AND` and `LIKE`.
{% endhighlight %}

{: .solution}
{% highlight sql %}
Hint: If you have the sample data, then three rows should be returned. If you got
more, double check the results.
{% endhighlight %}

{: .solution}
{% highlight sql %}
SELECT * FROM person WHERE (last_name LIKE 'C%' OR last_name LIKE 'K%') AND height > 170
{% endhighlight %}

## Task -- Search Using a Set
Select all persons with names 'Gilda', 'Alisha' and 'Lisette' and order them so that
the tallest are shown first.

{: .solution}
{% highlight sql %}
SELECT * FROM person WHERE first_name IN ('Gilda', 'Alisha', 'Lisette') ORDER BY height ASC
{% endhighlight %}

## Task -- Search Using a Sub-select Set 
Select all persons which have associated location in country *United Kingdom* and order
them by last name. You will now need to write two `SELECT`s and connect them using `IN`
operator. 

{: .solution}
{% highlight sql %}
Hint: Start by selecting `id_location` of the locations in *United Kingdom*. You should
get 12 rows with the sample data.
{% endhighlight %}

{: .solution}
{% highlight sql %}
Hint: Write a `SELECT` statement use the previous SELECT inside parenthesis of `IN` operator.
{% endhighlight %}

{: .solution}
{% highlight sql %}
SELECT * FROM person WHERE id_location IN
(SELECT id_location FROM location WHERE country = 'United Kingdom')
ORDER BY last_name
{% endhighlight %}

## Task -- Select Everything
Just a verification -- select all rows and all columns from the persons table.

{: .solution}
{% highlight sql %}
SELECT * FROM person
{% endhighlight %}

## Summary
You should now know how to work with relational database. There is [much more to come], but
so far, you should be able to `SELECT`, `INSERT`, `UPDATE` and `DELETE` data. This is 
sufficient to create a reasonably working web application. 
You should be familiar with general syntax of SQL statements, the common key words, and 
SQL specific operators (`LIKE`, `IN`). Do not be afraid to experiment with your database
you can always delete it (and import it again) by using the SQL queries:

{% highlight sql %}
    DROP TABLE relation CASCADE;
    DROP TABLE relation_type CASCADE;
    DROP TABLE contact CASCADE;
    DROP TABLE contact_type CASCADE;
    DROP TABLE person_meeting CASCADE;
    DROP TABLE meeting CASCADE;
    DROP TABLE person CASCADE;
    DROP TABLE location CASCADE;
    DROP TYPE gender;
{% endhighlight %}

### New Concepts and Terms
- (Relational) Database
- Database Schema 
- Tables
- SELECT
- INSERT
- UPDATE
- DELETE