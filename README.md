## About [![Build Status](https://travis-ci.org/biggora/caminte.png?branch=master)](https://travis-ci.org/biggora/caminte)

CaminteJS is cross-db ORM for nodejs, providing common interface to access
most popular database formats.

CaminteJS adapters:
mongoose, mysql, sqlite3, riak, postgres, couchdb, mongodb, redis, neo4j, firebird, nano

## Usage

```javascript
var Schema = require('caminte').Schema;
var schema = new Schema('redis2', {port: 6379}); // port number depends on your configuration
// define models
var Post = schema.define('Post', {
    title:     { type: String, length: 255 },
    content:   { type: Schema.Text },
    date:      { type: Date,    default: Date.now },
    published: { type: Boolean, default: false, index: true }
});

// simplier way to describe model
var User = schema.define('User', {
    name:         String,
    bio:          Schema.Text,
    approved:     Boolean,
    joinedAt:     Date,
    age:          Number
});

// define any custom method
User.prototype.getNameAndAge = function () {
    return this.name + ', ' + this.age;
};

// models also accessible in schema:
schema.models.User;
schema.models.Post;

// setup relationships
User.hasMany(Post,   {as: 'posts',  foreignKey: 'userId'});
// creates instance methods:
// user.posts(conds)
// user.posts.build(data) // like new Post({userId: user.id});
// user.posts.create(data) // build and save

Post.belongsTo(User, {as: 'author', foreignKey: 'userId'});
// creates instance methods:
// post.author(callback) -- getter when called with function
// post.author() -- sync getter when called without params
// post.author(user) -- setter when called with object

schema.automigrate(); // required only for mysql NOTE: it will drop User and Post tables

// work with models:
var user = new User;
user.save(function (err) {
    var post = user.posts.build({title: 'Hello world'});
    post.save(console.log);
});

// or just call it as function (with the same result):
var user = User();
user.save(...);

// Common API methods

// just instantiate model
new Post
// save model (of course async)
Post.create(cb);
// all posts
Post.all(cb)
// all posts by user
Post.all({where: {userId: user.id}, order: 'id', limit: 10, skip: 20});
// the same as prev
user.posts(cb)
// get one latest post
Post.findOne({where: {published: true}, order: 'date DESC'}, cb);
// same as new Post({userId: user.id});
user.posts.build
// save as Post.create({userId: user.id}, cb);
user.posts.create(cb)
// find instance by id
User.find(1, cb)
// count instances
User.count([conditions, ]cb)
// destroy instance
user.destroy(cb);
// destroy all instances
User.destroyAll(cb);


// Setup validations
User.validatesPresenceOf('name', 'email')
User.validatesLengthOf('password', {min: 5, message: {min: 'Password is too short'}});
User.validatesInclusionOf('gender', {in: ['male', 'female']});
User.validatesExclusionOf('domain', {in: ['www', 'billing', 'admin']});
User.validatesNumericalityOf('age', {int: true});
User.validatesUniquenessOf('email', {message: 'email is not unique'});

user.isValid(function (valid) {
    if (!valid) {
        user.errors // hash of errors {attr: [errmessage, errmessage, ...], attr: ...}
    }
})

```

## Callbacks

The following callbacks supported:

    - afterInitialize
    - beforeCreate
    - afterCreate
    - beforeSave
    - afterSave
    - beforeUpdate
    - afterUpdate
    - beforeDestroy
    - afterDestroy
    - beforeValidation
    - afterValidation

Each callback is class method of the model, it should accept single argument: `next`, this is callback which
should be called after end of the hook. Except `afterInitialize` because this method is syncronous (called after `new Model`).

## Object lifecycle:

```javascript
var user = new User;
// afterInitialize
user.save(callback);
// beforeValidation
// afterValidation
// beforeSave
// beforeCreate
// afterCreate
// afterSave
// callback
user.updateAttribute('email', 'email@example.com', callback);
// beforeValidation
// afterValidation
// beforeUpdate
// afterUpdate
// callback
user.destroy(callback);
// beforeDestroy
// afterDestroy
// callback
User.create(data, callback);
// beforeValidate
// afterValidate
// beforeCreate
// afterCreate
// callback
```

Read the tests for usage examples: ./test/common_test.js
Validations: ./test/validations_test.js

## Your own database adapter

To use custom adapter, pass it's package name as first argument to `Schema` constructor:

    mySchema = new Schema('couch-db-adapter', {host:.., port:...});

Make sure, your adapter can be required (just put it into ./node_modules):

    require('couch-db-adapter');

## Running tests

To run all tests (requires all databases):

    npm test

If you run this line, of course it will fall, because it requres different databases to be up and running,
but you can use js-memory-engine out of box! Specify ONLY env var:

    ONLY=memory nodeunit test/common_test.js

of course, if you have redis running, you can run

    ONLY=redis nodeunit test/common_test.js

## Package structure

Now all common logic described in `./lib/*.js`, and database-specific stuff in `./lib/adapters/*.js`. It's super-tiny, right?

## Contributing

If you have found a bug please write unit test, and make sure all other tests still pass before pushing code to repo.

## License

(The MIT License)

Copyright (c) 2012 Aleksej Gordejev <aleksej@gordejev.lv>

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


## Resources

- Visit the [author website](http://www.gordejev.lv).
- Follow [@biggora](https://twitter.com/#!/biggora) on Twitter for updates.
- Report issues on the [github issues](https://github.com/biggora/caminte/issues) page.