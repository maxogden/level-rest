# level-rest [![Build Status](https://secure.travis-ci.org/shama/level-rest.png?branch=master)](http://travis-ci.org/shama/level-rest) [![unstable](http://badges.github.io/stability-badges/dist/unstable.svg)](http://github.com/badges/stability-badges)

> A REST adapter for [LevelUP](https://github.com/rvagg/node-levelup)

[![NPM](https://nodei.co/npm/level-rest.png?compact=true)](https://nodei.co/npm/level-rest/)

## Example

```js
var levelup = require('levelup')
var LevelREST = require('level-rest')

var db = levelup('./mydb', {valueEncoding: 'json'})
var rest = new LevelREST(db, {
  id: 'id',
  metaKey: 'meta',
  serialize: function(url) {
    url = url.split('/')
    return { api: url[0], id: url[1] }
  },
})

// POST data in
rest.post('people', { id: 1, name: 'Kyle' }, function() {
  console.log('Saved!')
})

// GET all people
rest.get('people').on('data', function(data) {
  data.people.forEach(function(person) {
    console.log('Hi ' + person.name)
  })
})

// GET one person by id
rest.get('people/1').on('data', function(data) {
  console.log('Hi ' + data.person.name)
})

// PUT data into one person
rest.put('people/1', { name: 'Dude' }, function() {
  console.log('Person 1 is now named Dude')
})

// DELETE a person
rest.delete('people/1', function() {
  console.log('Person 1 has been deleted')
})
```

### Everything Streams

The primary goal of this adapter is to be able to stream requests in to leveldb and stream out to a response.

Here is an example using [Jaws](https://www.npmjs.org/package/jaws):

```js
// Create a REST interface to our LevelDB
var rest = new LevelREST(levelup('./mydb', {valueEncoding: 'json'}))

// Create a server
var app = jaws()

// Wire up API endpoints
app.route('/api/:endpoint/:id?', function(req, res) {
  var id = req.route.params.id || ''
  var endpoint = req.route.params.endpoint
  var method = req.method.toLowerCase()
  req.pipe(rest[method](endpoint + '/' + id)).pipe(res)
})
```

### With Relationships

> Relationships are a work in progress

```js
rest.get('people').belongsTo('profiles').on('data', function(data) {
  /*
  data = {
  }
  */
})
```

## License
Copyright (c) 2014 Kyle Robinson Young  
Licensed under the MIT license.
