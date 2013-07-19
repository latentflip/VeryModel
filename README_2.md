# VeryModel

VeryModel is a standalone model layer for creating, validating and editing object models in JavaScript. It is independent of a specific framework and can be used to integrate with your own persistence layer, etc.

Models are useful for managing the lifecycle of an object: initialization and creation, validation of object properties, and persistence of the model to some form of data store. Many frameworks and ORMs have their own model layers (e.g. the Model layer in Backbone.js), VeryModel gives you enough functionality to manage models without tying you to a specific framework

## Installation

`npm install VeryModel`

## Examples

### Defining Models

We define a model class by instantiating a `new VeryModel`, passing a model definition object to the constructor. For example a user model might be defined as:

```javascript
var User = new VeryModel({
    id: { primary: true, default: 1},
    username: { required: true, default: ''},
    password: { required: false, default: ''}
});
```

We can then initialize instances of our user model, and convert them back to plain javascript objects:

```javascript
// Create a user with default attributes:
var user = User.create();
user.toJSON() //=> { id: 1, username: '', password: '' }

// Create a user with initialized attributes:
var user = User.create({ username: 'bob', password: 'supersecret' })
user.toJSON() //=> { id: 1, username: 'bob', password: 'supersecret' }
```

### Validations

Validating model properties is a key feature of a model system. For example, we might require that our model's username property only consists of letters and numbers, and our password is at least 6 characters long.

We can define model validations using the VeryType class. VeryType has a number of methods which we can use to specify the requirements for a field. For example:

```javascript
var User = new VeryModel({
    //id field must me alphanumeric
    id: { primary: true, type: VeryType().isAlphanumeric(), default: 1 },   
    //username must be alphanumeric and between 4 and 25 chars
    username: { required: true, type: VeryType().isAlphanumeric().len(45, 25), default: '' },

    //password must be strong and at least 6 chars
    password: { required: false, type: VeryType().len(6).custom(strongPasswordCheck)}, default: '' }
})

function strongPasswordCheck(password) {
    //return true if password is strong or false if not
    //TODO: implement strong password check
    return true
}
```

With validation in place, we can have the model check it's validations using the `doValidate` method which returns a list of errors on te model:

```javascript
var user = User.create({ username: "Bob", password: "pass" })

var errors = user.doValidate();

console.log(errors); //=>
```

### Saving and retrieving models

VeryModel allows us to define our `save` and `load` methods on our model class to allow us to plug our VeryModels into custom data stores.

We define a `save` method for our model by calling the `setSave` function on our VeryModel class with a save function, and similarly for a `load` function. We can then call these functions with `doSave` and `doLoad` respectivly.

For example, to integrate with a riak database, we might define our save  like so:

```javascript
User.setSave(function(model, cb) {

    //remove our password field before saving, and instead
    //assign our private passhash field which the db is expecting
    if (model.password) {
        model.passhash = sha1(model.password + 'some salt');
        model.password = undefined;
    }
  
    //use riak to persist the model
    riak.put(model.id, model.toJSON({withPrivate: true}), cb);
});

var user = User.create({ username: "Peter", password: "my_super_secret" });

user.doSave();
```
