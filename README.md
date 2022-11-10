# Legacy RegExp features in JavaScript

## Current status

ECMAScript proposal at stage 3 of the process, see https://github.com/tc39/proposals

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


----

The amendments are relative to the last ECMAScript specification draft found at: https://tc39.github.io/ecma262/
Changes relative to existing algorithms  are marked in **bold**.

All the amendments are part of Annex B, including those that modify objects or algorithm defined in other parts of the spec.

## [%RegExp%](https://tc39.github.io/ecma262/#sec-regexp-constructor)

The %RegExp% intrinsic object, which is the builtin RegExp constructor, has the following additional internal slots:

* [[RegExpInput]]
* [[RegExpLastMatch]]
* [[RegExpLastParen]]
* [[RegExpLeftContext]]
* [[RegExpRightContext]]
* [[RegExpParen1]]
* [[RegExpParen2]]
* [[RegExpParen3]]
* [[RegExpParen4]]
* [[RegExpParen5]]
* [[RegExpParen6]]
* [[RegExpParen7]]
* [[RegExpParen8]]
* [[RegExpParen9]]

The initial value of all these internal slots is the empty String.


## [RegExpAlloc ( _newTarget_ )](https://tc39.github.io/ecma262/#sec-regexpalloc)

RegExp instances have an additional slot which optionally keeps a reference to its constructor. It is used for deciding whether a nonstandard legacy feature is enabled for that regexp. The RegExpAlloc abstract operation is modified as follows:

1. Let _obj_ be ? OrdinaryCreateFromConstructor(_newTarget_, "%RegExpPrototype%", «[[RegExpMatcher]], [[OriginalSource]], [[OriginalFlags]], **[[Realm]]**, **[[LegacyFeaturesEnabled]]**»).
1. **Let _thisRealm_ be the current Realm Record.**
1. **Set the value of _obj_’s [[Realm]] internal slot to _thisRealm_.**
1. **If SameValue(_newTarget_, _thisRealm_.[[Intrinsics]].[[%RegExp%]]) is true, then**
    1. **Set the value of _obj_’s [[LegacyFeaturesEnabled]] internal slot to true.**
1. **Else,**
    1. **Set the value of _obj_’s [[LegacyFeaturesEnabled]] internal slot to false.**
1. Perform ! DefinePropertyOrThrow(_obj_, "lastIndex", PropertyDescriptor {[[Writable]]: true, [[Enumerable]]: false, [[Configurable]]: false}).
1. Return _obj_.


## [RegExpBuiltInExec ( _R_, _S_ )](https://tc39.github.io/ecma262/#sec-regexpbuiltinexec)

In the RegExpBuiltInExec abstract operation, a hook is added for updating the static properties of %RegExp% after a successful match. The last three steps of the algorithm are modified as follows:

1. ...
1. (current step 23) Perform ! CreateDataProperty(_A_, "0", _matchedSubstr_).
1. **Let _capturedValues_ be an new empty List.**
1. (current step 24) For each integer _i_ such that _i_ > 0 and _i_ ≤ _n_
    1. ...
    1. (current step 24.e) Perform ! CreateDataProperty(_A_, ToString(_i_) , _capturedValue_).
    1. **Append _capturedValue_ to the end of _capturedValues_.** 
1. **Let _thisRealm_ be the current Realm Record.**
1. **Let _rRealm_ be the value of _R_’s [[Realm]] internal slot.**
1. **If SameValue(_thisRealm_, _rRealm_) is true, then**
    1. **If the value of _R_’s [[LegacyFeaturesEnabled]] internal slot is true, then**
        1. **Perform UpdateLegacyRegExpStaticProperties(%RegExp%, _S_, _lastIndex_, _e_, _capturedValues_).**
    1. **Else,**
        1. **Perform InvalidateLegacyRegExpStaticProperties(%RegExp%).**
1. (current step 25) Return _A_.


## UpdateLegacyRegExpStaticProperties ( _C_, _S_, _startIndex_, _endIndex_, _capturedValues_ )

The abstract operation UpdateLegacyRegExpStaticProperties updates the values of the static properties of %RegExp% after a successful match.

1. Assert: _C_ is an Object that has a [[RegExpInput]] internal slot.
1. Assert: Type(_S_) is String.
1. Let _len_ be the number of code units in _S_.
1. Assert: _startIndex_ and _endIndex_ are integers such that 0 ≤ _startIndex_ ≤ _endIndex_ ≤ _len_.
1. Assert: _capturedValues_ is a List of Strings.
1. Let _n_ be the number of elements in _capturedValues_.
1. Set the value of _C_’s [[RegExpInput]] internal slot to _S_.
1. Set the value of _C_’s [[RegExpLastMatch]] internal slot to a String whose length is _endIndex_ - _startIndex_ and containing the code units from _S_ with indices _startIndex_ through _endIndex_ - 1, in ascending order.
1. If _n_ > 0, set the value of _C_’s [[RegExpLastParen]] internal slot to the last element of _capturedValues_.
1. Else, set the value of _C_’s [[RegExpLastParen]] internal slot to the empty String.
1. Set the value of _C_’s [[RegExpLeftContext]] internal slot to a String whose length is _startIndex_ and containing the code units from _S_ with indices 0 through _startIndex_ - 1, in ascending order.
1. Set the value of _C_’s [[RegExpRightContext]] internal slot to a String whose length is _len_ - _endIndex_ and containing the code units from _S_ with indices _endIndex_ through _len_ - 1, in ascending order.
1. For each integer _i_ such that 1 ≤ _i_ ≤ 9
    1. If _i_ ≤ _n_, set the value of _C_’s [[RegExpParen<i>i</i>]] internal slot to the <i>i</i>th element of _capturedValues_.
    1. Else, set the value of _C_’s [[RegExpParen<i>i</i>]] internal slot to the empty String.

