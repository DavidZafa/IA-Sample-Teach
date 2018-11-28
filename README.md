[![General Assembly Logo](https://camo.githubusercontent.com/1a91b05b8f4d44b5bbfb83abac2b0996d8e26c92/687474703a2f2f692e696d6775722e636f6d2f6b6538555354712e706e67)](https://generalassemb.ly/education/web-development-immersive)

# IA Sample Teach


# Prerequisites

* Javascript OOP
* Variables, Functions, Callbacks

# Objectives

By the end of this, developers should be able to:

* Explain prototypal inheritance
* Understand the implementation of prototypal inheritance
* Understand the difference between class and prototypal inheritance

# Prototypal Inheritance

* In programming, we often want to take something and extend it.

For instance, we have a user object with its properties and methods, and want to make admin and guest as slightly modified variants of it. We’d like to reuse what we have in user, not copy/reimplement its methods, just build a new object on top of it.

Prototypal inheritance is a language feature that helps in that.

## [[Prototype]]
In JavaScript, objects have a special hidden property [[Prototype]] (as named in the specification), that is either null or references another object. That object is called “a prototype”:

![picture](https://javascript.info/article/prototype-inheritance/object-prototype-empty.png)

That [[Prototype]] has a “magical” meaning. When we want to read a property from object, and it’s missing, JavaScript automatically takes it from the prototype. In programming, such thing is called “prototypal inheritance”. Many cool language features and programming techniques are based on it.

The property [[Prototype]] is internal and hidden, but there are many ways to set it.

One of them is to use __proto__, like this:
```js
 let animal = {
  eats: true
};
let rabbit = {
  jumps: true
};

rabbit.__proto__ = animal;
```

* Please note that __proto__ is not the same as [[Prototype]]. That’s a getter/setter for it. We’ll talk about other ways of setting it later, but for now __proto__ will do just fine.

If we look for a property in rabbit, and it’s missing, JavaScript automatically takes it from animal.

For instance:
```js
 let animal = {
  eats: true
};
let rabbit = {
  jumps: true
};

rabbit.__proto__ = animal; // (*)
```

// we can find both properties in rabbit now:
```js
alert( rabbit.eats ); // true (**)
alert( rabbit.jumps ); // true
```
Here the line (*) sets animal to be a prototype of rabbit.

Then, when alert tries to read property rabbit.eats (**), it’s not in rabbit, so JavaScript follows the [[Prototype]] reference and finds it in animal (look from the bottom up):

![picture](https://javascript.info/article/prototype-inheritance/proto-animal-rabbit.png)

Here we can say that "animal is the prototype of rabbit" or "rabbit prototypally inherits from animal".

So if animal has a lot of useful properties and methods, then they become automatically available in rabbit. Such properties are called “inherited”.

If we have a method in animal, it can be called on rabbit:

```js
 let animal = {
  eats: true,
  walk() {
    alert("Animal walk");
  }
};

let rabbit = {
  jumps: true,
  __proto__: animal
};

// walk is taken from the prototype
rabbit.walk(); // Animal walk
```
The method is automatically taken from the prototype, like this:

![picture](https://javascript.info/article/prototype-inheritance/proto-animal-rabbit-walk.png)

The prototype chain can be longer:

```js
 let animal = {
  eats: true,
  walk() {
    alert("Animal walk");
  }
};

let rabbit = {
  jumps: true,
  __proto__: animal
};

let longEar = {
  earLength: 10,
  __proto__: rabbit
}
```
// walk is taken from the prototype chain
longEar.walk(); // Animal walk
alert(longEar.jumps); // true (from rabbit)

![picture](https://javascript.info/article/prototype-inheritance/proto-animal-rabbit-chain.png)

There are actually only two limitations:

1. The references can’t go in circles. JavaScript will throw an error if we try to assign __proto__ in a circle.
2. The value of __proto__ can be either an object or null. All other values (like primitives) are ignored.
Also it may be obvious, but still: there can be only one [[Prototype]]. An object may not inherit from two others.

# Read/Write Rules
The prototype is only used for reading properties.

For data properties (not getters/setters) write/delete operations work directly with the object.

In the example below, we assign its own walk method to rabbit:
```js
 let animal = {
  eats: true,
  walk() {
    /* this method won't be used by rabbit */
  }
};

let rabbit = {
  __proto__: animal
}

rabbit.walk = function() {
  alert("Rabbit! Bounce-bounce!");
};

rabbit.walk(); // Rabbit! Bounce-bounce!
```
From now on, rabbit.walk() call finds the method immediately in the object and executes it, without using the prototype:

![picture](https://javascript.info/article/prototype-inheritance/proto-animal-rabbit-walk-2.png)

For getters/setters – if we read/write a property, they are looked up in the prototype and invoked.

For instance, check out admin.fullName property in the code below:
```js
 let user = {
  name: "John",
  surname: "Smith",

  set fullName(value) {
    [this.name, this.surname] = value.split(" ");
  },

  get fullName() {
    return `${this.name} ${this.surname}`;
  }
};

let admin = {
  __proto__: user,
  isAdmin: true
};

alert(admin.fullName); // John Smith (*)

// setter triggers!
admin.fullName = "Alice Cooper"; // (**)
```
Here in the line (*) the property admin.fullName has a getter in the prototype user, so it is called. And in the line (**) the property has a setter in the prototype, so it is called.

# The value of “this”
An interesting question may arise in the example above: what’s the value of this inside set fullName(value)? Where the properties this.name and this.surname are written: user or admin?

The answer is simple: this is not affected by prototypes at all.

No matter where the method is found: in an object or its prototype. In a method call, this is always the object before the dot.

So, the setter actually uses admin as this, not user.

That is actually a super-important thing, because we may have a big object with many methods and inherit from it. Then we can run its methods on inherited objects and they will modify the state of these objects, not the big one.

For instance, here animal represents a “method storage”, and rabbit makes use of it.

The call rabbit.sleep() sets this.isSleeping on the rabbit object:
```js
 // animal has methods
let animal = {
  walk() {
    if (!this.isSleeping) {
      alert(`I walk`);
    }
  },
  sleep() {
    this.isSleeping = true;
  }
};

let rabbit = {
  name: "White Rabbit",
  __proto__: animal
};

// modifies rabbit.isSleeping
rabbit.sleep();

alert(rabbit.isSleeping); // true
alert(animal.isSleeping); // undefined (no such property in the prototype)
```
The resulting picture:

![picture](https://javascript.info/article/prototype-inheritance/proto-animal-rabbit-walk-3.png)

If we had other objects like bird, snake etc inheriting from animal, they would also gain access to methods of animal. But this in each method would be the corresponding object, evaluated at the call-time (before dot), not animal. So when we write data into this, it is stored into these objects.

As a result, methods are shared, but the object state is not.

# Summary
In JavaScript, all objects have a hidden [[Prototype]] property that’s either another object or null.
We can use obj.__proto__ to access it (there are other ways too, to be covered soon).
The object referenced by [[Prototype]] is called a “prototype”.
If we want to read a property of obj or call a method, and it doesn’t exist, then JavaScript tries to find it in the prototype. Write/delete operations work directly on the object, they don’t use the prototype (unless the property is actually a setter).
If we call obj.method(), and the method is taken from the prototype, this still references obj. So methods always work with the current object even if they are inherited.


# Tasks
## Working With Prototype

Here’s the code that creates a pair of objects, then modifies them.

Which values are shown in the process?

```js
let animal = {
  jumps: null
};
let rabbit = {
  __proto__: animal,
  jumps: true
};

alert( rabbit.jumps ); // ? (1)

delete rabbit.jumps;

alert( rabbit.jumps ); // ? (2)

delete animal.jumps;

alert( rabbit.jumps ); // ? (3)
```
There should be 3 answers.

<details>
 <summary>Solution</summary>
<ol> 
 <li> true, taken from rabbit </li>
 <li> null, taken from animal </li>
 <li>undefined, there’s no such property any more</li>
 </ol>
</details>

# Why two hamsters are full?

* We have two hamsters: speedy and lazy inheriting from the general hamster object.

When we feed one of them, the other one is also full. Why? How to fix it?

```js
 let hamster = {
  stomach: [],

  eat(food) {
    this.stomach.push(food);
  }
};

let speedy = {
  __proto__: hamster
};

let lazy = {
  __proto__: hamster
};

// This one found the food
speedy.eat("apple");
alert( speedy.stomach ); // apple

// This one also has it, why? fix please.
alert( lazy.stomach ); // apple
```
<details><summary>Solution</summary>
Let’s look carefully at what’s going on in the call speedy.eat("apple").

The method speedy.eat is found in the prototype (=hamster), then executed with this=speedy (the object before the dot).

Then this.stomach.push() needs to find stomach property and call push on it. It looks for stomach in this (=speedy), but nothing found.

Then it follows the prototype chain and finds stomach in hamster.

Then it calls push on it, adding the food into the stomach of the prototype.

So all hamsters share a single stomach!

Every time the stomach is taken from the prototype, then stomach.push modifies it “at place”.

Please note that such thing doesn’t happen in case of a simple assignment this.stomach=:
```js
 let hamster = {
  stomach: [],

  eat(food) {
    // assign to this.stomach instead of this.stomach.push
    this.stomach = [food];
  }
};

let speedy = {
   __proto__: hamster
};

let lazy = {
  __proto__: hamster
};

// Speedy one found the food
speedy.eat("apple");
alert( speedy.stomach ); // apple

// Lazy one's stomach is empty
alert( lazy.stomach ); // <nothing>
```
Now all works fine, because this.stomach= does not perform a lookup of stomach. The value is written directly into this object.

Also we can totally evade the problem by making sure that each hamster has their own stomach:

```js
let hamster = {
  stomach: [],

  eat(food) {
    this.stomach.push(food);
  }
};

let speedy = {
  __proto__: hamster,
  stomach: []
};

let lazy = {
  __proto__: hamster,
  stomach: []
};

// Speedy one found the food
speedy.eat("apple");
alert( speedy.stomach ); // apple

// Lazy one's stomach is empty
alert( lazy.stomach ); // <nothing>
```

As a common solution, all properties that describe the state of a particular object, like stomach above, are usually written into that object. That prevents such problems.
</details>
