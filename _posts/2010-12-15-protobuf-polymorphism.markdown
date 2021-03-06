---
layout: post
title: Protocol Buffer Polymorphism
category: python
---

[Protocol Buffers][protobuf] provide an efficient way to encode structured data
for serialization.  The language's basic organizational type is a *message*,
which can be thought of as a C-style structure.  It is named and contains
some number of fields.  Messages can also be extended, but the method by which
this is accomplished differs from familiar C++ or Java-style inheritance.
Instead, [message extension][extensions] is implemented by reserving some
number of field indices in the base message for use by the extending messages.

    message BaseType
    {
        // Reserve field numbers 100 to 199 for extensions.
        extensions 100 to 199;

        // All other field numbers are available for use here.
        required string name = 1;
        optional uint32 quantity = 2;
    }

    extend BaseType
    {
        // This extension can only use field numbers 100 to 199.
        optional float price = 100;
    }

But can protocol buffers model more flexible inheritance and polymorphism
hierarchies?

## Optional Message Fields

One approach to implementing polymorphism involves the use of optional message
fields defined in either the base message or in an extension.  Each "subclass"
is defined as an independent message type that can optionally be included in
the top-level message.

    message Cat
    {
        optional bool declawed = 1;
    }

    message Dog
    {
        optional uint32 bones_buried = 1;
    }

    message Animal
    {
        required float weight = 1;
        optional Dog dog = 2;
        optional Cat cat = 3;
    }

This composition scheme is simple but has the property of allowing multiple
nested message types to be filled out at the same time.  While desirable in
some contexts, this means that the deserialization code must test for the
availability of each and every optional message type (unless an additional
type hint field is added).  It can could also be considered less than "pure"
as a result.

## Embedded Serialized Messages

A second approach simply "embeds" the subclass's serialized contents within a
`bytes` field in the parent message.  An explicit type field informs the
deserialization code of the embedded message's type.

    message Animal
    {
        enum Type
        {
            Cat = 1;
            Dog = 2;
        }

        required Type type = 1;
        required bytes subclass = 2;
    }

This brute-force approach to the problem, which effective, is both inelegant
and inefficient.  It can be useful where it is desirable to defer the
deserialization of the embedded message, however, such as in routing systems
that are only interested in decoding the outer enveloping message's fields.

## Nested Extensions

The final (and most generally recommended) approach uses a combination of
[nested messages][nested] and extensions to implemented polymorphic message
types.  This works by having each subclass reference itself from within a
nested extension of the base type.  An explicit type field is still required to
guide the deserialization process.

    message Animal
    {
        extensions 100 to max;

        enum Type
        {
            Cat = 1;
            Dog = 2;
        }

        required Type type = 1;
    }

    message Cat
    {
        extend Animal
        {
            required Cat animal = 100; // Unique Animal extension number
        }

        // These fields can use the full number range.
        optional bool declawed = 1;
    }

    message Dog
    {
        extend Animal
        {
            required Dog animal = 101; // Unique Animal extension number
        }

        // These fields can use the full number range.
        optional uint32 bones_buried = 1;
    }

It may not be immediately obvious how to work with this message structure, so
here's an example using the Python API:

{% highlight python %}

    from animals_pb2 import *

    # Construct the polymorphic base message type.
    animal = Animal()
    animal.type = Animal.Cat

    # Create the subclass type by referencing the appropriate extension type.
    # Note that this uses the self-referential field (Cat.animal) from within
    # nested message extension.
    cat = animal.Extensions[Cat.animal]
    cat.declawed = True

    # Serialize the complete message contents to a string.  It will end up
    # looking roughly like this: [ type [ declawed ] ]
    bytes = animal.SerializeToString()

    # ---

    # Unpack the serialized bytes.
    animal = Animal()
    animal.ParseFromString(bytes)

    # Determine the appropriate extension type to use.
    extension_map = { Animal.Cat: Cat.animal, Animal.Dog: Dog.animal }
    extension = animal.Extensions[extension_map[animal.type]]

{% endhighlight %}

This approach tends to be the most efficient and robust implementation, albeit
a bit non-obvious to protocol buffer newcomers.

## Conclusion

As shown above, there are multiple ways to implement polymorphic message
behavior using protocol buffers.  Like most things in software, the approaches
differ in complexity and efficiency.

It's a little unfortunate that there isn't a more obvious language syntax for
expressing inheritance.  For example, many programmers would likely find these
definitions familiar:

    message Animal { ... }
    message Cat : Animal { ... }
    message Dog : Animal { ... }

There are of course open questions as to how this syntax would map to the
underlying concepts of messages and extensions, but it shouldn't be too
difficult to enforce things like extension number ranges, etc. during the
compilation process.  Perhaps something along these lines will be undertaken by
a motivated developer or an ambitious [Google Summer of Code][gsoc] student.

[protobuf]: http://code.google.com/p/protobuf/
[extensions]: http://code.google.com/apis/protocolbuffers/docs/proto.html#extensions
[nested]:  http://code.google.com/apis/protocolbuffers/docs/proto.html#nested
[gsoc]: http://code.google.com/soc/
