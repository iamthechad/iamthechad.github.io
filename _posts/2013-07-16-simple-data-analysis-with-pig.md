---
title: Simple Data Analysis with Pig
excerpt: Follow this simple example to get started analyzing real-world data with Apache Pig and Hadoop.
toc: true
tags: development pig
categories: Development
redirect_from: "/simple-data-analysis-with-pig-ee4390c93085"
---

Pig is a platform that works with large data sets for the purpose of analysis. The Pig dialect is called Pig Latin, and the Pig Latin commands get compiled into MapReduce jobs that can be run on a suitable platform, like Hadoop.

This is a simple getting started example that’s based upon “[Pig for Beginners](http://www.orzota.com/pig-for-beginners/)”, with what I feel is a bit more useful information.

([This example is also available using Hive.](https://megatome.com/simple-data-analysis-with-hive-cb4a48700419#.xwarutlku))

The installation and configuration of Hadoop and Pig is beyond the scope of this article. If you’re just getting started, I would highly recommend grabbing one of [Cloudera’s pre-built virtual machines](https://www.cloudera.com/downloads/quickstart_vms/5-8.html) that have everything you need.

> Cloudera’s VMs have changed substantially since this article was written. I have not been able to verify that the new VMs will work with these instructions.

I’m assuming that you will be running the following steps using the [Cloudera VM](https://www.cloudera.com/downloads/quickstart_vms/5-8.html), logged in as the `cloudera` user. If your setup is different, adjust accordingly.

### Step 1: Preparing the Data

Like “[Pig for Beginners](http://www.orzota.com/pig-for-beginners/)”, we’re going to use the [Book Crossing Dataset](http://www.informatik.uni-freiburg.de/~cziegler/BX/). Download the CSV dump and extract the files. We’re interested in the `BX-Books.csv` data file.

Type `head BX-Books.csv` to see the first few lines of the raw data. You’ll notice that’s it’s not really comma-delimited; the delimiter is ‘`;`‘. There are also some escaped HTML entities we can clean up, and the quotes around all of the values can be removed.

The first line in the file looks like this:

```bash
"ISBN";"Book-Title";"Book-Author";"Year-Of-Publication";"Publisher";"Image-URL-S";"Image-URL-M";"Image-URL-L"
```

This lines defines the data format of the fields in the file. We’ll want to refer back to it later.

Open a terminal and enter:

```bash
sed 's/\&amp;/\&/g' BX-Books.csv | sed -e '1d' |sed 's/;/$$$/g' | sed 's/"$$$"/";"/g' | sed 's/"//g' > BX-BooksCorrected.txt
```

This will:

1.  Replace all `&amp;` instances with `&`
2.  Remove the first (header) line
3.  Change all semicolons to `$$$`
4.  Change all `"$$$"` instances into `";"`
5.  Remove all `"` characters

Steps 3 and 4 may look strange, but some of the field content may contain semicolons. In this case, they will be converted to `$$$`, but they will not match the `"$$$"` pattern, and will not be converted back into semicolons and mess up the import process.

### Step 2: Importing the Data

Now that we have some normalized data, we need to add it to the Hadoop file system (HDFS) so that Pig can access it. This can be accomplished from the command line as well as within Pig itself. For this example, we’ll use the command line. In the terminal, type:

```bash
hadoop fs -mkdir input
hadoop fs -put /path/to/BX-BooksCorrected.txt input
```

(Use the correct path to the `BX-BooksCorrected.txt` file.)

### Step 3: Running Pig

There are several ways to run Pig. We’re going to run in MapReduce mode against the local node. MapReduce mode runs against the HDFS, which is why we needed to import the file in the previous step.

Enter `pig` at the console to start Pig.

Once Pig has started, you’ll see the `grunt>` command prompt. Let’s verify that our file did get loaded into HDFS.

```bash
ls
hdfs://localhost.localdomain:8020/user/cloudera/input	<dir>
cd input
ls
hdfs://localhost.localdomain:8020/user/cloudera/input/BX-BooksCorrected.txt	<dir>
```

### Step 4: Analyzing the Data

Now that we have the data ready, let’s do something with it. The simple example is to see how many books were published per year. We’ll start with that, then see if we can do a bit more.

**Load the data into a Pig collection:**

```bash
books = LOAD '/user/cloudera/input/BX-BooksCorrected.txt'
>> USING PigStorage(';') AS
>> (ISBN:chararray, BookTitle:chararray,
>> BookAuthor:chararray, YearOfPublication:int,
>> Publisher:chararray);
```

(Pig Latin commands are terminated with semicolons. If you press Return on a line without terminating it, you’ll get the `>>` characters, indicating that the command has been continued and not entered yet.)
 This line loads the information from HDFS into a Pig collection named `books`. Pig expects data to be tab-delimited by default, but ours is not; we have to tell it that semicolons are field separators by providing the `PigStorage()` argument. The `AS` clause defines how the fields in the file are mapped into Pig data types. You’ll notice that we left off all of the “`Image-URL-XXX`” fields; we don’t need them for analysis, and Pig will ignore fields that we don’t tell it to load.

If you want to see what the loaded data structure looks like, you can use the `DESCRIBE` command:

```bash
DESCRIBE books;
books: {ISBN: chararray,BookTitle: chararray,BookAuthor: chararray,YearOfPublication: int,Publisher: chararray}
```

### Finding books by year

We’ll start with the simple analysis of how many books were written by year.

**Group the collection by year of publication:**

```bash
groupByYear = GROUP books BY YearOfPublication;

DESCRIBE groupByYear;
groupByYear: {group: int,books: {(ISBN: chararray,BookTitle: chararray,BookAuthor: chararray,YearOfPublication: int,Publisher: chararray)}}
```

This is a lot like a Java Map. The `groupByYear` collection contains all unique year values, along with a list holding the full entry of each book that belongs to that year.

**Generate book count by year:**

```bash
countByYear = FOREACH groupByYear
>> GENERATE group AS YearOfPublication, COUNT($1) AS BookCount;

DESCRIBE countByYear;
countByYear: {YearOfPublication: int,BookCount: int}
```

This is the meat of the operation. The `FOREACH` loops over the `groupByYear` collection, and we `GENERATE` values. Our output is defined using some values available to us within the `FOREACH`. We first take `group`, which is an alias for the grouping value and say to place it in our new collection as an item named `YearOfPublication`. (This value is also available as `$0`.) The next value `$1` is an alias for the list of book entries for that group. We only care about the number of books, so we use `COUNT` and give it the name `BookCount`.

We have the results, but how do we see them? We could store them back into HDFS and extract them that way, or we can use the `DUMP` command.

```bash
DUMP countByYear;
```

You’ll see a listing of years, along with the number of books for that year. You may notice that some of the values don’t make much sense; there should be no year 0, nor should there be entries for a blank year. We’ll clean those problems up in the next analysis.

### More Advanced Analysis

There’s a lot more data in the set beyond years and books counts. What if we wanted to see books published per year by author? Why don’t we go a step farther and group those results by publisher as well?

You should still have your `books` collection defined if you haven’t exited your Pig session. You can redefine it easily by following the above steps again. Let’s do a little bit of cleanup on the data this time, however.

```bash
books = FILTER books BY YearOfPublication > 0;
```

This will only keep records where we have a positive year of publication value.

**Now, we need to create a set of all authors, and all years they wrote books:**

```bash
pivot = FOREACH (GROUP books BY BookAuthor)
>> GENERATE group AS BookAuthor, FLATTEN(books.YearOfPublication) AS Year;
```

Note that I’ve inlined the group creation in the `FOREACH` statement. It should be obvious that we’re grouping `books` by author.
 This statement also introduces the `FLATTEN` operation. We know that the `GROUP` operation creates a collection where each key corresponds to a list of values; `FLATTEN` “flattens” this list to generate entries for each list value. (See the [Pig Latin reference for](http://pig.apache.org/docs/r0.11.1/basic.html#flatten) a more detailed definition.) You may want to `DUMP` the `pivot` collection to see how the flattening works.

**Create author book count by year:**

```bash
authorYearGroup = GROUP pivot BY (BookAuthor, Year);

with_count = FOREACH authorYearGroup
>> GENERATE FLATTEN(group), COUNT(pivot) as count;

DESCRIBE with_count;

with_count: {group::BookAuthor: chararray,group::Year: int, count: long}
```

This time, we’re grouping on 2 fields. This will find all (author, year) combinations. We then execute a `FOREACH` over the group; this time, however, we’re using `FLATTEN` on the `group` which will expand to `BookAuthor` and `Year`. We also grab the count of items to get the number of books for that author/year combination. You can see the resulting data structure in the `DESCRIBE` results.

**Let’s group those results a little better:**

```bash
author_result = FOREACH (GROUP with_count BY BookAuthor) {
>> order_by_count = ORDER with_count BY count DESC;
>> GENERATE group AS Author, order_by_count.(Year, count) AS Books;
>> };

DESCRIBE author_result;

author_result: {Author: chararray,Books: {(group::Year: int,count: long)}}
```

We’re running a `FOREACH` that looks different than the ones we’ve seen to this point. This nested structure lets us perform some extra steps before generation of values. In this case, we’re sorting the book count, then placing the year/count tuples into the collection with the author.

Group the author results by publisher:

```bash
pub_auth = FOREACH books GENERATE Publisher, BookAuthor;

distinct_authors = FOREACH (GROUP pub_auth BY Publisher) {
>> da = DISTINCT pub_auth.BookAuthor;
>> GENERATE group AS Publisher, da AS Author;
>> };

distinct_flat = FOREACH distinct_authors GENERATE Publisher, FLATTEN(Author) AS Author;
```

First, we use a projection to extract only the publisher and author from the books collection. This is a [recommended practice](http://pig.apache.org/docs/r0.11.1/perf.html#projection) as it helps with performance. In this `FOREACH`, we’re grabbing the distinct authors per publisher. This collection is then flattened into `distinct_flat`, and we end up with a publisher/author list, similar to the author/year collection previously.

**Join the results so far:**

```bash
joined = JOIN distinct_flat BY Author, author_result BY Author;
filtered = FOREACH joined GENERATE
>> distinct_flat::Publisher AS Publisher,
>> distinct_flat::Author AS Author,
>> author_result::Books AS Books;
```

Here we’re joining the author/year results with the publisher/author results. The “`::`” syntax may look a little different than what we’ve seen before; it’s a dereference operator, and it’s used when you need to unambiguously declare a column. Try `DESCRIBE` on `joined` and `filtered` to see the structure difference.

**Generate the final results:**

```bash
result = FOREACH (GROUP filtered BY Publisher) {
>> order_by_pub = ORDER filtered BY Publisher ASC;
>> GENERATE group AS Publisher, order_by_pub.(Author, Books); };
```

This should be familiar by now. We sort publishers, then generate a collection of publishers/authors/books.

At last! We have our results. You can `DUMP` them, but let’s also save the results back into HDFS.

```bash
STORE result INTO '/user/cloudera/publisher_books';
```

Now you can exit Pig with “`quit`”.

**Let’s get a file listing:**

```bash
hadoop fs -ls publisher_books
```

Hadoop stores `publisher_books` as a directory with associated files. Our actual data is in `part-r-00000`

```bash
hadoop fs -cat publisher_books/part-r-00000
```

(Want to compare the Pig steps with Hive? [Here’s the same example using Hive.]({{ site.baseurl }}{% post_url 2013-07-17-simple-data-analysis-with-hive %}))
