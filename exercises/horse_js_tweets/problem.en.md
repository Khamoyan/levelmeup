In this exercise you will be provided with a LevelDB store that
contains over 2,000 tweets from [@horse_js](https://twitter.com/horse_js). 
Your job is to query this data set for tweets that were made on a particular 
date.

Each entry is a single tweet. The key is the exact time that the
tweet was sent, in standard ISO date format (i.e. the format
generated by the Date object's `toISOString()` method.) The value
of the entry is simply a String representing the tweet's content.

Write a **module** that exports a single function that
accepts three arguments: an instance of the levelup database, a date string, of
the form YYYY-MM-DD and a callback function.

The first argument to the callback should be an Error if one occurred
or null.

The second argument, if there was no error, should be an
**array where each element is the String text of a tweet**.

The array should contain ordered tweets for the **single day** given
as the first argument to your function. You must not return tweets
for any other day.

Your solution will be checked against the official solution to ensure
that your query is targeting the exact range (see details below).
The output will include the number of "streamed entries".

---
## Hints:

The ISO date format will always sort lexicographically, which means
that our tweets appear in date/time order in our data store without
any special work on our part.

In this exercise, you will need to use the `createReadStream()`
method but you will need to restrict the range of entries to just the
data you need, i.e. you must perform a "range query".

By default the range is the whole store, but you can narrow it down
to a start and/or end key. For this exercise you want to start at
the first tweet on the given day and end at the last tweet of that
day.

Some of the power of range queries comes from the fact that your
start and end keys need not even exist. If your start key does not
exist then the data will start from the entry with a key that
would come next in the sorted order. If your end key does not exist
then your query will end at the entry with a key that would come
before your end key in sorted order.

To target a range, you supply an options object to
`createReadStream()` with `gte` (greater than or equal) and/or `lte` (less than
or equal) properties that define your range keys:

```javascript
db.createReadStream({ gte: 'bar', lte: 'foo' })...
```

Keep in mind that since your range keys need not exist you only need
to supply any prefix of the keys you are looking for, e.g. `2010`
will jump to the first key starting with `2010`, which might end up
matching `2010-09-04T03:51:30.929Z`.

Because the `lte` key of your range will be inclusive, you will need
to create a pseudo-key that would match only the keys you want.
Consider that if you used a `gte` of `2010` and `lte` of `2011` you would
get all entries that start with both `2010` and `2011`. The idiomatic
way _(but not the only way)_ to do this with LevelUP is to append `'\xff'`
to the end of your key, this is the last ASCII character. So, `gte`
of `2010` and `lte` of `2010\xff` would only match keys starting with
`2010`.
