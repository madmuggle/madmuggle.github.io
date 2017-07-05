# The Trap Of Node.js Mongodb Library
2017/05/22 16:38:00
Node.js MongoDB


## MongoDB and Javascript

MongoDB uses Javascript expressions to do regular manipulations like `insert`, `remove`, `update` and `find`. But it is not a general purpose Javascript environment. The basic data types are different sometimes.

`String`, `Array`, `Object` are the same in both general purpose JS environment and MongoDB environment. But other types like `Date` are different.

In general purpose JS environment, you create a timestamp object like this:

```js
{ mydate: new Date() }
```

In MongoDB, you need to use `ISODate` to do the same thing:

```js
{ mydate : ISODate("2017-05-22T16:20:56.012+08:00") }
```

This case is simple, once you write `new Date`, everyone knows you want a timestamp. So Node.js's library `mongodb` will create an `ISODate` object automatically.

But for some situations, you can't count on the library to understand what you really mean.


## Creating ObjectId

`ObjectID` is a concept for MongoDB, there is no relative thing in Node.js. You may want to use String to represent ObjectID, but MongoDB will treat it simply as String, not ObjectID.

To represent an ObjectID, you can use the `mongodb.ObjectID` function like this:

```js
mongodb.ObjectId("563060fa77b984462a9c545b");
//=> 563060fa77b984462a9c545b
```

The problem is `mongodb.ObjectId` is not just used for converting String to ObjectId. It is also used to create new ObjectId. If you give it invalid arguments like `null` or `undefined`, it will ignore them and create a new ObjectId.

```js
mongodb.ObjectId();
//=> 5922a35a5607a949507a2f65
```

I called it a "problem" because it did caused some problems for me.

Imagine the situation that you want to insert some data that contains an ObjectId, and you get the ObjectId string from other data resources. When the ObjectId string in is `null`, which means there is **NO** such data, and you pass it to `mongodb.ObjectId` directly, what will happen?

It will create a new ObjectId and assign it to the target key. Just like you got a normal ObjectId string from your input.

How misleading it is!


## My Solution

Always check the string you pass to `mongodb.ObjectId`.

