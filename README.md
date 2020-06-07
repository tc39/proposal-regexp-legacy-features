# Legacy RegExp features in JavaScript

## Current status

ECMAScript proposal at stage 3 of the process, see https://github.com/tc39/proposals

See this proposal rendered [here](https://tc39.es/proposal-regexp-legacy-features/)

## Introduction

This is a specification draft for the legacy (deprecated) RegExp features in JavaScript, i.e., static properties of the constructor like `RegExp.$1` as well as the `RegExp.prototype.compile` method.

This does not reflect what the implementations do, but what the editor thinks to be the least bad thing they ought to do in order to maintain web compatibility.

RegExp static properties (currently not part of ECMA 262,see [tc39/ecma262#137](https://github.com/tc39/ecma262/issues/137)) are specified such that:

* The values returned by those properties are updated each time a successful match is done.
* They may be deleted. (This is important for secured environments that want to avoid global side-effects.)

The proposal includes another feature that needs consensus and implementation experience before being specced:

* RegExp legacy static properties as well as RegExp.prototype.compile are disabled for instances of proper subclasses of RegExp as well as for cross-realm regexps. [See the detailed motivation here.](subclass-restriction-motivation.md)

We have attempted to [identify potential risks](web-breaking-hazards.md) induced by the the backward-compatibility break introduced by that feature.

See also [the differences between this spec and the current implementations](changes.md).
