# Typed Objects

## Overview

_Structure_: a collection of field records. Every field record is a triple {_name_ : property name, _byteOffset_ : integer, _type_ : fieldType}. 

_fieldType_ is either a value type or an ECMAScript object, required to be a _type object_.

value types: uint8, int8, uint16, int16, uint32, int32, float32, float64, any, string, object

## Type Objects
_Type object_ is an exotic object, constructable with a type object constructor (StructType or ArrayType). 

Every type object is a constructor-like function object. Every type object carries a [[Structure]] internal slot and a \[\[Size]] internal slot.

### TypeObject(obj) constructor


### TypeObject(arrayBuffer\[, byteOffset\]) constructor 

### Ground type objects

# SameValue, SameValueZero and === algorithm on typed objects

All three algorithms are modified in the same way:
  - If _x_ and _y_ are typed objects:
        - Return true if values of internal slots [[TypeObject]], [[ViewedArrayBuffer]], [[ByteOffset]] and 
          [[Opacity]] are SameValue respectively.
        - Return false otherwise.

## Typed Object
_Typed objects_ are exotic objects that are created from Type Objects. They carry the following internal slots:
  - [[TypeObject]] 
  - [[ViewedArrayBuffer]] 
  - [[ByteOffset]]
  - [[Opacity]]

### \[\[GetOwnProperty]] (P) on typed object
When the [[GetOwnProperty]] internal method of an exotic typed object O is called with property key P the following steps are taken:

1. Let s be a result of Structure(O).
1. Let field record r be a field record with name P from s
1. Return undefined if r does exist
1. Let value be a result of GetFieldFromTypedObject(o, P)
1. Set isInteger to be true if ToInteger(P) is not an abrupt completion, false otherwise
1. Return a PropertyDescriptor { [[Value]] : value, [[Enumerable]]: isInteger, [[Writable]]: true, [[Configurable]]: false }


### \[\[Get]](P, Receiver) on typed object
When the [[Get]] internal method of an exotic typed object O is called with property key P and ECMAScript language 
value Receiver the following steps are taken:

1. Assert: IsPropertyKey(P) is true.
1. If Type(P) is String and if SameValue(O, Receiver) is true, then
    1. Return the result of GetFieldFromTypedObject (O, P).
1. Otherwise, return the result of calling the default ordinary object [[Get]] internal method (9.1.8) on O passing P and Receiver as arguments.


###\[\[Set]] ( P, V, Receiver)

When the [[Set]] internal method of an exotic typed object O is called with property key P, value V, and ECMAScript language value Receiver, the following steps are taken:

1. Assert: IsPropertyKey(P) is true.
1. If Type(P) is String and if SameValue(O, Receiver) is true, then
   1. Return the result of SetFieldInTypedObject(O, P, V).
1. Otherwise Return the result of calling the default ordinary object [[Set]] internal method (9.1.8) on O passing P, V, and Receiver as arguments.
2. 

### \[\[Structure]](O)

For exotic typed object O:

1. Let _typeObject_ be a value of [[TypeObject]] internal slot of _O_.
2. Return a value of [[Structure]] internal slot of _typeObject_.


# Type Object Constructors

## StructType

The StructType object is a constructor-like function that creates type objects. 

### StructType(object)

StructType called with _object_ argument performs the following steps:

1. Assert: Type\(_object_) is Object.
1. Let _O_ be *this* value.
1. If _O_ does not have \[\[Structure\]\] or \[\[Size\]\] internal slots, throw *TypeError*.
1. Let _currentOffset_ be zero.
1. Let _structure_ be an empty list.
1. For each own property key _P_ of _object_, iterated in the standard own property iteration order:
    1. Let _fieldType_ be a result of internal operation \[\]\[Get]](_P_, _O_) on object _O_.
    1. If _fieldType_ is not a type object, throw _TypeError_.
    1. Let _alignment_ be a result of Alignment\(_fieldType_).
    1. Set _currentOffset_ to a minimal integer equal to or greater than _currentOffset_ that 
       is evenly divisible by _alignment_.
    1. Let _r_ be a field record with _name_ equal to _fieldName_, _byteOffset_ equal to _currentOffset_,
       and _type_ equal to _fieldType_.
    1. Add _r_ to the end of _structure_ list.
    1. Let _s_ be Size\(_fieldType_\).
    1. ReturnIfAbrupt\(_s_\).
    1. Set _currentOffset_ to _currentOffset_ + _s_. 
