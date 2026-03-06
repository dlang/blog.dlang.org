---
author: JonDegenhardt
comments: false
date: 2017-05-24 13:32:49+00:00
excerpt: This post will show how a few simple D programming constructs can turn an
  already fast command line tool into one that really screams, and in ways that retain
  the inherent simplicity of the original program. The techniques used are applicable
  to many programming problems, not just command line tools. This post describes how
  these methods work and why they are effective. A simple programming exercise is
  used to illustrate these optimizations. Applying the optimizations cuts the run-time
  by more than half.
layout: post
link: https://dlang.org/blog/2017/05/24/faster-command-line-tools-in-d/
slug: faster-command-line-tools-in-d
title: Faster Command Line Tools in D
wordpress_id: 802
categories:
- Code
- Guest Posts
permalink: /faster-command-line-tools-in-d/
redirect_from: /2017/05/24/faster-command-line-tools-in-d/
---

_Jon Degenhardt is a member of eBay's search team focusing on recall, ranking, and search engine design. He is also the author of [eBay's TSV utilities](https://github.com/eBay/tsv-utils), an open source data mining toolkit for delimited files. The toolkit is written in D and is [quite fast](https://github.com/eBay/tsv-utils/blob/master/docs/Performance.md). Much of its performance derives from approaches like those described here._



* * *



![](http://dlang.org/blog/wp-content/uploads/2016/08/d6.png)This post will show how a few simple D programming constructs can turn an already fast command line tool into one that really screams, and in ways that retain the inherent simplicity of the original program. The techniques used are applicable to many programming problems, not just command line tools. This post describes how these methods work and why they are effective. A simple programming exercise is used to illustrate these optimizations. Applying the optimizations cuts the run-time by more than half.


### Task: Aggregate values in a delimited file based on a key


It's a common programming task: Take a data file with fields separated by a delimiter (comma, tab, etc), and run a mathematical calculation involving several of the fields. Often these programs are one-time use scripts, other times they have longer shelf life. Speed is of course appreciated when the program is used more than a few times on large files.

The specific exercise we'll explore starts with files having keys in one field, integer values in another. The task is to sum the values for each key and print the key with the largest sum. For example:

    A   4
    B   5
    B   8
    C   9
    A   6
With the first field as key, second field as value, the key with the max sum is `B`, with a total of 13.

Fields are delimited by a TAB, and there may be any number of fields on a line. The file name and field numbers of the key and value are passed as command line arguments. Below is a Python program to do this:

**max_column_sum_by_key.py**

```python
#!/usr/bin/env python

import argparse
import fileinput
import collections

def main():
    parser = argparse.ArgumentParser(description='Sum a column.')
    parser.add_argument('file', type=open)
    parser.add_argument('key_field_index', type=int)
    parser.add_argument('value_field_index', type=int)

    args = parser.parse_args()
    delim = '\t'

    max_field_index = max(args.key_field_index, args.value_field_index)
    sum_by_key = collections.Counter()

    for line in args.file:
        fields = line.rstrip('\n').split(delim)
        if max_field_index < len(fields):
            sum_by_key[fields[args.key_field_index]] += int(fields[args.value_field_index])

    max_entry = sum_by_key.most_common(1)
    if len(max_entry) == 0:
        print 'No entries'
    else:
        print 'max_key:', max_entry[0][0], 'sum:', max_entry[0][1]

if __name__ == '__main__':
    main()
```
(_Note: For brevity, error handling is largely omitted from programs shown._)

The program follows a familiar paradigm. A dictionary (`collections.Counter`) holds the cumulative sum for each key. The file is read one line at a time, splitting each line into an array of fields. The key and value are extracted. The value field is converted to an integer and added to the cumulative sum for the key. After the program processes all of the lines, it extracts the entry with the largest value from the dictionary.


### The D program, first try


It's a common way to explore a new programming language: write one of these simple programs and see what happens. Here's a D version of the program, using perhaps the most obvious approach:

**max_column_sum_by_key_v1.d**

```d
int main(string[] args)
{
    import std.algorithm : max, maxElement;
    import std.array : split;
    import std.conv : to;
    import std.stdio;

    if (args.length < 4)
    {
        writefln ("synopsis: %s filename keyfield valuefield", args[0]);
        return 1;
    }

    string filename = args[1];
    size_t keyFieldIndex = args[2].to!size_t;
    size_t valueFieldIndex = args[3].to!size_t;
    size_t maxFieldIndex = max(keyFieldIndex, valueFieldIndex);
    char delim = '\t';

    long[string] sumByKey;

    foreach(line; filename.File.byLine)
    {
        auto fields = line.split(delim);
        if (maxFieldIndex < fields.length)
        {
            string key = fields[keyFieldIndex].to!string;
            sumByKey[key] += fields[valueFieldIndex].to!long;
        }
    }

    if (sumByKey.length == 0) writeln("No entries");
    else
    {
        auto maxEntry = sumByKey.byKeyValue.maxElement!"a.value";
        writeln("max_key: ", maxEntry.key, " sum: ", maxEntry.value);
    }
    return 0;
}
```
Processing is basically the same as the Python program. An associative array (`long[string] sumByKey`) holds the cumulative sum for each key. Like the Python program, it splits each line into an array of fields, extracts the key and value fields, and updates the cumulative sum. Finally, it retrieves and prints the entry with the maximum value.

We will measure performance using an ngram file from the Google Books project: [googlebooks-eng-all-1gram-20120701-0](https://storage.googleapis.com/books/ngrams/books/googlebooks-eng-all-1gram-20120701-0.gz) (`ngrams.tsv` in these runs). This file is 10.5 million lines, 183 MB. Each line has four fields: the ngram, year, total occurrence count, and the number of books the ngram appeared in. Visit the [ngram viewer dataset page](https://storage.googleapis.com/books/ngrams/books/datasetsv2.html) for more information. The file chosen is for unigrams starting with the digit zero. Here are a few lines from the file:

    0       1898    114067  6140
    0       1906    208805  7933
    0       1922    204995  9042
    0.5     1986    143398  13938
    0.5     1999    191449  19262
The `year` (second column) is used as the key, and the `total occurrence count` (third column) as the value. There are 414 distinct years in the data file.

The LDC compiler is used to build the D programs, as it generates fast code:

```bash
$ ldc2 -release -O max_column_sum_by_key_v1.d
```
Here are the commands to perform the task:

```bash
$ max_column_sum_by_key.py ngrams.tsv 1 2   # Python program
max_key: 2006 sum: 22569013

$ max_column_sum_by_key_v1 ngrams.tsv 1 2   # D program
max_key: 2006 sum: 22569013
```
(_Note: These programs use field numbers starting at zero._)

The `time` command was used to measure performance. e.g. `$ time max_column_sum_by_key.py ngrams.tsv 1 2`. On the author's MacBook Pro, the Python version takes 12.6 seconds, the D program takes 3.2 seconds. This makes sense as the D program is compiled to native code. But suppose we run the Python program with [PyPy](https://pypy.org/), a just-in-time Python compiler? This gives a result of 2.4 seconds, actually beating the D program, with no changes to the Python code. Kudos to PyPy, this is an impressive result. But we can still do better with our D program.


### Second version: Using `splitter`


The first key to improved performance is to switch from using `split` to `splitter`. The `split` function is “eager”, in that it constructs and returns the entire array of fields. Eventually the storage for these fields needs to be deallocated. `splitter` is "lazy". It operates by returning an `input range` that iterates over the fields one-at-a-time. We can take advantage of that by avoiding constructing the entire array, and instead keeping a single field at a time in a reused local variable. Here is an augmented program that does this, the main change being the introduction of an inner loop iterating over each field:

**max_column_sum_by_key_v2.d**

```d
int main(string[] args)
{
    import std.algorithm : max, maxElement, splitter;
    import std.conv : to;
    import std.range : enumerate;
    import std.stdio;

    if (args.length < 4)
    {
        writefln ("synopsis: %s filename keyfield valuefield", args[0]);
        return 1;
    }

    string filename = args[1];
    size_t keyFieldIndex = args[2].to!size_t;
    size_t valueFieldIndex = args[3].to!size_t;
    size_t maxFieldIndex = max(keyFieldIndex, valueFieldIndex);
    string delim = "\t";

    long[string] sumByKey;

    foreach(line; filename.File.byLine)
    {
        string key;
        long value;
        bool allFound = false;

        foreach (i, field; line.splitter(delim).enumerate)
        {
            if (i == keyFieldIndex) key = field.to!string;
            if (i == valueFieldIndex) value = field.to!long;
            if (i == maxFieldIndex) allFound = true;
        }

        if (allFound) sumByKey[key] += value;
    }

    if (sumByKey.length == 0) writeln("No entries");
    else
    {
        auto maxEntry = sumByKey.byKeyValue.maxElement!"a.value";
        writeln("max_key: ", maxEntry.key, " sum: ", maxEntry.value);
    }
    return 0;
}
```
The modified program is quite a bit faster, running in 1.8 seconds, a 44% improvement. Insight into what changed can be seen by using the `--DRT-gcopt=profile:1` command line option. This turns on garbage collection profiling, shown below (_output edited for brevity_):

```bash
$ max_column_sum_by_key_v1 --DRT-gcopt=profile:1 ngrams.tsv 1 2
max_key: 2006 sum: 22569013
        Number of collections:  132
        Grand total GC time:  246 milliseconds
GC summary:   35 MB,  132 GC  246 ms

$ max_column_sum_by_key_v2 --DRT-gcopt=profile:1 ngrams.tsv 1 2
max_key: 2006 sum: 22569013
      Number of collections:  167
      Grand total GC time:  101 milliseconds
GC summary:    5 MB,  167 GC  101 ms
```
(_Note: The `--DRT-gcopt=profile:1` parameter is invisible to normal option processing._)

The reports show two key differences. One is the 'max pool memory', the first value shown on the "GC summary line". The significantly lower value indicates less memory is being allocated. The other is the total time spent in collections. The improvement, 145ms, only accounts for a small portion of the 1.4 seconds that were shaved off by the second version. However, there are other costs associated with storage allocation. Note that allocating and reclaiming storage has a cost in any memory management system. This is not limited to systems using garbage collection.

Also worth mentioning is the role D's [slices](http://dlang.org/d-array-article.html) play. When `splitter` returns the next field, it is not returning a copy of characters in the line. Instead, it is returning a "slice". The data type is a `char[]`, which is effectively a pointer to a location in the input line and a length. No characters have been copied. When the next field is fetched, the variable holding the slice is updated (pointer and length), a faster operation than copying a variable-length array of characters. This is a remarkably good fit for processing delimited files, as identifying the individual fields can be done without copying the input characters.


### Third version: The `splitter` / `Appender` combo


Switching to `splitter` was a big speed win, but came with a less convenient programming model. Extracting specific fields while iterating over them is cumbersome, more so as additional fields are needed. Fortunately, the simplicity of random access arrays can be reclaimed by using an `Appender`. Here is a revised program:

**max_column_sum_by_key_v3.d**

```d
int main(string[] args)
{
    import std.algorithm : max, maxElement, splitter;
    import std.array : appender;
    import std.conv : to;
    import std.stdio;

    if (args.length < 4)
    {
        writefln ("synopsis: %s filename keyfield valuefield", args[0]);
        return 1;
    }

    string filename = args[1];
    size_t keyFieldIndex = args[2].to!size_t;
    size_t valueFieldIndex = args[3].to!size_t;
    size_t maxFieldIndex = max(keyFieldIndex, valueFieldIndex);
    string delim = "\t";

    long[string] sumByKey;
    auto fields = appender!(char[][])();

    foreach(line; filename.File.byLine)
    {
        fields.clear;
        fields.put(line.splitter(delim));
        if (maxFieldIndex < fields.data.length)
        {
            string key = fields.data[keyFieldIndex].to!string;
            sumByKey[key] += fields.data[valueFieldIndex].to!long;
        }
    }

    if (sumByKey.length == 0) writeln("No entries");
    else
    {
        auto maxEntry = sumByKey.byKeyValue.maxElement!"a.value";
        writeln("max_key: ", maxEntry.key, " sum: ", maxEntry.value);
    }
    return 0;
}
```
The `Appender` instance in this program works by keeping a growable array of `char[]` slices. The lines:

```d
    fields.clear;
    fields.put(line.splitter(delim));
```
at the top of the `foreach` loop do the work. The statement `fields.put(line.splitter(delim))` iterates over each field, one at a time, appending each slice to the array. This will allocate storage on the first input line. On subsequent lines, the `fields.clear` statement comes into play. It clears data from the underlying data store, but does not deallocate it. Appending starts again at position zero, but reusing the storage allocated on the first input line. This regains the simplicity of indexing a materialized array. GC profiling shows no change from the previous version of the program.

Copying additional slices does incur a performance penalty. The resulting program takes 2.0 seconds, versus 1.8 for the previous version. This is still a quite significant improvement over the original program (down from 3.2 seconds, 37% faster), and represents a good compromise for many programs.


### Fourth version: Associative Array (AA) lookup optimization


The `splitter` / `Appender` combo gave significant performance improvement while retaining the simplicity of the original code. However, the program can still be faster. GC profiling indicates storage is still being allocated and reclaimed. The source of the allocations is the following two lines in the inner loop:

```d
    string key = fields.data[keyFieldIndex].to!string;
    sumByKey[key] += fields.data[valueFieldIndex].to!long;
```
The first line converts `fields.data.[keyFieldIndex]`, a `char[]`, to a `string`. The `string` type is immutable, `char[]` is not, forcing the conversion to make a copy. This is both necessary and required by the associative array. The characters in the `fields.data` buffer are valid only while the current line is processed. They will be overwritten when the next line is read. Therefore, the characters forming the key need to be copied when added to the associative array. The associative array enforces this by requiring immutable keys.

While it is necessary to store the key as an immutable value, it is not necessary to use immutable values to retrieve existing entries. This creates the opportunity for an improvement: only copy the key when creating the initial entry. Here's a change to the same lines that does this:

```d
    char[] key = fields.data[keyFieldIndex];
    long fieldValue = fields.data[valueFieldIndex].to!long;

    if (auto sumValuePtr = key in sumByKey) *sumValuePtr += fieldValue;
    else sumByKey[key.to!string] = fieldValue;
```
The expression `key in sumByKey` returns a pointer to the value in the hash table, or `null` if the key was not found. If an entry was found, it is updated directly, without copying the key. Updating via the returned pointer avoids a second associative array lookup. A new string is allocated for a key only the first time it is seen.

The updated program runs in 1.4 seconds, an improvement of 0.6 seconds (30%). GC profiling reflects the change:

```bash
$ ./max_column_sum_by_key_v4 --DRT-gcopt=profile:1 ngrams.tsv 1 2
max_key: 2006 sum: 22569013
        Number of collections:  2
        Grand total GC time:  0 milliseconds
GC summary:    5 MB,    2 GC    0 ms
```
This indicates that unnecessary storage allocation has been eliminated from the main loop.

_Note: The program will still allocate and reclaim storage as part of rehashing the associative array. This shows up on GC reports when the number of unique keys is larger._


### Early termination of the field iteration loop


The optimizations described so far work by reducing unnecessary storage allocation. These are by far the most beneficial optimizations discussed in this document. Another small but obvious enhancement would be to break out of the field iteration loops after all needed fields have been processed. In version 2, using `splitter`, the inner loop becomes:

```d
    foreach (i, field; line.splitter(delim).enumerate)
    {
        if (i == keyFieldIndex) key = field.to!string;
        if (i == valueFieldIndex) value = field.to!long;
        if (i == maxFieldIndex)
        {
            allFound = true;
            break;
        }
    }
```
This produced a 0.1 second improvement. A small gain, but will be larger in use cases excluding a larger number of fields.

The same optimization can be applied to the `splitter` / `Appender` combo. The D standard library provides a convenient way to do this: the `take` method. It returns an input range with at most N elements, effectively short circuiting traversal. The change is to the `fields.put(line.splitter(delim))` line:

```d
    import std.range : take;
    ...
    fields.put(line.splitter(delim).take(maxFieldIndex + 1));
```
### Putting it all together


The final version of our program is below, adding `take` for early field iteration termination to version 4 (`splitter`, `Appender`, associative array optimization). For a bit more speed, drop `Appender` and use the manual field iteration loop shown in version two (version 5 in the results table at the end of this article).

**max_column_sum_by_key_v4b.d**

```d
int main(string[] args)
{
    import std.algorithm : max, maxElement, splitter;
    import std.array : appender;
    import std.conv : to;
    import std.range : take;
    import std.stdio;

    if (args.length < 4)
    {
        writefln ("synopsis: %s filename keyfield valuefield", args[0]);
        return 1;
    }

    string filename = args[1];
    size_t keyFieldIndex = args[2].to!size_t;
    size_t valueFieldIndex = args[3].to!size_t;
    size_t maxFieldIndex = max(keyFieldIndex, valueFieldIndex);
    string delim = "\t";

    long[string] sumByKey;
    auto fields = appender!(char[][])();

    foreach(line; filename.File.byLine)
    {
        fields.clear;
        fields.put(line.splitter(delim).take(maxFieldIndex + 1));
        if (maxFieldIndex < fields.data.length)
        {
            char[] key = fields.data[keyFieldIndex];
            long fieldValue = fields.data[valueFieldIndex].to!long;

            if (auto sumValuePtr = key in sumByKey) *sumValuePtr += fieldValue;
            else sumByKey[key.to!string] = fieldValue;
        }
    }

    if (sumByKey.length == 0) writeln("No entries");
    else
    {
        auto maxEntry = sumByKey.byKeyValue.maxElement!"a.value";
        writeln("max_key: ", maxEntry.key, " sum: ", maxEntry.value);
    }
    return 0;
}
```
### Summary


This exercise demonstrates several straightforward ways to speed up command line programs. The common theme: avoid unnecessary storage allocation and data copies. The results are dramatic, more than doubling the speed of an already quick program. They are also a reminder of the crucial role memory plays in high performance applications.

Of course, these themes apply to many applications, not just command line tools. They are hardly specific to the D programming language. However, several of D's features proved especially well suited to minimizing both storage allocation and data copies. This includes ranges, dynamic arrays, and slices, which are related concepts, and lazy algorithms, which operate on them. All were used in the programming exercise.

The table below compares the running times of each of the programs tested:
<table >
<tbody >
<tr >
Program
What
Time(sec)
</tr>
<tr >

<td >Python Program
</td>

<td >Run with Python2
</td>

<td style="text-align: center;" >12.6
</td>
</tr>
<tr >

<td >Python Program
</td>

<td >Run with PyPy
</td>

<td style="text-align: center;" >2.4
</td>
</tr>
<tr >

<td >D version 1
</td>

<td >Using `split`
</td>

<td style="text-align: center;" >3.2
</td>
</tr>
<tr >

<td >D version 2
</td>

<td >Replace `split` with `splitter`
</td>

<td style="text-align: center;" >1.8
</td>
</tr>
<tr >

<td >D version 3
</td>

<td >`splitter`/`Appender` combo
</td>

<td style="text-align: center;" >2.0
</td>
</tr>
<tr >

<td >D version 4
</td>

<td >`splitter`/`Appender`, AA optimization
</td>

<td style="text-align: center;" >1.4
</td>
</tr>
<tr >

<td >D version 4b
</td>

<td >Version 4 plus `take`
</td>

<td style="text-align: center;" >1.3
</td>
</tr>
<tr >

<td >D version 5
</td>

<td >`splitter`, AA optimization, loop exit
</td>

<td style="text-align: center;" >1.1
</td>
</tr>
</tbody>
</table>
_The author thanks Ali Çehreli, Steven Schveighoffer, and Steve Schneider for providing valuable input to this article._
