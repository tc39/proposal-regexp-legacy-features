This document aims to describe the main differences between what is described in this repository and what current browsers implement.

This document is most probably incomplete.

# Features that are implemented in at least one mainstream web browser, but not in all

Among the different variations between implementations, we have chosen what we think to be the most reasonable/regular/simple one.

## The legacy static properties of RegExp are accessors

Because it is more natural to spec that way, and because it reflects better the changing nature of those properties.

Implemented at least by Firefox and Chrome.

## The legacy static properties of RegExp have no setter, except for RegExp.input and its alias RegExp.$_

As a consequence, `RegExp.$1 = val` fails silently in slopply mode but loudly in strict mode.

Because loud failure is easier to debug.

Implemented at least by Firefox.

## The legacy static properties of RegExp are configurable/deletable

So that the associated features may be easily removed by deleting the associated API.

This is important for secured environments that want to remove global communication channels.

Implemented at least by Chrome.

## The legacy static properties of RegExp should be nonenumerable

Just for consistency with the rest of ECMA262.

Implemented at least by Chrome.


# Features that are currently not implemented in any mainstream web browser

These features may need to be discussed and gain consensus.

For more detailed motivations, see [subclass-restriction-motivation.md](subclass-restriction-motivation.md)

## All nonstandard legacy features of RegExp (i.e., static properties of RegExp as well as RegExp#compile) are disabled for cross-realm calls

where “cross-realm calls” mean things such as:

* `RegExp.prototype.compile.call(otherRealm_regexp, ...)`
* `RegExp.prototype.exec.call(otherRealm_regexp, ...)`

So that realms do not pollute each others. Or, so that if those features are removed in one realm using `delete RegExp.$1`, etc., they are *really* removed for that realm.

Note that this is a corner case; in particular, it does *not* concern `otherRealm_regexp.compile()`, because the `compile` method is in the same realm as `otherRealm_regexp`.

Currently, Firefox and Chrome have divergent semantics in that situation.

## All nonstandard legacy features of RegExp are disabled for proper subsclasses of RegExp

In short, if the subclass does non-trivial transformations, the legacy features, as currently implemented, have good chances not to work as expected.

