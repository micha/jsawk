Quick Start
===========

You need to have the `js` interpreter installed. On a Debian system you can
install it with the command `apt-get install spidermonkey-bin`. Or you can 
build it from source from [here](http://www.mozilla.org/js/spidermonkey/).
Okay? Okay. Ready to go.

Install It
----------

First, get the `jsawk` script:

      curl http://github.com/micha/jsawk/raw/master/jsawk > jsawk

Then make it executable and put it somewhere in your path:

      chmod 755 jsawk && mv jsawk ~/bin/

Use It
------

Now you can do some stuff with JSON data. Here's an example using data from
a REST service that serves JSON (we use [resty](http://github.com/micha/resty)
to do the HTTP requests):

      resty http://example.com:8080/data*.json
      GET /people/47 | jsawk 'this.favoriteColor = "blue"' | PUT /people/47

This would do a `GET` request on the resource `/data/people/47.json`, which
would result in a JSON object. Then `jsawk` takes the JSON via stdin and for
each JSON object it runs the little snippet of JavaScript, setting the
`favoriteColor` property to `"blue"`, in this case. The modified JSON is then
output via stdout to `resty` again, which does the `PUT` request to update
the resource.

Usage
=====

**jsawk [**options**] [**script**]**

### OPTIONS ###

  * **-h** <br />
    Print short help page and exit.
  * **-n** <br />
    Suppress printing of JSON result set.
  * **-d \<delim\>** <br />
    Choose a token to indicate end of input. If this token appears on a
    line by itself then reading of JSON input will end. Otherwise, 10 blank
    lines in a row will end input. (Don't look at me, this is a spidermonkey
    issue.) Default delimiter is a single period: `.`.
  * **-q \<query\>** <br />
    Filter JSON through the specified 
    [JSONQuery](http://docs.persvr.org/documentation/jsonquery) query.
  * **-f \<file\>** <br />
    Load and run the specified JavaScript file prior to
    processing JSON. This option can be specified multiple times to load
    multiple JavaScript libraries.
  * **-b \<script\>** <br />
    **-a \<script\>** <br />
    Run the specified snippet of JavaScript before (**-b**) or after (**-a**)
    processing JSON input. The `this` object is set to the whole JSON array
    or object.

### SCRIPT ###

This is a snippet of JavaScript that will be run on each element of the
input array, if input is a JSON array, or on the object if it's an object.
The `this` object is set to the current array element or object.

Scripting Environment
=====================

### JSONQuery ###

  * **$(**_query_**, **_thing_**)** <br />
    Print arguments (JSON encoded, if necessary) to stderr. <br />
    **params:** _thing_ Object, Array, or String. <br />
    **return:** void.

### Input/Output ###

  * **err(**_thing_**)** <br />
    Print arguments (JSON encoded, if necessary) to stderr. <br />
    **params:** _thing_ Object, Array, or String. <br />
    **return:** void.
  * **out(**_thing_**)** <br />
    Print arguments (JSON encoded, if necessary) to stdout. <br />
    **params:** _thing_ Object, Array, or String. <br />
    **return:** void.

Errors and Output
=================

Exit Status
===========

JSON Pretty-Printing
====================

[Resty](http://github.com/micha/resty] includes a `pp` script will 
pretty-print JSON for you. You just need to install the JSON perl module 
from CPAN.

      GET /blogs.json | jsawk -q '..author' | pp
