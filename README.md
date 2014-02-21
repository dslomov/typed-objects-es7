# Typed Objects

## Overview

_Structure_: either
  - one of ``uint8``, ``int8``, ``uint16``, ``int16``, ``uint32``, ``int32``, ``float32``, ``float64``, 
    ``any``, ``string``, ``object`` (_ground structures_)
  - an ordered list of _FieldRecord_s.

_FieldRecord_: a triple of {``name`` : property name, ``byteOffset`` : integer, ``type`` : _TypeObject_ }. 

_Dimensions_: A list of integers (positive).

## Type Descriptors

_TypeDescriptor_ is an exotic object that represents a shape of a typed object. 

Type descriptors carry the following internal slots:
  - ``[[Structure]]``, should have a _Structure_ value.
  - ``[[Rank]]``, an integer.
  - ``[[Opacity]]``, a boolean.
  - ``[[ArrayDescriptor]]``, either ``undefined`` or a _TypeDescriptor_.
  - ``[[OpaqueDescriptor]]``, either ``undefined`` or a _TypeDescriptor_.

Type descriptors represent a shape of typed object. Identical typed objects have identical type descriptor. Type descriptors act as prototypes of typed objects.

All array type objects with the same element type share their type descriptor.


### Ground Type Descriptors

There is a fixed set of _ground type descriptors_. Their ``[[Structure]]``s are gorund structures/

All ground type descriptors are have ``[[Rank]]`` equal to 0.
Their ``[[ArrayDescriptor]]`` and ``[[OpaqueDescriptor]]`` internal slots are initially ``undefined``.

Their other properties are as follows:

  - ``[[uint8]]``: ``[[Structure]]``: ``uint8``, ``[[Opacity]]``: *false*.
  - ``[[int8]]``: ``[[Structure]]``: ``int8``, ``[[Opacity]]``: *false*.
  - ``[[uint16]]``: ``[[Structure]]``: ``uint16``, ``[[Opacity]]``: *false*.
  - ``[[int16]]``: ``[[Structure]]``: ``int16``, ``[[Opacity]]``: *false*.
  - ``[[uint32]]``: ``[[Structure]]``: ``uint32``, ``[[Opacity]]``: *false*.
  - ``[[int32]]``: ``[[Structure]]``: ``int32``, ``[[Opacity]]``: *false*.
  - ``[[float32]]``: ``[[Structure]]``: ``float32``, ``[[Opacity]]``: *false*.
  - ``[[float64]]``: ``[[Structure]]``: ``float64``, ``[[Opacity]]``: *false*.
  - ``[[any]]``: ``[[Structure]]``: ``any``, ``[[Opacity]]``: *true*.
  - ``[[string]]``: ``[[Structure]]``: ``string``, ``[[Opacity]]``: *true*.
  - ``[[object]]``: ``[[Structure]]``: ``object``, ``[[Opacity]]``: *true*.

  

## Type Objects
_TypeObject_ is an exotic object, constructable with a type object constructor (StructType or ArrayType). 

Every type object carries the following internal slots:
  - ``[[TypeDescriptor]]``
  - ``[[Dimensions]]``

For every tpe object, ``length([[Dimensions]]) == [[Rank]] of [[TypeDescriptor]]``.

### Ground type objects

The following type objects with ``[[TypeDescriptor]]``s being ground type descriptor ara available to ECMAScript programs
under the following names:

  - ``${NAME}`` : ``[[TypeDescriptor]]`` : ``[[${NAME}]]``. TODO nice list

Their ``[[Dimensions]]`` is an empty list, as required by the above invariant.

  
### ``[[Call]]`` for Type Objects  

Type objects have a ``[[Call]]`` internal method defined. Its behaviour is specified below.
  

### TypeObject(obj)

### TypeObject(arrayBuffer\[, byteOffset\])



## Typed Object

_Typed objects_ are exotic objects that are created from Type Objects. They carry the following internal slots:
  - ``[[TypeDescriptor]]``: the values should be a type descriptor
  - ``[[Dimensions]]``: number of elements in dimensions should be equal to the rank of type descriptor.
  - ``[[ViewedArrayBuffer]]``
  - ``[[ByteOffset]]``
  - ``[[Opacity]]``

