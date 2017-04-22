---
layout: post
section-type: post
title: Stealing passwords with SQL Injections
category: tech
tags: [ 'redteam', 'kali', 'dvwa' ]
---
In this post we'll use [DVWA]({% post_url 2017-04-02-dvwa-kali %}) as usually, in order to access its user passwords using SQL injections.
SQL injections are similar to [XSS]({% post_url 2017-04-15-dvwa-xss %}) and [command injection]({% post_url 2017-04-18-command-injection %}) attacks, in the way that user input is not sanitized and it ends up manipulating the vulnerable application.

Let's navigate to the *SQL Injection* section of DVWA and look at its source:

![sqli](/img/posts/sqli/sqli-source.png)

As you can see the web app queries the users table, for a given ID value, which is provided by the user and is not validated.
Then the results are being iterated and returned to the client.

First let's use the web app to query for user the id 1:

![sqli](/img/posts/sqli/sqli.png)

Let's inject a condition that will always be true, in order to read all the tuples from the users table:

<pre><code data-trim class="bash">
' OR ''='
</code></pre>

![sqli](/img/posts/sqli/sqli-0.png)

That's a start, but the data we got back are not that interesting.
We are able enumerate manually the id and get the same results back, so this is not data that we're not authorized to access.
How can we access other columns other than the first and the last name?

Let's try to append an SQL query after the web app's query:

<pre><code data-trim class="bash">
'; SELECT password FROM users WHERE user_id = '1
</code></pre>

Well, that didn't quite work:

> You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'SELECT password FROM users WHERE user_id = '1'' at line 1

The reason that didn't work is because *mysqli_query* doesn't allow the execution of more than one query in a single call.

The SQL UNION operator will come handy in here.
The UNION operator expects two result table operands, with the same number of columns and appends the second table at the end of the first one.
So, let's close the web app's query, and then do a UNION on our injected query, which will be asking for the passwords column of the users table:

<pre><code data-trim class="bash">
' UNION SELECT user_id, password FROM users #
</code></pre>

Note that we are commenting out the web app's SQL query that comes after our injected one, using the '#'.

![sqli-passwords](/img/posts/sqli/sqli-passwords.png)

Success!

As you can see the passwords are hashed, but we'll crack them in a future post :smile:
