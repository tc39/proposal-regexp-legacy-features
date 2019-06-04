# Backward compatibility considerations

The design of our semantics is very simple: any attempt to use a feature when it is disabled will throw a TypeError.
However, that needs some careful considerations about potential breaking of the web, especially concerning static properties of RegExp, because the moment when they are updated and the moment when they are read are distinct.

However, recall that although theoretical considerations are useful, only practical experiment could show whether some expected or unexpected failure would occur.

## Mixing deprecated features with brand new features

It is clear that pages that take advantage of RegExp subclassing features *and* of the deprecated features have good chance to break (by design!).
Since subclassing RegExp (in the sense of ES 2015) has been only very recently implemented in browsers, and even not in all stable releases of them, we don't expect that this pattern are yet common on the web. At least, the few web pages that use native RegExp subclassing have good chance to be actively maintained.

So, we will restrict our considerations to legacy pages that don't take advantage of RegExp subclassing features, that is those that don't execute one of those two instructions:

* `class MyRegExp extends RegExp { }`
* `Reflect.construct(RegExp, [], not_RegExp)`

For those pages, we have identified the following potential hazards:

## Cross-realm RegExp.prototype.compile() uses

The following code will fail loudly:

```js
RegExp.prototype.compile.call(otherRealm_regexp, 'foo') // will throw
```

Note that this is distinct from:

```js
otherRealm_regexp.compile('foo') // will work as expected
```

so that we don't expect that such code is common.

## Cross-realm RegExp.prototype.exec() uses

The following code will continue to work:

```js
var result = RegExp.prototype.exec.call(otherRealm_regexp, 'bar') // ok
```

and the following code will fail:

```js
if (RegExp.prototype.exec.call(otherRealm_regexp, 'bar')) {
    RegExp.$1 // will silently fail. (Currently works on Chrome but silently fails on Firefox)
    otherRealm_regexp.constructor.$1 // will silently fail. (Currently works on Firefox but silently fails on Chrome)
}
```

Note that the current semantics differ among browsers, so that we remain inside intersection semantics.

Also, note that this is different from the following, more likely code:

```js
otherRealm_regexp.compile('foo') // will work as expected

if (otherRealm_regexp.exec('foo')) {
    otherRealm_regexp.constructor.$1 // will work as expected
    RegExp.$1 // will silently fail as expected
}
```


## Using the RegExp static properties with unexpected this-value

Because we have specified the RegExp static properties to be accessors that check their this-value, the following code will fail:

```js
var Foo = Object.create(RegExp);
Foo.$1 // will throw
Foo.input = 'bar' // will throw
```

This may be problematic in case of the following pattern:

```js
var originalRegExp = RegExp;

var RegExp = function() {
    // modified version of the RegExp constructor
};

RegExp.__proto__ = originalRegExp; // so that RegExp.$1 is expected to continue to work.
```

It is the only identified risk that the author of the proposal judges as somewhat serious.

A reasonable use case of the above pattern would be as part of a polyfill for ES2015 RegExp semantics. Because latest development versions of web browsers have almost complete support of those new features, that particular case should not be an issue. 

Another possible case to consider is when the getter is extracted:

```js
var get$1 = Object.getOwnPropertyDescriptor(RegExp, '$1').get;
get$1(); // will throw
```

But since not all mainstream browsers have yet implemented those properties as accessors, it is unlikely that it would pose a problem for web compatibility.



