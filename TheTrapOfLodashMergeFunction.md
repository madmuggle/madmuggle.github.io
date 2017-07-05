# The Trap Of Lodash's merge Function
2017/06/14 19:26:00
Javascript


The Javascript's built-in function `Object.assign` can do shallow copy, but sometimes we just need deep copy, so we need to write a merge function ourselves, or use 3rd party librariy like [lodash][lodash].

I chose the lodash's merge function several weeks ago when I was writting some work scripts. Today something unexpected happened, the result of that script just went wrong sometimes.

After some debugging, I found it's the merge function in lodash that caused the wrong result. Lodash's merge function is not suitable for my situation, it treats array in a different way from what I wished.

For example, when you use lodash's `merge` function to merge the following 2 objects:
```js
{ a: { b: [ 1, 2, 3 ] } }
```

```js
{ a: { b: { x: 1 } } }
```

By the definition of `merge`, we can imagine the result should be like:
```js
{ a: { b: { x: 1 } } }
```

But actually this is what you will get:
```js
{ a: { b: [ 1, 2, 3, x: 1 ] } }
```

Weird. So I had to write [my own merge function][github]. Luckily, it's much easier than I have expected.


[lodash]: https://lodash.com/
[github]: https://github.com/madmuggle/DeepMerge

