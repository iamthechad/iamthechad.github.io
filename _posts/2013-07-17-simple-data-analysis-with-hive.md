---
title: Simple Data Analysis with Hive
excerpt: Follow this simple example to get started analyzing real-world data with Hive and Hadoop.
tags: development hive
categories: Development
toc: true
redirect_from: "/simple-data-analysis-with-hive-cb4a48700419"
---

Hive is a data warehouse system for Hadoop that facilitates easy data summarization, ad-hoc queries, and the analysis of large datasets stored in Hadoop compatible file systems. Hive provides a mechanism to project structure onto this data and query the data using a SQL-like language called HiveQL.

This is a simple getting started example that’s based upon “[Hive for Beginners](http://www.orzota.com/hive-for-beginners/)”, with what I feel is a bit more useful information.

([This example is also available using Pig.]({{ site.baseurl }}{% post_url 2013-07-16-simple-data-analysis-with-pig %}))

The installation and configuration of Hadoop and Hive is beyond the scope of this article. If you’re just getting started, I would highly recommend grabbing one of [Cloudera’s pre-built virtual machines](https://ccp.cloudera.com/display/SUPPORT/Cloudera+QuickStart+VM) that have everything you need.

> Cloudera’s VMs have changed substantially since this article was written. I have not been able to verify that the new VMs will work with these instructions.

I’m assuming that you will be running the following steps using the [Cloudera VM](https://ccp.cloudera.com/display/SUPPORT/Cloudera+QuickStart+VM), logged in as the cloudera user. If your setup is different, adjust accordingly.

### Step 1: Preparing the Data

Like “[Hive for Beginners](http://www.orzota.com/hive-for-beginners/)”, we’re going to use the [Book Crossing Dataset](http://www.informatik.uni-freiburg.de/~cziegler/BX/). Download the CSV dump and extract the files. We’re interested in the `BX-Books.csv` data file.

Type `head BX-Books.csv` to see the first few lines of the raw data. You’ll notice that’s it’s not really comma-delimited; the delimiter is ‘`;`’. There are also some escaped HTML entities we can clean up, and the quotes around all of the values can be removed.

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

Now that we have some normalized data, we need to add it to the Hadoop file system (HDFS) so that Hive can access it. In the terminal, type:

```bash
hadoop fs -mkdir input
hadoop fs -put /path/to/BX-BooksCorrected.txt input
```

(Use the correct path to the `BX-BooksCorrected.txt` file.)

### Step 3: Running Hive

Enter `hive` at the console to start Hive.

Once Hive has started, you’ll see the `hive>` command prompt. Let’s verify that our file did get loaded into HDFS.

```bash
dfs -ls input;
```

### Step 4: Analyzing the Data

Now that we have the data ready, let’s do something with it. The simple example is to see how many books were published per year. We’ll start with that, then see if we can do a bit more.

**Load the data into a Hive table:**

```bash
CREATE TABLE IF NOT EXISTS BookData
> (ISBN STRING,
> BookTitle STRING,
> BookAuthor STRING,
> YearOfPublication INT,
> Publisher STRING)
> ROW FORMAT DELIMITED
> FIELDS TERMINATED BY '\;'
> STORED AS TEXTFILE;

LOAD DATA INPATH '/user/cloudera/input/BX-BooksCorrected.txt'
> OVERWRITE INTO TABLE BookData;
```

(HQL commands are terminated with semicolons. If you press Return on a line without terminating it, you’ll get the `>` character, indicating that the command has been continued and not entered yet.)
 This creates a Hive table named `BookData` and loads the information from HDFS into it. Hive expects data to be tab-delimited by default, but ours is not; we have to tell it that semicolons are field separators by providing the `FIELDS TERMINATED BY` argument. You’ll notice that we left off all of the “`Image-URL-XXX`” fields; we don’t need them for analysis, and Hive will ignore fields that we don’t tell it to load.

If you want to see what the loaded data structure looks like, you can use the `DESCRIBE` command:

```bash
DESCRIBE BookData;
```

### Finding books by year

We’ll start with the simple analysis of how many books were written by year. In Hive, this can be accomplished with a single query.

**Get number of books by year:**

```bash
SELECT YearOfPublication, COUNT(BookTitle)
> FROM BookData GROUP BY YearOfPublication;
```

You’ll see a listing of years, along with the number of books for that year. You may notice that some of the values don’t make much sense; there should be no year 0, nor should there be entries for a blank year. We’ll clean those problems up in the next analysis.

### More Advanced Analysis

There’s a lot more data in the set beyond years and books counts. What if we wanted to see books published per year by author? Why don’t we go a step farther and group those results by publisher as well?

Let’s do a little bit of cleanup on the data to eliminate the unwanted years.

```bash
INSERT OVERWRITE TABLE BookData
> SELECT BookData.*
> FROM BookData WHERE YearOfPublication > 0;
```

This will only keep records where we have a positive year of publication value.

**Generating the final results is again a single query:**

```bash
SELECT Publisher, BookAuthor, YearOfPublication, COUNT(BookTitle)
> FROM BookData
> GROUP BY Publisher, BookAuthor, YearOfPublication;
```

At last! We have our results.

(Want to compare the Hive steps with Pig? [Here’s the same example using Pig.]({{ site.baseurl }}{% post_url 2013-07-16-simple-data-analysis-with-pig %}))
