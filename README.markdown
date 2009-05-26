Quick Start
===========

Jsawk is like awk, but for JSON. You work with an array of JSON objects
read from stdin, filter them using JavaScript to produce a results array
that is printed to stdout. You can use this as a filter to manipulate data
from a REST JSON web service, for example, in a shell script. Also, you can
suppress JSON output and use the built-in printing functions to translate
your JSON input to other formats and send that to stdout, to be piped to
other processes. You can load JavaScript libraries on the command line to
increase your processing power, and other things.

Setup
-----

You need to have the `js` interpreter installed. On a Debian system you can
install it with the command `apt-get install spidermonkey-bin`. Or you can 
build it from source from [here](http://www.mozilla.org/js/spidermonkey/).
Ready? Go.

Install
-------

First, get the jsawk script:

      curl http://github.com/micha/jsawk/raw/master/jsawk > jsawk

Then make it executable and put it somewhere in your path:

      chmod 755 jsawk && mv jsawk ~/bin/

Use
---

Now you can do some stuff with JSON data. Here's an example using data from
a REST service that serves JSON (we use [resty](http://github.com/micha/resty)
to do the HTTP requests):

      resty http://example.com:8080/data*.json
      GET /people/47 | jsawk 'this.favoriteColor = "blue"' | PUT /people/47

This would do a `GET` request on the resource `/data/people/47.json`, which
would result in a JSON object. Then jsawk takes the JSON via stdin and for
each JSON object it runs the little snippet of JavaScript, setting the
`favoriteColor` property to `"blue"`, in this case. The modified JSON is then
output via stdout to `resty` again, which does the `PUT` request to update
the resource.

Usage
=====

      jsawk [OPTIONS] [SCRIPT]

      OPTIONS
      -------

      -h
          Print short help page and exit.

      -n
          Suppress printing of JSON result set.

      -d <delim>
          Choose a token to indicate end of input. If this token appears on
          a line by itself then reading of JSON input will end. Otherwise,
          10 blank lines in a row will end input. (Don't look at me, this
          is a spidermonkey issue.) Default delimiter is a single period.

      -q <query>
          Filter JSON through the specified JSONQuery query.

      -f <file>
          Load and run the specified JavaScript file prior to processing
          JSON. This option can be specified multiple times to load multiple
          JavaScript libraries.

      -b <script> | -a <script>
          Run the specified snippet of JavaScript before (-b) or after (-a)
          processing JSON input. The `this` object is set to the whole JSON
          array or object. This is used to preprocess (-b) or postprocess
          (-a) the JSON array before or after the main script is applied.

      SCRIPT
      ------

      This is a snippet of JavaScript that will be run on each element
      of the input array, if input is a JSON array, or on the object if
      it's an object. For each iteration, the `this` object is set to the
      current element.

Jsawk Scripting
===============

Jsawk is intended to serve the purpose that is served by `awk` in the shell
environment, but instead of working with words and lines of text, it works
with JavaScript objects and arrays of objects.

In awk, a text file is split into an array of "records", each of which being
an array of "fields". The awk script that is specified on the command line is
run once for each record in the array, with the `$1`, `$2`, etc. variables
set to the various fields in the record. The awk script can set variables,
perform calculations, do various text-munging things, and print output. This
printing capablity makes awk into a filter, taking text input, transforming
it record by record, printing out the resulting modified records at the end.

Jsawk is similar, but in jsawk records are elements of the JSON input array
(if the input was a single object then there is a single record consisting
of that object). The jsawk script is run once for each record object, with
the `this` object set to the current record. So here the properties of the
record object are equivalent to the `$1`, `$2`, etc. in awk. The jsawk
script can then modify the record, perform calculations, do things. However,
instead of printing the modified record, the modified record is `return`ed.
At then end, if the `-n` option was not specified, the resulting array is
printed as JSON to stdout.

Jsawk JavaScript Environment
----------------------------

Jsawk uses the Spidermonkey JavaScript interpreter, so you have access to all
of the Spidermonkey functions and whatnot. Additionally, the following
functions and properties are available from within a jsawk script:

      PROPERTIES
      ----------

        window
            The global object.

        IS
            The input set.

        RS
            The result set.

        $_
            The current record index (corresponding to the index of the
            element in the IS array).

        $$
            The current record object (global variable corresponding to the
            `this` object in the script scope).

      METHODS
      -------

        get()
            Get the next record from the input set. This will prevent jsawk
            from iterating over that record.

            params: void
            return: Object|Array|Number|String (the next input record)

        put(record)
            Push 'record' onto the input set so that jsawk will iterate over
            it next.

            params: Object|Array|Number|String record (the record to push)
            return: void

        json(thing)
            Serialize 'thing' to JSON string.

            params: Object|Array|Number|String thing (what to serialize)
            return: String (the resulting JSON string)

        Q(query, thing)
            Runs the JSONQuery 'query' on the JSON input 'thing'.

            params: String query (the JSONQuery)
                    Array|Object thing (the JSON input)
            return: Array|Object (result of running the query)

        err(thing)
            Print arguments (JSON encoded, if necessary) to stderr.

            params: Object|Array|Number|String thing (what to encode)
            return: void

        out(thing)
            Print arguments (JSON encoded, if necessary) to stdout.

            params: Object|Array|Number|String thing (what to encode)
            return: void

Errors and Output
-----------------

Errors in parsing scripts, JSON queries, or JSON input, and errors executing
scripts will all result in the appropriate error message on stderr, and
immediate exit with a non-zero exit status. Normal output is written to
stdout, unless the `-n` option is specified. In that case only output from
the `out()` or `err()` functions and error messages will appear.

Exit Status
-----------

On successful completion jsawk returns an exit status of `0`. If an error
ocurred and execution was aborted, a non-zero exit status will be returned.

### Exit Status

  * **0** Successful completion.
  * **1** Command line parsing error.
  * **2** JSON parsing error.
  * **3** Script error.
  * **4** JSONQuery parsing error.
  * **5** JSON stringify error.

JSONQuery
=========

Jsawk supports JSONQuery with the `-q` option. You can do almost anything
with JSONQuery that you can do with jsawk scripts, to include selecting
records, drilling down into records, mapping input sets to output sets as
a sort of filter, modifying the JSON, sorting, whathaveyou. JSONQuery is
to JSONPath is to JSON, as XQuery is to XPath is to XML.

  * [The persevere JSONQuery documentation](http://docs.persvr.org/documentation/jsonquery)
  * [Kris Zyp's intro to JSONQuery in dojo](http://www.sitepen.com/blog/2008/07/16/jsonquery-data-querying-beyond-jsonpath/)

Examples
========

For the following examples, suppose there is a file `/tmp/t`, with the
following contents:

      [
        {
          "first"   : "trevor",
          "last"    : "wellington",
          "from"    : "england",
          "age"     : 52,
          "sports"  : [ "rugby", "badmitton", "snooker" ]
        },
        {
          "first"   : "yoni",
          "last"    : "halevi",
          "from"    : "israel",
          "age"     : 26,
          "sports"  : [ "soccer", "windsurfing" ]
        },
        {
          "first"   : "cory",
          "last"    : "parker",
          "from"    : "united states",
          "age"     : 31,
          "sports"  : [ "windsurfing", "baseball", "extreeeeme kayaking" ]
        }
      ]

This is going to be the input JSON text we will use in the examples.

JSON-to-JSON Transformations
----------------------------

These examples transform the input JSON, modifying it and returning the
modified JSON as output on stdout to be piped elsewhere. Transformations of
this type are generally done with a script that follows one of these simple 
patterns:

  1. Modify the `this` object in place (no `return` statement necessary).
  1. Create a replacement object for each record, and `return` it at the end
     of each iteration.

These patterns leave the records in JSON format, and they are automatically
printed to stdout without the use of the `out()` function.

### The Identity Mapping

This is the identity transformation: it doesn't really do anything other
than pass the input straight through.

      cat /tmp/t | jsawk

You should get the input back out, unmolested.

### Increment Everyone's Age

Looks like it's everyone's birthday today. We'll take the JSON input and
increment each object's `age` property, sending the resulting JSON output to
stdout.

      cat /tmp/t | jsawk 'this.age++'

Notice that there is no need to write `return this` in the script. That is
assumed---the runtime does it for you automatically if you don't explicitly
call `return` yourself.

### Flatten The "Sports" Array Of Each Element

Here we modify the input by replacing the `sports` property of each object
in the input array (the `sports` property is itself an array of strings) with
a single string containing all of the person's sports, separated by commas.

      cat /tmp/t | jsawk 'this.sports = this.sports.join(",")'

Notice how altering the `this` object in place alters the result array
accordingly.

### JSON Grep: Select Certain Elements From Input ###

Sometimes you want to use awk to select certain records from the input set,
leaving the rest unchanged. This is like the `grep` pattern of operation. In 
this example we will extract all the records corresponding to people who are 
over 30 years old.

      cat /tmp/t | jsawk 'if (this.age <= 30) return null'

This demonstrates how you can remove records from the results array by
returning a null value from your script.

Aggregate Functions
-------------------

Before and after scripts can be used to manipulate the JSON working set as
a whole, somewhat similar to the way aggregate functions like `SUM()` or
`COUNT()` work in SQL. These types of operations fall under a few basic
patterns.

  1. Use a before script (`-b` option) to do things to the JSON input before 
     transformations are done by the main script.
  2. Use an after script (`-a` option) to do things to the JSON result set
     after all transformations are completed by the main script.

When doing aggregate operations like this, it is likely that you will want to
suppress the JSON result set output, and either use the `-n` option

### Count How Many Elements Are In The Input Array

Here we use an after script to modify the result set, like this:

      cat /tmp/t | jsawk -a 'return this.length'

Notice how the entire results array is replaced by the single number and
printed to stdout.

JSON-to-Text Transformations
----------------------------

In the following examples we will be manipulating the JSON input to
produce text output instead of JSON, for cases where you will be extracting
information from a JSON data source and piping it to non JSON-accepting
processes elsewhere.

### Get A List Of All Sports

This one generates a list of all the sports that are played by the people
in our little JSON list, one per line, without duplicate entries, sorted
alphabetically.

      cat /tmp/t \
        | jsawk -a 'return this.join("\n")' 'return this.sports.join("\n")' \
        | sort -u

Notice the use of JSONQuery to drill down into the JSON objects, an "after"
script to collate the results, and everything piped to the Unix `sort`
tool to remove duplicate entries and do the lexical ordering.  This is 
starting to show the power of the awk-like behavior now.

### Return a Boolean Value

Sometimes you want to just check for a certain condition in a shell script.
Suppose you want to know if there are any people over the age of 50 in the
JSON input array, like this:

      jsawk -n 'if (this.age > 50) quit(1)' < /tmp/t || echo "We have people over 50 here, naptime is in effect."

We suppress normal result set output with `-n` and use the `quit()` function
to return a value in the exit status. The default exit status is, of course,
zero for success.

JSON Pretty-Printing
====================

[Resty](http://github.com/micha/resty] includes a `pp` script will 
pretty-print JSON for you. You just need to install the JSON perl module 
from CPAN.

      GET /blogs.json | jsawk -q '..author' | pp
