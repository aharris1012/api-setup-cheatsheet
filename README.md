cheatsheet.md
# Node API CheatSheet

## Initial set up with npm
1. $ npm i -y
2. $ npm i express knex cors helmet bcryptjs jsonwebtoken axios dotenv sqlite3
3. $ npm i -D nodemon jest supertest

## Setup File Structures  
This is primarily personal preference, but this is where you'll plan how to structure your folders and files
* api
  * api-router.js
  * configure-middleware.js
  * server.js
* auth
  * auth-router.js
* database
  * migrations { Sub Folder } 
  * seeds { Sub Folder }
  * db3 file
  * dbConfig.js
* users 
  * users-model.js
  * users-router.js
- .env
- knexfile.js
- index.js
## Start Your Server
Configure index.js to start your server
```javascript
  require('dotenv').config();
  const server = require('./api/server'); // <-- The magic will come from this file 🧙🏾‍♂️
  const PORT = process.env.PORT || 9000
  server.listen(PORT, console.log(`\n*** IT'S OVER ${PORT} ***\n`));
```
> ./api/server.js
```javascript
  const express = require('express');
  const apiRouter = require('./api-router');
  const configureMiddleware = require('./configure-middleware'); 
  const server = express();
  configureMiddleware(server)
  server.use('/api', apiRouter); // after the api endpoint is reached, activate "apiRouter"
  module.exports = server
```
## Knex Init
> $ knex init will create your knexfile.js
```javascript
  module.exports = {
  development: {
    client: 'sqlite3',
    useNullAsDefault: true,
    connection: {
      filename: './database/auth.db3',
    },
    pool: {
      afterCreate: (conn, done) => {
        conn.run('PRAGMA foreign_keys = ON', done); // <-- enforce foreign keys 🔑
      },
    },
    migrations: {
      directory: './database/migrations', // <-- migrations directory 
    },
    seeds: {
      directory: './database/seeds', // <-- seeds directory 
    },
  },
};
```

## Configure dbConfig.js
```javascript
  const knex = require('knex');
  const knexfile = require('../knexfile');
  const environment = process.env.NODE_ENV || 'development';
  module.exports = knex(knexfile[environment])
```
## Create a table
Our data will need structure to lets provide some.
> $ knex migrate:make tableName
1. This will create a timestamped file name, which will migrate the "latest" when we run migrations
2. This will create a barebones migration for your table, here is an example of a users table with a username and password column
3. Also don't forget to drop your exports.down as well to let knex know which table to delete.

```javascript
  exports.up = function(knex) {
    return knex.schema.createTable('users', users => {
      users.increments();
      users
        .string('username', 128)
        .notNullable()
        .unique();
      users.string('password', 128).notNullable();
    });
  };
  exports.down = function(knex, Promise) {
    return knex.schema.dropIfTableExists('users');
};
```

## Knex Seeds
> $ knex seed:make 000-cleanup
This is necessary to keep our tables and data clean on each run
database/seeds/000-cleanup.js
```javascript
  const cleaner = require('knex-cleaner');
  exports.seed = function(knex) {
    // Deletes ALL existing entries
    return cleaner.clean(knex, {
      mode: 'truncate',
      ignoreTables: ['knex_migrations', 'knex_migrations_lock'] // don't empty migration tables <--
    });
  };
```

> $ knex seed:make 001-users
Give your db some seed data, make sure the key value pairs match your tables data structure
```javascript
  exports.seed = function(knex) {
  // 000-cleanup.js deleted the data from all the tables, we only need to insert the data
  return knex('users').insert([
    { username: 'ace', password: 'paid' },
    { username: 'mitch', password: 'in' },
    { username: 'rico', password: 'full' }
  ]);
};
```
## Data Manipulation and Helper functions
Time to make our db useful, know that these functions are very similar to SQL statements. Here is an example of a model function
Models are the bnb (bread n butter) of the knex/sql db. These are JS statements that compile to SQL, the syntax is nearly identical
```javascript
  async function add(user) {
  const [id] = await db('users').insert(user)
  return findById(id);
}
```
Once you have your helper functions you can have several options but whichever you choose must be tied into your routes
```javascript
  router.post('/register',  (req, res) => {
  let user = req.body
  // hash the password
  const hash = bcrypt.hashSync(user.password, 14)
  // override the plain text password with the hash
  user.password = hash
  Users.add(user)
    .then(saved => {
      res.status(201).json(saved)
    })
    .catch(error => res.status(500).json(error))
})
```
## 🏁 You should now have a working Node base CRUD handling DB. 🏁