# You Can Do That in PostgreSQL
## Episode 1: The possibilities are endless 

By Mark Burggraf, Supabase Developer

### PostgreSQL is more than just a database
What makes Postgres special is its extensibility. Most Postgres extensions are written in C, giving them low-level access to OS and database processes, so there's nothing you can't do with an extension.  But you don't need to know C to take advantage of this power. Extensions have already been written that allow you to write database functions in your favorite procedural language.

Don't like writing functions in SQL?  Try [PLPGSQL](https://www.postgresql.org/docs/9.6/plpgsql.html).  Too obscure for you?  

Try writing your functions in one of these:

- Javascript [PLV8](https://plv8.github.io/)  
- Java [PL/Java](https://github.com/tada/pljava)
- Lua [PL/Lua](https://github.com/pllua)
- R [PL/R](http://www.joeconway.com/plr.html)
- PHP [PL/PHP](https://public.commandprompt.com/projects/plphp)
- Python [PL/Python](https://www.postgresql.org/docs/10/plpython.html)
- Ruby [Pl/Ruby](https://github.com/knu/postgresql-plruby)
- Scheme [PL/Scheme](https://github.com/vy/plscheme)
- Shell [PL/sh](https://github.com/petere/plsh)

People sure write Postgres functions in a lot of different languages!

### Open up a whole new world

Need to communicate with an external API over HTTP or HTTPS?  Try [PGSQL-HTTP](https://github.com/pramsey/pgsql-http).  This HTTP extension handles everything you need: headers, authentication including OAuth, get, post, put, patch, delete, head, etc. Need async http calls?  There's [pg_net](https://github.com/supabase/pg_net). 

HTTP / API access is just one of the more powerful extensions for Postgres.  In the coming episodes we'll look at other extensions that let you do all sorts of creative things like talking to other databases directly from inside of your SQL statements (other Postgres databases, MySQL, Oracle, SQL Server, SQLite, and many more.) 

We'll look at handling geographic data (need to find the 20 closest store locations to any given point on the globe in a few milliseconds?  No problem!) How about automatically scheduling processes (pg_cron), or sending emails or SMS text messages directly from your database?  Extensions make everything possible with Postgres.

### Look Ma!  No Middleware!
Do you really need serverless functions?  Do you real need middleware?  Maybe.  But maybe not.

#### PostgreSQL Replacing Middleware: Pros and Cons
First, let's look at the pros of using Postgres to handle those functions you'd normally reserve for a middle tier running on, say, Ubuntu Server running on a VM at a host, with Node JS, Express and all that stacky goodness.

#### Pros

**Less/Smaller Infrastructure**

- Fewer servers to buy, rent, or maintain
- More efficient routing (less hops -- (**app** to **db** back to **app**) vs. (**app** to **middleware** to **db** to **middleware** back to **app**)
- Better security (no need to pass a security context to a third tier)
- Faster to deploy (replicating your database is essentially also replicating your functions automatically)
- Easier to back up (since your functions are already in your database, backing up and restoring your database also backs up and restores all your functions)

#### Cons

**Missing Functionality**

Some things just aren't going to be practical inside of Postgres (yet, that is until somebody creates an efficient, working extension specifically for them.). Some examples:

- video file streaming or file conversion
- PhantomJS applications that require a complete scriptable headless browser
- Server-based PDF generation (PDFs can be created on the client, but if you want to throw some HTML at a process and have it spit out a PDF, you're better off with a server process for that at this point in time)
- Image manipulation (again, the client could probably handle this better and it would be much more scalable, but if you already have a good library for doing image manipulation on the server, you should probably be using it)

### Can vs. Should: Your Use Case Makes a Difference
Just because you can doesn't mean you should.  The point of this article series is to demonstrate some of the lesser-known interesting powers of PostgreSQL and to expand your mind to include the idea of using it to solve more than just your database issues.

Depending on your use case, PostgreSQL could be all you need on the backend.  Ever.  How well will it scale?  Will it be easy to debug and maintain?  Can I easily port it to a new instance or move it to a new server?  Will upgrades and updates cause problems down the line?

Many times these questions are moot.  Is this a small management project that'll only be used by a few people internally?  What's the projected lifespan of this project?  Am I already really comfortable working with PostgreSQL?  Either way, you should consider the purpose of your project, your scope, and future needs before you decide whether to put any of these techniques into production.

In upcoming episodes, we'll look at different application challenges that'll give you an opportunity to learn that **"You Can Do That in PostgreSQL!"**


