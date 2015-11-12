## Why disable legacy static properties for proper subclasses of RegExp?

Because they may not do what you want.

Suppose you write a subclass of RegExp that allows to apply a regular expression simultaneously (to be more correct: successively) to each elements of an array of strings. Then, the RegExp static properties will likely give information about the match against the last string only, which is probably not what is intended.

## Why disable RegExp.prototype.compile() for proper subclasses of RegExp?

Because they may not do what you want.

Suppose you have a subclass of RegExp that supports the `x` flag. Likely, the constructor will rewrite the provided pattern by removing spaces and comments, and forward the modified pattern to the base constructor. However, as currently specified, the `.compile()` method will not give the opportunity to rewrite its provided pattern, as it will not call the constructor. Since it is a nonstandard and deprecated feature, attempting to fix it properly is not worth the trouble.

## Why disable those features for cross-realm regexpes

That is, if you apply `RegExp.prototype.exec()`, resepcetively `RegExp.prototype.compile()`, to a regexp constructed in another realm, then the static properties of RegExp won’t be updated, respectively a TypeError will be thrown.

First, this is really an edge case. Code like `otherRealm_regexp.exec()` is *not* affected, because `otherRealm_regexp` is from the same realm as `otherRealm_regexp.exec`. The issue arises, e.g., in `RegExp.prototype.exec.call(otherRealm_regexp)`.

Now, concerning the static properties of the RegExp constructor: The constructor of which realm should be affected? The realm of the regexp (as thinks Firefox) or the current realm—i.e., the realm of the `.exec()` method—(as think other browsers)? Well, we don’t need to decide: the test that disables the feature for proper subclasses of RegExp will naturally disable it for RegExp objects from other realms.

For the `.compile()` method: The restriction enables to *really* protect regexpes from tampering by doing `delete RegExp.prototype.compile` at the beginning of the script, as you can’t recover a working method from another realm. Well, it is probably not a very well motivated reason; but again, the test that disable the method for proper subclasses of RegExp will naturally disable it for cross-realm RegExp objects, and it is probably not worth to support that edge case.


## Backward compatibility consideration

We do not expect that the introduction these restrictions will cause much havoc beyond test suites, given that true subclasses of RegExp are not yet common (to say the least!), and that the cross-realm issue is quite obscure.

However, the risk of breakage isn’t nonexistent. Suppose that the RegExp constructor is replaced by a user-defined one using the following pattern:

```js
var OriginalRegExp = RegExp
RegExp = function(/* ... */) { /* ... */ }
RegExp.prototype = new OriginalRegExp
RegExp.__proto__ = OriginalRegExp
```

Then, expressions like `RegExp.rightContext` that used to work as expected in implementations that support prototype mutation via \_\_proto\_\_, will now throw a TypeError.