### ``[[GetOwnProperty]]`` (P) on typed object
When the ``[[GetOwnProperty]]`` internal method of an exotic typed object O is called with property key P the following steps are taken:

TODO: Arrays

1. Let s be a result of Structure(O).
1. Let field record r be a field record with name P from s
1. Return undefined if r does exist
1. Let value be a result of GetFieldFromTypedObject(o, P)
1. Set isInteger to be true if ToInteger(P) is not an abrupt completion, false otherwise
1. Return a PropertyDescriptor { [[Value]] : value, [[Enumerable]]: isInteger, [[Writable]]: true, [[Configurable]]: false }



### ``[[GetPrototypeOf]]``()

When ``[[GetPrototypeOf]]`` is called on typed object _O_, the following steps are taken:

1. Let _typeDescriptor_ be a value of _O_'s ``[[TypeDescriptor]]`` internal slot.
3. Return _typeDescriptor_


### ``[[IsExtensible]]``()

``[[IsExtensible]]`` for typed object _O_ returns *false*.


### ``[[Structure]]``(O)

For exotic typed object O:

1. Let _typeDescriptor_ be a value of ``[[TypeDescriptor]]`` internal slot of _O_.
2. Return a value of ``[[Structure]]`` internal slot of _typeDescriptor_.

### SameValue, SameValueZero and === algorithm on typed objects

All three algorithms are modified in the same way:
  - If _x_ and _y_ are typed objects
      * Return true if the following holds:
         1. values of internal slots ``[[TypeDescriptor]]``, ``[[ViewedArrayBuffer]]``, ``[[ByteOffset]]`` and 
            ``[[Opacity]]`` are SameValue respectively.
         2. Values of internal slot ``[[Dimensions]]`` of _x_ and _y_ are identical lists of numbers.
      * Return false otherwise.


# Type Object Constructors

## ``StructType``

The ``StructType`` object is a constructor-like function that creates type objects. 

### ``StructType``(object)

``StructType`` called with _object_ argument performs the following steps:

TODO: Opaque structures

1. Assert: Type\(_object_) is Object.
1. Let _O_ be *this* value.
1. If _O_ does not have ``[[TypeDescriptor]]`` internal slots, throw *TypeError*. (TODO check other slots)
1. Let _currentOffset_ be zero.
1. Let _structure_ be an empty list.
1. For each own property key _P_ of _object_, iterated in the standard own property iteration order:
    1. Let _fieldType_ be a result of internal operation Get(_P_, _O_) on object _O_.
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
1. Let _typeDescriptor_ be CreateStructTypeDescriptor(_structure_).
1. Set _O_'s ``[[TypeDescriptor]]`` to _typeDescriptor_.
1. Set _O_'s ``prototype`` property to _typeDescriptor_.
1. Make _O_'s ``prototype`` property *read-only* and *non-configurable*.
1. Return _O_.

### ``StructType.prototype.ArrayType``(N)

TODO: convert N to integer number properly.

1. Let _O_ be *this* value.
1. Let _typeDescriptor_ be a value of _O_'s ``[[TypeDescriptor]]`` internal slot.
2. Let _structure_ be _ownTypeDescriptor_'s ``[[Structure]]``.
3. Let _dimensions_ be _O_'s ``[[Dimensions]]``.
5. Let _arrayDescriptor_ be GetOrCreateArrayTypeDescriptor(_typeDescriptor_)
5. Create a new _TypeObject_ _result_. TODO.
6. Set _result_'s ``[[TypeDescriptor]]`` to _arrayDescriptor_.
6. Set _result_'s ``prototype`` to _arrayDescriptor_.
6. Set _result_'s ``prototype`` property *read-only* and *non-configurable*.
7. Let _newDimesions_ be a result of appending _N_ to _dimensions_.
8. Set _result_'s ``[[Dimensions]]`` to _newDimensions_.
8. Return _result_.

### ``StructType.prototype.OpaqueType``

TODO: A copy of *this* with \[\[Opacity\]] set to *false* if it is true, *this* otherwise + appropriate caching.


# Abstract Operations

## Alignment(typeDescriptor)

1. Let _S_ be a value of _typeDescriptor_'s `[[Structure]]`` internal slot.
2. If _S_ is a ground structure, return Size(_S_). TODO: opaque
1. Otherwise, return a maximum of Alignment(TypeDescriptor(_t_)) where _t_ goes over values of _fieldType_ properties of field records in _S_. 

