---
title: Using Database in PHP
permalink: /en/apv/walkthrough/backend/
---

* TOC
{:toc}

In previous chapters, you learned how to create [PHP scripts](todo), and
in the latest chapter also how to [work with database using SQL language](todo).
In this chapter, you'll learn how to work with database from within a 
PHP script. This is a very important step in connecting all the 
[technologies in the stack](todo) together.

## Getting Started
Before you start, you need to have working credentials to a database, and
you should have the [sample database](todo) imported. Also you should be 
familiar with [creating and running PHP script](todo).

To create an application which communicates with a database system, you 
always need some kind of library. Database libraries are specific to 
the application language (PHP, Java, C++) an database (PostgreSQL, MySQL, ...),
so there are hundreds of them.

For PHP, there is a very good built-in library -- [PDO (PHP Database Objects)](todo), 
which is capable of communicating with multiple databases (including PostgreSQL and MySQL).

### Connecting to database
To connect to the database, you need to write a [DSN](todo) connection string. This is 
done usually once, so it is not at all important to remember this. For PostgreSQL you should
use: 

    pgsql:host=SERVER_NAME;dbname=DATABASE_NAME

The *pgsql* is [driver name](http://php.net/manual/en/ref.pdo-pgsql.connection.php).
For the [prepared PostgreSQL server](todo), the connection string would be e.g.:

    pgsql:host=akela.mendelu.cz;dbname=xpopelka 

To create a database connection, create a new instance of the [`PDO` class](todo).
Provide the *DSN connection string*, *database username* and *password* in the constructor.

{% highlight php %}
<?php 

$db = new PDO('pgsql:host=akela.mendelu.cz;dbname=xpopelka', 'xpopelka', 'password');
{% endhighlight %}

### Selecting Data
To select data from database, use the `query` method of the `PDO` connection object.
Supply a SQL [`SELECT`](todo) query as a string to the function. The function will
return a [`PDOStatement` object](todo). The `PDOStatement` represents an SQL query and
also its result. One way to obtain the result is calling the [`fetchAll` function](todo). 

{% highlight php %}
{% include /en/apv/walkthrough/backend/select-simple.php %}
{% endhighlight %}

The `fetchAll` function returns a [two-dimensional array](todo). It returns an array
of result table (`person`) rows. Each row is an array indexed by column keys, values 
are table cells. Therefore the following code will print `first_name` of the 
second person (as ordered by `first_name`). We used the [`print_r` function](todo) to
print the complete array (it's not beautiful, but it shall be good enough at the moment).

{% highlight php %}
{% include /en/apv/walkthrough/backend/select-simple-fetch.php %}
{% endhighlight %}

### Selecting Data with Parameters
Often you need to provide dynamic values (obtained from PHP variables and/or HTML forms) to
the SQL queries. E.g. assume you need to run a query like this (where *Bill* is provided
by the end-user and stored in a PHP variable):

{% highlight sql %}
SELECT * FROM person WHERE first_name = 'Bill';
{% endhighlight %}

The solution is to use [**prepared statements**](todo). This means that you **prepare** a
SQL statement with **placeholders**, then **bind** values to the placeholders and
then **execute** the statement:

{% highlight php %}
{% include /en/apv/walkthrough/backend/select-prepared.php %}
{% endhighlight %}

In the above query, I used a placeholder name `:name` (placeholder must start with colon `:`). 
Then I bind value to it using the [`bindValue`](todo) method of the `$stmt` [`PDOStatement`](todo) 
object. Last, I (`execute`)[todo] the statement. Then the result can be printed as in
the previous example. 

{: .note}
If you are tempted to use the `$personName` variable directly within the SQL query string,
in the `query` method, don't do it! Such approach would introduce [SQL injection vulnerability](todo). 

### Inserting Data
Let's insert a new row in the `location` table. The principle remains the same as in the 
above example with prepared statement. You just need to use the [`INSERT`](todo) statement and
provide the right parameters to it: 

{% highlight php %}
{% include /en/apv/walkthrough/backend/insert-prepared.php %}
{% endhighlight %}

Note that there is no `fetchAll` call, because the `INSERT` statement does not return a table 
(or anything that useful). Because working with prepared parameters can be a little bit tricky, you can
use `$stmt->debugDumpParams();` function to print the SQL statement and actual values of parameters for
debugging purposes.  
   
{: .note}
I have named the keys in the `$location` variable the same way as the SQL placeholders (`:name`, `:city`, `:country`)
and also the same way as columns in the `location` table. This is not at all necessary, because these names
are totally unrelated. However, it reduces a lot of confusion to use consistent naming (also saves you a lot of time inventing 
new names). 

### Error Control
An important part of communicating with database is [handling errors](todo). There are 
[multiple options](todo), but the easiest way is to use [exceptions](todo). 
The following example extends the previous `INSERT` example with 
error handling.

{% highlight php %}
{% include /en/apv/walkthrough/backend/insert-error.php %}
{% endhighlight %}
 
The first important part is the line `$db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);`
which makes the database driver switch into mode in which it [*throws*](todo) an exception
whenever an error occurs in an operations.

Second, I wrapped the whole code in a `try -- catch` statement. As the name suggest, the code
inside `try -- catch` is executed normally unless an exception occurs. Whenever an exception
occurs, the rest of the `try` code is skipped and the `catch` code is executed.
In the `catch` code I catch exceptions of class [`PDOException`](todo) -- those are exceptions
thrown by the PDO database driver. The method `getMessage` of the exception object returns the
actual error message returned by the database.

The above `INSERT` statement can fail for many reasons, e.g

- database server is not available
- database credentials are wrong
- inserted values are not allowed in the table (e.g. are too long)
- there is something wrong in with the database structure (e.g. table does not exist)
- many others... 

Try to simulate some of the possible error conditions to make sure that the 
error handling is triggered correctly.

## Task -- Select Data
Select `first_name`, `last_name`, `nickname` of all persons. Order the persons by their
last name and first name (ascending). Make sure to use appropriate error handling.

{: .solution}
{% highlight php %}
{% include /en/apv/walkthrough/backend/select-sol-1.php %}
{% endhighlight %}

Notice that I used two try-catch blocks, one for connecting to the database and one for the
actual query. This will become more useful in future, when we need to distinguish between 
errors in different parts of code. In the first `catch` I used the `exit` function to
immediately terminate the execution of the script. 

## Task -- Select Data
Select `first_name`, `last_name`, `age`, `height` of all persons, whose first name or last name 
begins with **L**. Order the persons by their
height and age (descending). Make sure to use appropriate error handling. I suggest you approach 
the task in parts, first make a working SQL query, then add it to a PHP script.                      

{: .solution}
This was a little test whether you can [search for new stuff](http://bfy.tw/7HLc) --
Use the [AGE function](https://www.postgresql.org/docs/8.4/static/functions-datetime.html) in SQL.
The first person should be *Leonora Nisbet*. 

{: .solution}
{% highlight sql %}
SELECT first_name, last_name, nickname, AGE(birth_day) AS age, height 
		FROM person 
		WHERE first_name LIKE 'L%' OR last_name LIKE 'L%'
		ORDER BY height DESC, age DESC
{% endhighlight %}

{: .solution}
{% highlight php %}
{% include /en/apv/walkthrough/backend/select-sol-2.php %}
{% endhighlight %}

{: .note}
I used an [alias](todo) in the SQL query to define a name of the computed column. It is important to know
 the column name, because we need to reference it in the PHP script.      

## Task -- Print Data in HTML
A big task lies ahead of you. Print `first_name`, `last_name`, `nickname` and 
`age` rounded to years of all persons ordered by `last_name` and `first_name` (ascending).
Print the persons in a HTML table, one row each. Use layout template](todo) for the HTML page. 
Again, approach the task in steps, e.g.:

1. Make a static HTML page with some sample data (skip this if you are confident with templates).
2. Make a PHP script to print the page using templates.
3. Make the data in the script dynamic -- load them from variable, make sure the variable has same 
format as obtained from database. 
4. Write the SQL query to obtain the data you want.
5. Hook the SQL query into the PHP script.

### Step 1
Consult the [HTML guide] if you are not sure.

{: .solution}
{% highlight html %}
{% include /en/apv/walkthrough/backend/persons-static.html %}
{% endhighlight %}

### Step 2
Create a PHP script, a template and a layout template.

{: .solution}
{% highlight php %}
{% include /en/apv/walkthrough/backend/persons-dynamic-1.php %}
{% endhighlight %}

{: .solution}
{% highlight html %}
{% include /en/apv/walkthrough/backend/persons-dynamic-1.latte %}
{% endhighlight %}

{: .solution}
{% highlight html %}
{% include /en/apv/walkthrough/backend/layout.latte %}
{% endhighlight %}

### Step 3
Define the persons to be displayed as an array in the PHP script, make 
sure the array has the same form as the one [returned from database functions](todo). 

{: .solution}
{% highlight php %}
{% include /en/apv/walkthrough/backend/persons-dynamic-2.php %}
{% endhighlight %}

{: .solution}
{% highlight html %}
{% include /en/apv/walkthrough/backend/persons-dynamic-2.latte %}
{% endhighlight %}

### Step 4
Write the SQL query and test that it works. 

{: .solution}
{% highlight sql %}
SELECT first_name, last_name, nickname, date_part('years', AGE(birth_day)) AS age 
FROM person
ORDER BY last_name ASC, first_name ASC
{% endhighlight %}

### Step 5
Modify the PHP script to load the variable from database. 

{: .solution}
{% highlight php %}
SELECT first_name, last_name, nickname, date_part('years', AGE(birth_day)) AS age 
FROM person
ORDER BY last_name ASC, first_name ASC
{% endhighlight %}

No one is forcing you to take all the above steps separately or in the shown order. 
But **you must always be able to divide a complex task into simpler steps**. This
is really important -- the scripts will become only more and more complicated and there is really 
only one way to orientate in all the code and debug it. You have to split it into smaller pieces, 
write and test the pieces individually. Notice how in the above steps I changed only one thing
at a time. Some parts (like the template layout) don't need to be changed all.

## Summary
In this chapter, you learned how to use SQL queries from within a PHP script.
Non-parametric queries are quite simple (just call the `query` function). Parametric
queries are more complicated (`prepare`, `bindValue`, `execute` function calls).
Using proper error control adds further complexity to the script. However the error control
is very important, otherwise the application will [misbehave in](todo) case an error condition occurs.
Because the entire application code is now becoming a bit complex, it is really important that
you are able to separate the code into individual parts and test each part individually.

### New Concepts and Terms
- Database Driver
- PDO/PDOStatement
- Prepared Statement
- Query Parameters
- Error Control