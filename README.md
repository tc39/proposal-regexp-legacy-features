# Legacy static properties of the RegExp constructor in JavaScript

This is a specification draft for the RegExp legacy static properties in JavaScript.

This does not reflect what the implementations do, but what the editor thinks what the least bad thing they ought to do in order to maintain web compatibility.

* The values returned by those properties are updated each time a successful match is done.
* They are updated only for direct instances of RegExp.
* They may be deleted.


The amendments are relative to the last ECMAScript specification draft found at: http://tc39.github.io/ecma262/
Changes relative to existing algorithms  are marked in **bold**.

## [%RegExp%](http://tc39.github.io/ecma262/#sec-regexp-constructor)

The %RegExp% instrinsic object, which is the builtin RegExp constructor, has the following additional internal slots:

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


## [RegExpAlloc ( _newTarget_ )](http://tc39.github.io/ecma262/#sec-regexpalloc)

RegExp instances have an additional slot, which optionally points to the object whose static properties should be updated after a successful match, namely %RegExp%. The RegExpAlloc abstraction operation is modified as follows:

1. Let _obj_ be ? OrdinaryCreateFromConstructor(_newTarget_, "%RegExpPrototype%", «[[RegExpMatcher]], [[OriginalSource]], [[OriginalFlags]], **[[LegacyConstructor]]**»).
1. **If _newTarget_ is %RegExp%, let _legacyConstructor_ be %RegExp%; else, let _legacyConstructor_ be __undefined__.**
1. **Set the value of _obj_'s [[LegacyConstructor]] internal slot to _legacyConstructor_.**
2. Let _status_ be DefinePropertyOrThrow(_obj_, "lastIndex", PropertyDescriptor {[[Writable]]: true, [[Enumerable]]: false, [[Configurable]]: false}).
3. Assert: _status_ is not an abrupt completion.
4. Return _obj_.


## [RegExpBuiltInExec ( _R_, _S_ )](http://tc39.github.io/ecma262/#sec-regexpbuiltinexec)

In The RegExpBuiltInExec abstract operation, a hook is added for updating the static properties of %RegExp% after a successful match. The three last steps of the algorithm are modified as follows:

1. ...
1. Perform CreateDataProperty(_A_, "0", _matchedSubstr_).
1. **Let _capturedValues_ be an new empty List.**
1. For each integer _i_ such that _i_ > 0 and _i_ ≤ _n_
    1. ...
    1. Perform CreateDataProperty(_A_, ToString(_i_) , _capturedValue_).
    1. **Append _capturedValue_ to the end of _capturedValues_.** 
1. **Perform UpdateLegacyRegExpStaticProperties(_R_, _S_, _lastIndex_, _e_, _capturedValues_).**
1. Return _A_.



## UpdateLegacyRegExpStaticProperties ( _R_, _S_, _startIndex_, _endIndex_, _capturedValues_ )

The abstract operation UpdateLegacyRegExpStaticProperties updates the values of the static properties of %RegExp% after a successful match.

1. Assert: _R_ is an initialized RegExp instance.
2. Let _C_ be the value of _R_’s [[LegacyConstructor]] internal slot.
1. If _C_ is undefined, return.
1. Assert: _C_ is a %RegExp% intrinsic object associated to some realm.
2. Assert: Type(_S_) is String.
3. Let _len_ be the number of code units in _S_.
4. Assert: _startIndex_ and _endIndex_ are integers such that 0 ≤ _startIndex_ ≤ _endIndex_ ≤ _len_.
5. Assert: _capturedValues_ is a List of Strings.
6. Let _n_ be the number of elements in _capturedValues_.
1. Set the value of _C_’s [[RegExpInput]] internal slot to _S_.
1. Set the value of _C_’s [[RegExpLastMatch]] internal slot to a String whose length is _endIndex_ - _startIndex_ and containing the code units from S with indices _startIndex_ through _endIndex_ - 1, in ascending order.
1. If _n_ > 0, set the value of _C_’s [[RegExpLastParen]] internal slot to the last element of _capturedValues_.
1. Else, set the value of _C_’s [[RegExpLastParen]] internal slot to the empty String.
1. Set the value of _C_’s [[RegExpLeftContext]] internal slot to a String whose length is _startIndex_ and containing the code units from S with indices 0 through _startIndex_ - 1, in ascending order.
1. Set the value of _C_’s [[RegExpRightContext]] internal slot to a String whose length is _len_ - _endIndex_ and containing the code units from S with indices _endIndex_ through _len_ - 1, in ascending order.
1. For each integer _i_ such that _i_ > 0 and _i_ ≤ 9
    1. If _n_ ≤ _i, set the value of _C_’s [[RegExpParen<i>i</i>]] internal slot to the <i>i</i>th element of _capturedValues_.
    1. Else, set the value of _C_’s [[RegExpParen<i>i</i>]] internal slot to the empty String.
    
Additional properties of the RegExp constructor
================================================

All the below properties are accessor properties with attributes {[[Set]]: undefined, [[Enumerable]]: false, [[Configurable]]: true}.

### get input

1. Return the value of %RegExp%’s [[RegExpInput]] internal slot.

### get $_

1. Return the value of %RegExp%’s [[RegExpInput]] internal slot.

### get lastMatch

1. Return the value of %RegExp%’s [[RegExpLastMatch]] internal slot.

### get $&

1. Return the value of %RegExp%’s [[RegExpLastMatch]] internal slot.

### get lastParen

1. Return the value of %RegExp%’s [[RegExpLastParen]] internal slot.

### get $+

1. Return the value of %RegExp%’s [[RegExpLastParen]] internal slot.

### get leftContext

1. Return the value of %RegExp%’s [[RegExpLeftContext]] internal slot.

### get $`

1. Return the value of %RegExp%’s [[RegExpLeftContext]] internal slot.

### get rightContext

1. Return the value of %RegExp%’s [[RegExpRightContext]] internal slot.

### get $'

1. Return the value of %RegExp%’s [[RegExpRightContext]] internal slot.

### get $1

1. Return the value of %RegExp%’s [[RegExpParen1]] internal slot.

### get $2

1. Return the value of %RegExp%’s [[RegExpParen2]] internal slot.

### get $3

1. Return the value of %RegExp%’s [[RegExpParen3]] internal slot.

### get $4

1. Return the value of %RegExp%’s [[RegExpParen4]] internal slot.

### get $5

1. Return the value of %RegExp%’s [[RegExpParen5]] internal slot.

### get $6

1. Return the value of %RegExp%’s [[RegExpParen6]] internal slot.

### get $7

1. Return the value of %RegExp%’s [[RegExpParen7]] internal slot.

### get $8

1. Return the value of %RegExp%’s [[RegExpParen8]] internal slot.

### get $9

1. Return the value of %RegExp%’s [[RegExpParen9]] internal slot.