## Size(structure)

1. If _structure_ is one of the ground type objects return:
   1. 1 for *uint8*, *int8*.
   1. 2 for *uint16*, *int16*.
   1. 4 for *uint32*, *int32*, *float32*.
   1. 8 for *float64*.
1. Otherwise:
    1. Let _currentOffset_ be zero.
    1. For each field record _r_ in _structure_:
        1. Let _fieldType_ be _r_._fieldType_.
        1. Let _alignment_ be a result of Alignment\(_fieldType_).
        1. Set _currentOffset_ to a minimal integer equal to or greater than _currentOffset_ that 
           is evenly divisible by _alignment_.
        1. Let _s_ be Size\(_fieldType_\).
        1. Set _currentOffset_ to _currentOffset_ + _s_. 
    1. Return _currentOffset_.


## CreateStructTypeDescriptor(_structure_)

Creates a new type descriptor that describes a struct.

1. Let _result_ be a new _TypeDescriptor_ TODO: proper spec
1. Set _result_'s ``[[Structure]]`` to _structure_.
1. Set _result_'s ``[[Rank]]`` to 0.
1. Set _result_'s ``[[ArrayDescriptor]]`` to ``undefined``.
1. Set _result_'s ``[[Opacity]]`` to Opaque(``structure``).
1. Return _result_.
  
## CreateArrayTypeDescriptor(_typeDescriptor_)

Creates a new type descriptor that describes an array with elements of type described by _typeDescriptor_.

1. Let _result_ be a new _TypeDescriptor_ TODO: proper spec
2. Set _result_'s ``[[Structure]]`` to _typeDescriptor_'s ``[[Structure]]``.
3. Set _result_'s ``[[Rank]]`` to _typeDescriptor_'s ``Rank`` + 1
4. Set _result_'s ``[[Opacity]]`` to _typeDescriptor_'s ``[[Opacity]]`.
5. Set _result_'s ``[[ArrayDescriptor]]`` to ``undeifined``.
6. Return _result_.
7. 

## GetOrCreateArrayTypeDescriptor(_typeDescriptor_)

1. Let _cached_ be _typeDescriptor_'s ``[[ArrayDescriptor]]``.
2. If _cached_ is not ``undefined``, return _cached_.
3. Let _result_ be CreateArrayTypeDescriptor(_typeDescriptor_).
4. Set ``[[ArrayDescriptor]]`` of _typeDescriptor_ to _result_.
5. Return _result_.

## Size(_structure_, _dimensions_)

_structure_ is a structure and _dimensions_ is a possibly empty list of integers.

1. Let _baseSize_ be Size(_structure_).
2. If _dimensions_ is an empty list, return _baseSize_.
3. Let _count_ be a product of all integers in _dimensions_ list.
4. Return _count_ * _baseSize_.

## CreateTypedObjectFromBuffer(arrayBuffer, byteOffset, typeObject)

1. If _byteOffset_ + Size\(_typeObject_\) is bigger than _arrayBuffer_'s \[\[ByteLength\]\], throw *RangeError*.
1. Let _s_ be _typeObject_'s \[\[Structure\]\].
1. Let _O_ be a result of ObjectCreate(_typeObject_.*prototype*, 
    (\[\[Structure\]\], \[\[ViewedArrayBuffer\]\], \[\[ByteOffset\]\], \[\[TypeObject\]\])).
1. ReturnIfAbrupt(_O_).
1. InitializeTypeObjectInternals(_O_, _arrayBuffer_, _typeObject_).
1. Return _O_.


## GetFieldFromTypedObject(typedObject, fieldName)

1. Let structure be Structure ( _typedObject_ )
1. Let buffer be [[ViewedArrayBuffer]] from typedObject.
1. Find field record r for fieldName in structure
1. Return undefined if r does not exist
1. If r.type is value type, return GetValueFromBuffer(buffer, byteIndex + r.byteIndex, r.type)
1. Otherwise, return CreateTypedObjectFromBuffer(buffer, byteIndex + r.byteIndex, r.type)

## SetFieldInTypedObject(typedObject, fieldName, value)

1. Let _structure_ be Structure ( _typedObject_ ).
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

