# p2p-db

> An open-ended peer-to-peer database.

## Usage

Let's create a minimal distributed OpenStreetMap database:

```js
// database creation
var p2p = require('p2p-db')
var hyperdb = require('hyperdb')
var ram = require('random-access-memory')

var hyper = hyperdb(ram, { valueEncoding: 'json' })
var db = p2p(hyper)


// add an API module that extends the database with OpenStreetMap primitives
var OsmTypes = require('p2p-db-osm-types')

db.install('osm', OsmTypes())

db.osm.beginChangeset(function (err, id) {
  db.osm.insertNode({
    lat: 14,
    lon: 27,
    changeset: id,
    tags: {}
  })
  db.osm.finishChangeset(id)
})

// add a view for querying points
var GeoStore = require('grid-point-store')
var memdb = require('memdb')
var Spatial = require('p2p-db-point-store')

var geo = GeoStore(memdb())

db.install('geo', Spatial(geo))



// sync with another p2p-db
var db2 = p2p(hyper(ram(), { valueEncoding: 'json' }))

var rs1 = db.replicate()
var rs2 = db2.replicate()

rs1.pipe(rs2).pipe(rs1)

rs1.once('end', onDone)
rs1.once('error', onDone)

function onDone (err) {
  console.log('replication', err ? 'failed' : 'succeeded')

  // query data
  db2.geo.query([[-30, -30], [30, 30]], function (err, nodes) {
    console.log(err, nodes)
  })
}
```

outputs

```
replication succeeded
{ lat: 14, lon: 27, changeset: '52033272934', tags: {} }
```

## API

```js
var p2p = require('p2p-db')
```

### var db = p2p(hyper)

Creates a new p2p-db `db`, using the
[hyperdb](https://github.com/mafintosh/hyperdb) instance `hyper`.

### db.install(name, apiModule)

Installs the API provided by the `apiModule` instance under the object key
`name`. `name` becomes a property of `db`. An error is thrown if there is a name
conflict.

### var ds = db.replicate()

Creates a duplex stream `ds` that can be used to replicate this database with a
p2p-db on the other end of another replication stream.


## Install

With [npm](https://npmjs.org/) installed, run

```
$ npm install p2p-db
```

## See Also

- [flumedb](https://github.com/flumedb/flumedb)
- [hyperdb](https://github.com/mafintosh/hyperdb)

## License

ISC

