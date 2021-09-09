# You Can Do That in PostgreSQL
## Episode 2: The Price of Tea in China 

By Mark Burggraf, Supabase Developer

### Live currency conversion for PostgreSQL
It's time to sell some tea out of the back of our Postgres Tea Shop.  But to make some real money, we need to tap into those international tea buyers.  The only problem is, they've been asking us for pricing information in all these foreign currencies, and we only have our tea prices in $USD - US Dollars.

Ideally, we should have the exchange rates in the database, and in a perfect world, those prices would be automatically updated daily so they're always current.  Can that be done?  **You Can Do That in PostgreSQL!**

### The tea data
```sql
create table tea (id TEXT PRIMARY KEY, name TEXT, price NUMERIC);
insert into tea (id, name, price) values 
   ('earl', 'Earl Gray', 13.99),
   ('chai', 'Chai Masala', 14.49),
   ('breakfast', 'Breakfast Blend', 11.49),
   ('hyson', 'Hyson Green Tea', 16.79),
   ('black-lemon', 'Black Lemon Tea', 12.29),
   ('peach', 'Peach Tea', 14.99);
```

| id          | name            | price |
| ----------- | --------------- | ----- |
| earl        | Earl Gray       | 13.99 |
| chai        | Chai Masala     | 14.49 |
| breakfast   | Breakfast Blend | 11.49 |
| hyson       | Hyson Green Tea | 16.79 |
| black-lemon | Black Lemon Tea | 12.29 |
| peach       | Peach Tea       | 14.99 |

### How do we get international pricing?
We need a list of currencies and the current exchange rate for each one.  A quick Google search turns up [Free Currency Rates API](https://github.com/fawazahmed0/currency-api), which is free, with no authetication necessary, no restrictions, and hosted on GitHub.  Just what we need.

You might ask -- what if I choose an API that requires authentication?  That's no problem, of course, since the [PostgreSQL HTTP Client](https://github.com/pramsey/pgsql-http) handles headers, authentication, and anything else we might need.

### Get the list of currencies and exchange rates
First, let's get the currency exchange rates for today using the [Free Currency Rates API](https://github.com/fawazahmed0/currency-api):
```sql
-- load the HTTP extension
create extension if not exists HTTP;
select content from http_get('https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies/usd.json');
```

This returns a JSON object that looks like this:
```json
{
    "date": "2021-09-09",
    "usd": {
        "aed": 3.6732,
        "afn": 86.88109,
        "all": 102.4972,
        ... over 100 more lines here representing currencies ...
    }
}
```
Now we have the current exchange rates, but what are all those currencies?  We can get that from our API too:
```sql
select content from http_get('https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies.json');
```
which returns another JSON object:
```json
{
    "aed": "United Arab Emirates Dirham",
    "afn": "Afghan afghani",
    "all": "Albanian lek",
    ... over 100 more lines here with currency names ...
}
```
### Now that I've got JSON what am I gonna do with it?
Great, so now I know how to get `JSON` objects that give me the current exchange rates and the names of all the currencies.  What can I do with those?  Do I need to parse all that JSON into text and numeric values then store those values in rows and columns of my database then do a `JOIN`?  Well, you could do that if you wanted to.  But Postgres has really great support for `JSON` data that simplifies this.

The easier solution is just to dump this `JSON` data into a `JSON` field in a table.  Well, technically we're going to use a `JSONB` field which is just slightly more efficient, but it's almost exactly the same thing.  See [PostgreSQL JSON Types](https://www.postgresql.org/docs/9.4/datatype-json.html).

### Storing the exchange rates
Since we only need to update our prices once per day, it makes sense for us to store the currency exchange rates in our database to act as a cache.  Let's create a table to store this data:
```sql
create table currency_exchange_rates (currencies JSONB, prices JSONB);
```
Now, we can just grab the data from our API and store it in the table:
```sql
-- create a single empty record in our table to get it started
insert into currency_exchange_rates (currencies, prices)
   values (null, null);
-- update the rates and the currency names
update currency_exchange_rates
  set currencies = 
    (select content from http_get('https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies.json'))::JSONB,
  prices = 
    (select content from http_get('https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies/usd.json'))::JSONB;
```
Let's go through the `SQL` above.  First we're inserting a single record in the `currency_exchange_rates` table with some `null` values.  This is the only record we'll ever need, so we'll only do this once.  Daily we'll just call the `update` command to update this record with the current prices (and currency names, in case they change for some reason.)

The `update` command uses two `subquery` commands to go out to our `api` and get the `JSON`, then we put that `JSON` into our two columns named `currencies` and `prices`.  At the end of each subquery we add `::JSONB` to "cast" the returned data to the `JSONB` type.  You can "cast" or "force" data into a specific type by using the `::typename` operator in PostgreSQL.  Say you want to concatenate a text column `name` and numeric column `age`.  You could say `select name || age::text from table...`.  The `::text` part is forcing the numeric `age` column to be `text` here.

### Let's display our prices
Now you have everything you need for your application.  You may want to send the list of currency types down to your application so your user can select one: `select currencies from currency_exchange_rates;`.  You can also just send the exchange rates to your application and let it display all the different prices: `select prices from currency_exchange_rates;`.  

Or maybe you just want to get the price of `Earl Gray` tea in `Euros`?

```sql
select price * (select prices->'usd'->'eur' from currency_exchange_rates limit 1)::numeric as eur_price
 from tea where id = 'earl';
```
| eur_price   |
| ----------- |
| 11.83993286 |
This `SQL` starts with the simple command `select price from tea where id = 'earl';`  Then we multiple the price by the current exchange rate for Euros, `eur`, which we get using the subquery `select prices->'usd'->'eur' from currency_exchange_rates limit 1` which we force to a numeric with `::numeric`.

We are using the `JSON` `->` [operators](https://www.postgresql.org/docs/9.5/functions-json.html) to get to the `eur` exchange rate.  Basically we just say "grab the `JSONB` data in the `prices` column, then grab the `usd` object from that, and then grab the `eur` object inside of that.  It's pretty easy to drill down into nested `JSON` objects this way.  Of course, this isn't a scalable way to do this process and I wouldn't recommend this approach for a large production app.

But we've used this exercise to demonstrate how to:
- get data from a web `API` using the `HTTP` extension
- take `JSON` data returned from an `API` and store it in a `JSONB` database column
- retrieve `JSONB` data from a database column and parse it to get at specific data
- cast data into a specific type so it can be used in a calculation

I hope this gets your mind wandering into all the fun places you can use the `HTTP` PostgreSQL extension to simplify your application development.
