Quick Start
===========

You need to have the `js` interpreter installed. On a Debian system you can
install it with the command `apt-get install spidermonkey-bin`. Or you can 
build it from source from (here)[http://www.mozilla.org/js/spidermonkey/].
Okay? Okay. Ready to go.

First, jsawk script:

      curl http://github.com/micha/jsawk/raw/master/jsawk > jsawk

Then make it executable and put it somewhere in your path:

      chmod 755 jsawk && mv jsawk ~/bin/

Now you can do some stuff with JSON data. Here's an example using data from
a REST service that serves JSON (we use [resty](http://github.com/micha/resty]
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

      jsawk [OPTIONS] [SCRIPT]

      Where OPTIONS are:

Options
-------

  * **-h** Print short help page and exit.
  * **-n** Suppress printing of JSON result set.
  * **-q <query>** Filter JSON through the specified 
    [JSONQuery](http://docs.persvr.org/documentation/jsonquery) query.
  * **-f <file>** Load and run the specified JavaScript file prior to
    processing JSON. This option can be specified multiple times to load
    multiple JavaScript libraries.

Script
------

This is a snippet of JavaScript that will be run 

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
