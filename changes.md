This document aims to describe the main differences between what is described in this repository and what current browsers implement.

This document is probably incomplete.

# Features that are implemented in at least one mainstream web browser, but not in all

Among the different variations between implementations, we have chosen what we think to be the most reasonable/regular/simple one.

## The legacy static properties of RegExp are accessors

Because it is more natural to spec that way especially if we want to make them deletable, and because it reflects better the changing nature of those properties.

Implemented by Firefox and Chrome.

## The legacy static properties of RegExp have no setter, except for RegExp.input and its alias RegExp.$_

As a consequence, `RegExp.$1 = val` fails silently in sloppy mode but loudly in strict mode.

Because loud failure is easier to debug.

Implemented by Firefox. Functionaly equivalent semantics (nonwritable properties) implemented by Safari.

## The legacy static properties of RegExp are configurable and deletable

So that the associated features may be easily removed by deleting the associated API.

This is important for secured environments that want to remove global communication channels.

Implemented by Chrome.

## The legacy static properties of RegExp should be nonenumerable

Just for consistency with the rest of ECMA262.

Implemented by Chrome.

# Features that are currently not implemented in any mainstream web browser

These features may need to be discussed and gain consensus.

For more detailed motivations, see [subclass-restriction-motivation.md](subclass-restriction-motivation.md)

## All nonstandard legacy features of RegExp (i.e., static properties of RegExp as well as RegExp#compile) are disabled for cross-realm calls

where “cross-realm calls” mean things such as:

* `RegExp.prototype.compile.call(otherRealm_regexp, ...)`
* `RegExp.prototype.exec.call(otherRealm_regexp, ...)`

So that realms do not pollute each others. Or, so that if those features are removed in one realm using `delete RegExp.$1`, etc., they are *really* removed for that realm.

Note that this is a corner case; in particular, it does *not* concern `otherRealm_regexp.compile()`, because the `compile` method is from the same realm as `otherRealm_regexp`.

Currently, Firefox and Chrome have divergent semantics in that situation.

## All nonstandard legacy features of RegExp are disabled for proper subsclasses of RegExp

In short, if the subclass does non-trivial transformations, the legacy features, as currently implemented, have good chances not to work as expected.


# Features not described here and implemented by some browsers only

Those features are considered as not needed for web compatibility and therefore are not part of the proposal.

## RegExp.index, RegExp.lastIndex

Respectively the start and the end position in the string of the last succesful match.

Implemented by Edge.

## RegExp.multiline, RegExp.$*

A boolean flag that, when set to true, forces new regexps to have the multiline flag.

Implemented by Firefox, but intended to be removed in v48. The property is present in Safari but is nonfunctional.

## Miscellaneous

* Old versions of Firefox (until v44) had a mechanism to restore previous values of RegExp static properties in some situations.
* For some methods (e.g., String#split), the RegExp static properties are typically not updated.
* For some methods (e.g., String#replace used with a callback), the moment when the RegExp static properties are updated is observably different accross implementations.

See [bugzilla@mozilla bug:1208835#c1](https://bugzilla.mozilla.org/show_bug.cgi?id=1208835#c1) for a testcase illustrating the behaviour of different implementations w.r.t. RegExp#replace. (For this particular testcase, our proposal will lead to the same result as Chrome and Safari.)

We have preferred to keep the spec simple rather than trying to be smart or to mimic some implementation: that is to say, we have just specified what is observable for RegExp#exec; and for other methods working with regexps we have relied on the fact that they are specified in terms of RegExp#exec since ES 2015.

