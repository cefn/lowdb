# Examples

## CLI

```js
const low = require('lowdb')
const FileSync = require('lowdb/adapters/FileSync')

const adapter = new FileSync('db.json')
const db = low(adapter)

db.defaults({ posts: [] })
  .write()

const result = db.get('posts')
  .push({ title: process.argv[2] })
  .write()

console.log(result)
```

```sh
$ node cli.js hello
# [ { title: 'hello' } ]
```

## Browser

```js
import low from 'lowdb'
import LocalStorage from 'lowdb/adapters/LocalStorage'

const adapter = new LocalStorage('db')
const db = low(adapter)

db.defaults({ posts: [] })
  .write()

// Data is automatically saved to localStorage
db.get('posts')
  .push({ title: 'lowdb' })
  .write()
```

## Server

Please __note__ that if you're developing a local server and don't expect to get concurrent requests, it's often easier to use `file-sync` storage, which is the default.

But if you need to avoid blocking requests, you can do so by using `file-async` storage.

```js
const express = require('express')
const bodyParser = require('body-parser')
const low = require('lowdb')
const FileAsync = require('lowdb/adapters/FileAsync')

// Create server
const app = express()
app.use(bodyParser.json())

// Create database instance and start server
const adapter = new FileAsync('db.json')
low(adapter)
  .then(db => {
    // Routes
    // GET /posts/:id
    app.get('/posts/:id', (req, res) => {
      const post = db.get('posts')
        .find({ id: req.params.id })
        .value()

      res.send(post)
    })

    // POST /posts
    app.post('/posts', (req, res) => {
      db.get('posts')
        .push(req.body)
        .last()
        .assign({ id: Date.now().toString() })
        .write()
        .then(post => res.send(post))
    })

    // Set db default values
    return db.defaults({ posts: [] }).write()
  })
  .then(() => {
    app.listen(3000, () => console.log('listening on port 3000'))
  })
```

## In-memory

With this adapter, calling `write` has the same effect as calling `value`. One use case for this adapter can be for tests.

```js
const fs = require('fs')
const low = require('lowdb')
const FileSync = require('lowdb/adapters/FileSync')
const Memory = require('lowdb/adapters/Memory')

const db = low(
  process.env.NODE_ENV === 'test'
    ? new Memory()
    : new FileSync('db.json')
)

db.defaults({ posts: [] })
  .write()

db.get('posts')
  .push({ title: 'lowdb' })
  .write()
```