## InvalidateLegacyRegExpStaticProperties ( _C_)

The abstract operation InvalidateLegacyRegExpStaticProperties marks the values of the static properties of %RegExp% as non-available.

1. Assert: _C_ is an Object that has a [[RegExpInput]] internal slot.
1. Set the value of the following internal slots of _C_ to **empty**:
  * [[RegExpInput]]
  * [[RegExpLastMatch]]
  * [[RegExpLastParen]]
  * [[RegExpLeftContext]]
  * [[RegExpRightContext]]
  * [[RegExpParen1]]
  * [[RegExpParen2]]
  * [[RegExpParen3]]
  * [[RegExpParen4]]
  * [[RegExpParen5]]
  * [[RegExpParen6]]
  * [[RegExpParen7]]
  * [[RegExpParen8]]
  * [[RegExpParen9]]

## Additional properties of the RegExp constructor

All the below properties are accessor properties who have the attributes { [[Enumerable]]: false, [[Configurable]]: true }. Moreover, for the properties whose setter is not explicitely defined, the [[Set]] attribute is set to undefined.

The accessors check for their this value, so that the properties do not appear to be inherited by subclasses.

### Abstract operations

#### GetLegacyRegExpStaticProperty( _C_, _thisValue_, _internalSlotName_ ).
The abstract operation GetLegacyRegExpStaticProperty is used when retrieving a value from a legacy RegExp static property.

1. Assert _C_ is an object that has an internal slot named _internalSlotName_.
2. If SameValue(_C_, _thisValue_) is false, throw a TypeError exception.
3. Let _val_ be the value of the internal slot of _C_ named _internalSlotName_.
4. If _val_ is **empty**, throw a TypeError exception.
3. Return _val_.

#### SetLegacyRegExpStaticProperty( _C_, _thisValue_, _internalSlotName_, _val_ ).
The abstract operation SetLegacyRegExpStaticProperty is used when assigning a value to a legacy RegExp static property.

1. Assert _C_ is an object that has an internal slot named _internalSlotName_.
2. If SameValue(_C_, _thisValue_) is false, throw a TypeError exception.
3. Let _strVal_ be ? ToString(_val_).
4. Set the value of the internal slot of _C_ named _internalSlotName_ to _strVal_.

### RegExp.input
#### get RegExp.input

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpInput]]).

#### set RegExp.input = _val_ 

1. Perform ? SetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpInput]], _val_).

### RegExp.$_
#### get RegExp.$_

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpInput]]).

#### set RegExp.$_ =  _val_ 

1. Perform ? SetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpInput]], _val_).


### get RegExp.lastMatch

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpLastMatch]]).

### get RegExp.$&

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpLastMatch]]).

### get RegExp.lastParen

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpLastParen]]).

### get RegExp.$+

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpLastParen]]).

### get RegExp.leftContext

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpLeftContext]]).

### get RegExp.$`

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpLeftContext]]).

### get RegExp.rightContext

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpRightContext]]).

### get RegExp.$'

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpRightContext]]).

### get RegExp.$1

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpParen1]]).

### get RegExp.$2

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpParen2]]).

### get RegExp.$3

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpParen3]]).

### get RegExp.$4

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpParen4]]).

### get RegExp.$5

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpParen5]]).

### get RegExp.$6

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpParen6]]).

### get RegExp.$7

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpParen7]]).

### get RegExp.$8

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpParen8]]).

### get RegExp.$9

1. Return ? GetLegacyRegExpStaticProperty(%RegExp%, this value, [[RegExpParen9]]).


## [RegExp.prototype.compile ( _pattern_, _flags_ )](https://tc39.github.io/ecma262/#sec-regexp.prototype.compile)

The modification below will disable RegExp.prototype.compile for objects that are not direct instances of RegExp as well as in case of mismatch between realms.

1. Let _O_ be the this value.
1. If Type(_O_) is not Object or Type(_O_) is Object and _O_ does not have a [[RegExpMatcher]] internal slot, then
    1. Throw a TypeError exception.
1. **Let _thisRealm_ be the current Realm Record.**
1. **Let _oRealm_ be the value of _O_’s [[Realm]] internal slot.**
1. **If SameValue(_thisRealm_, _oRealm_) is false, throw a TypeError exception.**
1. **If the value of _R_’s [[LegacyFeaturesEnabled]] internal slot is false, throw a TypeError exception.**
1. If Type(_pattern_) is Object and _pattern_ has a [[RegExpMatcher]] internal slot, then
    1. If _flags_ is not undefined, throw a TypeError exception.
    1. Let _P_ be the value of _pattern_’s [[OriginalSource]] internal slot.
    1. Let _F_ be the value of _pattern_’s [[OriginalFlags]] internal slot.
1. Else,
    1. Let _P_ be _pattern_.
    1. Let _F_ be _flags_.
1. Return ? RegExpInitialize(_O_, _P_, _F_).