1. Let _size_ be _currentOffset_.
1. Set _O_'s \[\[Structure\]\] to _structure_.
1. Set _O_'s \[\[Size\]\] to _size_.
1. Return _O_.


## ArrayType

# Abstract Operations

## Alignment(typeObject)

1. If _typeObject_ is one of the ground type objects return \[\[Size\]\](typeObject).
1. Let _S_ be a value of _typeObject_'s \[\[Structure\]\] internal slot.
1. Return a maximum of Alignment(_t_) where _t_ goes over values of _fieldType_ properties of field records in _S_. 

## Size(typeObject)

1. If _typeObject_ is one of the ground type objects return:
   1. 1 for *uint8*, *int8*.
   1. 2 for *uint16*, *int16*.
   1. 4 for *uint32*, *int32*, *float32*.
   1. 8 for *float64*.
1. If _typeObject_ has a  \[\[Size\[]\] internal slot, return its value.
1. Otherwise, throw *TypeError*.


## InitializeTypeObjectInternals(O, arrayBuffer, byteOffset, typeObject)
   
1. Set _O_'s \[\[Structure\]\] to _s_.
1. Set _O_'s \[\[ViewedArrayBuffer\]\] to _arrayBuffer_.
1. Set _O_'s \[\[ByteOffset\]\] to _byteOffset_.
1. Set _O_'s \[\[TypeObject\]\] to _typeObject_.

## CreateTypedObjectFromBuffer(arrayBuffer, byteOffset, typeObject)

1. If _byteOffset_ + Size\(_typeObject_\) is bigger than _arrayBuffer_'s \[\[ByteLength\]\], throw *RangeError*.
1. Let _s_ be _typeObject_'s \[\[Structure\]\].
1. Let _O_ be a result of ObjectCreate(_typeObject_.*prototype*, 
    (\[\[Structure\]\], \[\[ViewedArrayBuffer\]\], \[\[ByteOffset\]\], \[\[TypeObject\]\])).
1. ReturnIfAbrupt(_O_).
1. InitializeTypeObjectInternals(_O_, _arrayBuffer_, _typeObject_).
1. Return _O_.




## GetFieldFromTypedObject(typedObject, fieldName)

1. Let structure be [[Structure]] from typedObject
1. Let buffer be [[ViewedArrayBuffer]] from typedObject.
1. Find field record r for fieldName in structure
1. Return undefined if r does not exist
1. If r.type is value type, return GetValueFromBuffer(buffer, byteIndex + r.byteIndex, r.type)
1. Otherwise, return CreateTypedObjectFromBuffer(buffer, byteIndex + r.byteIndex, r.type)

## SetFieldInTypedObject(typedObject, fieldName, value)

1. Let _structure_ be [[Structure]] from _typedObject_.
1. Let _buffer_ be \[\[ViewedArrayBuffer\]\] from _typedObject_.
1. Find field record _r_ for _fieldName_ in structure
1. If r.type is value type, return SetValueInBuffer(buffer, byteIndex + r.byteIndex, value, r.type)
1. Otherwise,
   1. if value is not typed object, throw TypeError
   1. if not EquivalentTypes([[TypeObject]] of value, r.type), throw TypeError (?)
   1. Let t = CreateTypeObjectFromBuffer(buffer, byteOffset + r.byteOffset, value)
   1. For each field record r1 in [[Structure]] of r.type:
     1. SetFieldInTypedObject(t, r1.fieldName, Get(value, fieldName))

## EquivalentTypes(typeObject1, typeObject2)

