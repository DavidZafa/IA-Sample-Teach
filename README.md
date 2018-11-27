[![General Assembly Logo](https://camo.githubusercontent.com/1a91b05b8f4d44b5bbfb83abac2b0996d8e26c92/687474703a2f2f692e696d6775722e636f6d2f6b6538555354712e706e67)](https://generalassemb.ly/education/web-development-immersive)

# IA-Sample-Teach


# Prerequisites

*Javascript OOP
*Variables, Functions, Callbacks

## Objectives

By the end of this, developers should be able to:

* Explain prototypal inheritance
* Understand the difference between class and prototypal inheritance


# What’s the Difference Between Class & Prototypal Inheritance?

* Class Inheritance: A class is like a blueprint — a description of the object to be created. Classes inherit from classes and create subclass relationships: hierarchical class taxonomies.

* Instances are typically instantiated via constructor functions with the `new` keyword. Class inheritance may or may not use the `class` keyword from ES6. Classes as you may know them from languages like Java don’t technically exist in JavaScript. Constructor functions are used, instead. The ES6 `class` keyword desugars to a constructor function:

```js
class Foo {}
typeof Foo // 'function'
```
In JavaScript, class inheritance is implemented on top of prototypal inheritance, but that does not mean that it does the same thing:

JavaScript’s class inheritance uses the prototype chain to wire the child `Constructor.prototype` to the parent `Constructor.prototype` for delegation. Usually, the `super()` constructor is also called. Those steps form single-ancestor parent/child hierarchies and create the tightest coupling available in OO design.

“Classes inherit from classes and create subclass relationships: hierarchical class taxonomies.”
Prototypal Inheritance: A prototype is a working object instance. Objects inherit directly from other objects.

Instances may be composed from many different source objects, allowing for easy selective inheritance and a flat [[Prototype]] delegation hierarchy. In other words, class taxonomies are not an automatic side-effect of prototypal OO: a critical distinction.

Instances are typically instantiated via factory functions, object literals, or `Object.create()`.

“A prototype is a working object instance. Objects inherit directly from other objects.”
Why Does this Matter?
Inheritance is fundamentally a code reuse mechanism: A way for different kinds of objects to share code. The way that you share code matters because if you get it wrong, it can create a lot of problems, specifically:

Class inheritance creates parent/child object taxonomies as a side-effect.

Those taxonomies are virtually impossible to get right for all new use cases, and widespread use of a base class leads to the fragile base class problem, which makes them difficult to fix when you get them wrong. In fact, class inheritance causes many well known problems in OO design:

The tight coupling problem (class inheritance is the tightest coupling available in oo design), which leads to the next one…
The fragile base class problem
Inflexible hierarchy problem (eventually, all evolving hierarchies are wrong for new uses)
The duplication by necessity problem (due to inflexible hierarchies, new use cases are often shoe-horned in by duplicating, rather than adapting existing code)
The Gorilla/banana problem (What you wanted was a banana, but what you got was a gorilla holding the banana, and the entire jungle)
I discuss some of the issues in more depth in my talk, “Classical Inheritance is Obsolete: How to Think in Prototypal OO”:


The solution to all of these problems is to favor object composition over class inheritance.

“Favor object composition over class inheritance.”
~ The Gang of Four, “Design Patterns: Elements of Reusable Object Oriented Software”
Summed up nicely here:


Is All Inheritance Bad?
When people say “favor composition over inheritance” that is short for “favor composition over class inheritance” (the original quote from “Design Patterns” by the Gang of Four). This is common knowledge in OO design because class inheritance has many flaws and causes many problems. Often people leave off the word class when they talk about class inheritance, which makes it sound like all inheritance is bad — but it’s not.

There are actually several different kinds of inheritance, and most of them are great for composing composite objects from multiple component objects.

Three Different Kinds of Prototypal Inheritance
Before we dive into the other kinds of inheritance, let’s take a closer look at what I mean by class inheritance:


You can experiment with this example on Codepen.

`BassAmp` inherits from `GuitarAmp`, and `ChannelStrip` inherits from `BassAmp` & `GuitarAmp`. This is an example of how OO design goes wrong. A channel strip isn’t actually a type of guitar amp, and doesn’t actually need a cabinet at all. A better option would be to create a new base class that both the amps and the channel strip inherits from, but even that has limitations.

Eventually, the new shared base class strategy breaks down, too.

There’s a better way. You can inherit just the stuff you really need using object composition:


Experiment with this on CodePen.

If you look carefully, you might see that we’re being much more specific about which objects get which properties because with composition, we can. It wasn’t really an option with class inheritance. When you inherit from a class, you get everything, even if you don’t want it.

At this point, you may be thinking to yourself, “that’s nice, but where are the prototypes?”

To understand that, you have to understand that there are three different kinds of prototypal OO.

Concatenative inheritance: The process of inheriting features directly from one object to another by copying the source objects properties. In JavaScript, source prototypes are commonly referred to as mixins. Since ES6, this feature has a convenience utility in JavaScript called `Object.assign()`. Prior to ES6, this was commonly done with Underscore/Lodash’s `.extend()` jQuery’s `$.extend()`, and so on… The composition example above uses concatenative inheritance.

Prototype delegation: In JavaScript, an object may have a link to a prototype for delegation. If a property is not found on the object, the lookup is delegated to the delegate prototype, which may have a link to its own delegate prototype, and so on up the chain until you arrive at `Object.prototype`, which is the root delegate. This is the prototype that gets hooked up when you attach to a `Constructor.prototype` and instantiate with `new`. You can also use `Object.create()` for this purpose, and even mix this technique with concatenation in order to flatten multiple prototypes to a single delegate, or extend the object instance after creation.

Functional inheritance: In JavaScript, any function can create an object. When that function is not a constructor (or `class`), it’s called a factory function. Functional inheritance works by producing an object from a factory, and extending the produced object by assigning properties to it directly (using concatenative inheritance). Douglas Crockford coined the term, but functional inheritance has been in common use in JavaScript for a long time.

As you’re probably starting to realize, concatenative inheritance is the secret sauce that enables object composition in JavaScript, which makes both prototype delegation and functional inheritance a lot more interesting.

When most people think of prototypal OO in JavaScript, they think of prototype delegation. By now you should see that they’re missing out on a lot. Delegate prototypes aren’t the great alternative to class inheritance — object composition is.

Why Composition is Immune to the Fragile Base Class Problem
To understand the fragile base class problem and why it doesn’t apply to composition, first you have to understand how it happens:

`A` is the base class
`B` inherits from `A`
`C` inherits from `B`
`D` inherits from `B`
`C` calls `super`, which runs code in `B`. `B` calls `super` which runs code in `A`.

`A` and `B` contain unrelated features needed by both `C` & `D`. `D` is a new use case, and needs slightly different behavior in `A`’s init code than `C` needs. So the newbie dev goes and tweaks `A`’s init code. `C` breaks because it depends on the existing behavior, and `D` starts working.

What we have here are features spread out between `A` and `B` that `C` and `D` need to use in various ways. `C` and `D` don’t use every feature of `A` and `B`… they just want to inherit some stuff that’s already defined in `A` and `B`. But by inheriting and calling `super`, you don’t get to be selective about what you inherit. You inherit everything:

“…the problem with object-oriented languages is they’ve got all this implicit environment that they carry around with them. You wanted a banana but what you got was a gorilla holding the banana and the entire jungle.” ~ Joe Armstrong — “Coders at Work”
With Composition
Imagine you have features instead of classes:

feat1, feat2, feat3, feat4
`C` needs `feat1` and `feat3`, `D` needs `feat1`, `feat2`, `feat4`:

const C = compose(feat1, feat3);
const D = compose(feat1, feat2, feat4);
Now, imagine you discover that `D` needs slightly different behavior from `feat1`. It doesn’t actually need to change `feat1`, instead, you can make a customized version of `feat1` and use that, instead. You can still inherit the existing behaviors from `feat2` and `feat4` with no changes:

const D = compose(custom1, feat2, feat4);
And `C` remains unaffected.

The reason this is not possible with class inheritance is because when you use class inheritance, you buy into the whole existing class taxonomy.

If you want to adapt a little for a new use-case, you either end up duplicating parts of the existing taxonomy (the duplication by necessity problem), or you refactor everything that depends on the existing taxonomy to adapt the taxonomy to the new use case due to the fragile base class problem.

Composition is immune to both.

You Think You Know Prototypes, but…
If you were taught to build classes or constructor functions and inherit from those, what you were taught was not prototypal inheritance. You were taught how to mimic class inheritance using prototypes. See “Common Misconceptions About Inheritance in JavaScript”.

In JavaScript, class inheritance piggybacks on top of the very rich, flexible prototypal inheritance features built into the language a long time ago, but when you use class inheritance — even the ES6+ `class` inheritance built on top of prototypes, you’re not using the full power & flexibility of prototypal OO. In fact, you’re painting yourself into corners and opting into all of the class inheritance problems.
