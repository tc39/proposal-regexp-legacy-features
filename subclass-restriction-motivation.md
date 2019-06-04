## Why disable legacy RegExp features for proper subclasses of RegExp?

Basically, because it breaks encapsulation.

In ES 2015 + web reality, there are two ways to set the semantics of a regexp:

1. using the constructor (at construction time);
2. using the deprecated `RegExp.prototype.compile()` method.

Similarly, there are two ways to get information about a successful match:

1. reading the returned value of the `RegExp.prototype.exec()` method. (Recall that all other methods that needs to execute a regexp are written in terms of `RegExp.prototype.exec()`);
2. reading the deprecated `RegExp.$1`, etc. static properties.
 
A subclass of `RegExp` may want to redefine the constructor and the `exec()` method, without caring about the legacy features, leaving them as potentially broken.

Below are concrete illustrations of what could be wrong.

### Legacy static properties (RegExp.$1, etc.)

Suppose you write a subclass of RegExp that allows to apply a regular expression simultaneously (to be more correct: successively) to each elements of an array of strings. Then, the RegExp static properties will likely give information about the match against the last string only, which is probably not what is intended.

Another issue with those static properties is their lack of encapsulation. For example, suppose that we evaluate the three following expressions in order:

```js
/(a)/.exec('a')
Object.keys(bar)
RegExp.$1
```

Suppose now that you are running in an environment incorporating a polyfill that emulates symbols using strings of shape `/^symbol-[0-9]{20}-/`; that polyfill could monkey-patch `Object.keys` in order to filter out strings that are of the form of emulated symbols. Then, `RegExp.$1` will likely leak implementation details from that polyfill instead of returning the desired result.

Although that problem is not specific to proper subclasses of RegExp, it may become worse, because the offending expression could be hidden in the implementation of the subclass, e.g., in an `exec()` or `@@replace()` method of the subclass.


### RegExp.prototype.compile()

Suppose you have a subclass of RegExp that supports the `x` flag. Likely, the constructor will rewrite the provided pattern by removing spaces and comments, and forward the modified pattern to the base constructor. However, as currently specified, the `.compile()` method will not give the opportunity to rewrite its provided pattern, as it will not call the constructor. Since it is a nonstandard and deprecated feature, attempting to fix it properly is not worth the trouble.


## Why disable those features for cross-realm regexps

That is, if you apply `RegExp.prototype.exec()`, respectively `RegExp.prototype.compile()`, to a regexp constructed in another realm, then the static properties of RegExp won’t be updated, respectively a TypeError will be thrown.

First, this is really an edge case. Code like `otherRealm_regexp.exec()` is *not* affected, because `otherRealm_regexp` is from the same realm as `otherRealm_regexp.exec`. The issue arises, e.g., in `RegExp.prototype.exec.call(otherRealm_regexp)`.

Now, concerning the static properties of the RegExp constructor: The constructor of which realm should be affected? The realm of the regexp (as thinks Firefox) or the current realm—i.e., the realm of the `.exec()` method—(as think other browsers)? Well, we don’t need to decide: the test that disables the feature for proper subclasses of RegExp will naturally disable it for RegExp objects from other realms. This has the further advantage to prevent different realms from polluting each other.


About the `.compile()` method: The restriction enables one to *really* protect all regexps of a given realm from tampering by doing `delete RegExp.prototype.compile`, because you couldn’t recover a working method from another realm.
