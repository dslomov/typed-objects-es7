# Typed Objects

## Overview

_Structure_: a collection of field records. Every field record is a triple {_name_ : property name, _byteOffset_ : integer, _type_ : fieldType}. 

_fieldType_ is either a value type or an ECMAScript object, required to be a _type object_.

value types: uint8, int8, uint16, int16, uint32, int32, float32, float64, any, string, object

## Type Objects
_Type object_ is an exotic object, constructable with a type object constructor (StructType or ArrayType). 

Every type object is a constructor-like function object. Every type object carries a [[Structure]] internal slot. 

## Typed Object
_Typed objects_ are exotic objects that are instances of Type Objects. They carry a [[ViewedArrayBuffer]] internal slot, a [[Structure]] internal slot, a [[TypeObject]] and [[ByteOffset]] internal slot.

### \[\[GetOwnProperty]] (P) on typed object
When the [[GetOwnProperty]] internal method of an exotic typed object O is called with property key P the following steps are taken:
1. Let s be a value of internal slot [[Structure]] of object o.
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


# Type Object Constructors
## StructType
## ArrayType

# Abstract Operations

## Size(typeObject)

## CreateTypedObjectFromBuffer(arrayBuffer, byteIndex, typeObject)

## GetFieldFromTypedObject(typedObject, fieldName)

1. Let structure be [[Structure]] from typedObject
1. Let buffer be [[ViewedArrayBuffer]] from typedObject.
1. Find field record r for fieldName in structure
1. Return undefined if r does not exist
1. If r.type is value type, return GetValueFromBuffer(buffer, byteIndex + r.byteIndex, r.type)
1. Otherwise, return CreateTypedObjectFromBuffer(buffer, byteIndex + r.byteIndex, r.type)

## SetFieldInTypedObject(typedObject, fieldName, value)

1. Let structure be [[Structure]] from typedObject
1. Let buffer be [[ViewedArrayBuffer]] from typedObject.
1. Find field record r for fieldName in structure
1. If r.type is value type, return SetValueInBuffer(buffer, byteIndex + r.byteIndex, value)
1. Otherwise,
   1. if value is not typed object, throw TypeError
   1. if not EquilvalentTypes([[TypeObject]] of value, r.type), throw TypeError (?)
   1. Let t = CreateTypeObjectFromBuffer(buffer, byteIndex + r.byteIndex, value)
   1. For each field record r1 in [[Structure]] of r.type:
     1. SetFieldInTypedObject(t, r1.fieldName, Get(value, fieldName))

## EquivalentTypes(typeObject1, typeObject2)

