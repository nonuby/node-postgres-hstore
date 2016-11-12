node-postgres-hstore
======

* NOTE: Sorry, this is no longer maintained, I am currently not active in node.js space. *

Provides basic hstore parsing and stringify methods (stringify / parse) for 
use with hstore datatype in postgres. Intended to be used with 
node-postgres. The existing solutions on npmjs didnt handle null correctly, or had other
flaws that made it easier to write my own, of course this probably has it own
defects and flaws to, **it will do for now**.

**Note**: this doesnt do any fancy type metadata serialization, so if you need something other
than string -> string/null then hstore isnt a good fit, look at JSON support in
postgresql 9.2/9.3.

integration
=======

**parser**:
there doesnt seem to be a clean hook point, the FAQ refers to a test that 
includes the pgtypes file via a relative path, I couldnt see anything on the
primary pg object that we could use, so path hacking it is.

A simple example of what you may have to do (see exp/test.js):

```javascript
var pg = require('pg');
var hstore = require('../index'); //your code = require('node-postgres-hstore')
var path = require('path');

var conString = "tcp://" + process.env.PGUSER + ":" + 
  process.env.PGPASSWORD + "@localhost/deroku";

// SELECT oid FROM pg_type WHERE typname = 'hstore'

var hstoreOid = 74144; // different for every db

var pgTypes = require(path.join(path.dirname(require.resolve('pg')),'types'));
pgTypes.setTypeParser(hstoreOid, hstore.parse)

var payload = {
  a: 1,
  b: true,
  c: false,
  d: null,
  e: "Matt single' quote",
  f: "Matt double quote\" ",
  g: "/usr/local/bin",
  h: "c:\\windozes\\mshell\\killme"
};

console.log(payload.h)

pg.connect(conString, function(err, client) {
  var query = client.query("SELECT $1::hstore As test", [hstore.stringify(payload)]);
  query.on('row', function(row) {
    console.log(row.test.d)
    console.log(row.test.e)
    console.log(row.test.f)
    console.log(row.test.g)
    console.log(row.test.h)
  });
  query.on('end', function(row) {
    client.end();
    process.exit();
  })
});
```

**serialization:**
node-postgres doesnt have any extension points for serialization, so you have to
do this in your own layers, although it looks like it may come soon [query.js#L130](https://github.com/brianc/node-postgres/blob/master/lib/query.js#L130)

```javascript
var hstore = require('node-postgres-hstore');
var strOutput = hstore.stringify(payload);
```

future - node-postgres integration
=======

As of commit accb94b (07/07/12) the unit tests of node-postgres are failing, I suspect this is part of an
overhaul as they target node v0.8.x. I will revisit once things have stablized
again and attempt to expose a clean way to register parsers, a way to register
serializers, and possibly an explicit method to collect up the metadata (oids) for 
custom types (needs some thought as we dont want to invisibly be hitting the db).
TODO: consult with maintainers of node-postgres

install
=======

cd projectdir  
npm install --save node-postgres-hstore

tests
=======
See test/simple.js  
git clone repo  
npm install --dev .  
npm test  

license
=======
The MIT License 

