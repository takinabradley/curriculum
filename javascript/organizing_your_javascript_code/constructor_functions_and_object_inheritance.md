### Introduction

Now that you've got a basic understanding of *why* and *how* you might use objects to organize data and functionality into objects, it's important to about some basic strategies for creating creating duplicates of objects, and using existing types of objects as a base for creating new ones.

### Lesson overview

This section contains a general overview of topics that you will learn in this lesson.

- Constructor Functions and instantiating objects.
- Object Inheritance
- Prototypical Inheritance and the `[[Prototype]]` property
- Classical Inheritance
- Emulating Classical Inheritance in JavaScript
- The `.prototype` property of Constructor Functions

### Creating Duplicate Objects with Constructor Functions

When you have a specific *type* of object that you want to duplicate, one way to easily create them is by using a Constructor Function, which is a function that looks like this:

```js
function LightBulb(name, marker) {
  this.color = color
  this.lit = false
}
```

and which you use by calling the function with the special JavaScript keyword [`new`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new).

```js
const lightbulb = new Lightbulb('white');
console.log(lightbulb.color); // 'white'
```

Just like with objects created using the Object Literal method, you can add functions to the object:

```js
function LightBulb(color) {
  this.color = color
  this.lit = false
  this.switchOn = function() {
    if(this.lit === true) return false
    this.lit = true
    return true
  },
  this.switchOff = function() {
    if (this.lit === false) return false
    this.lit = false
    return true
  }
}

const redBulb = new LightBulb('red')
const yellowBulb = new LightBulb('yellow')
const greenBulb = new LightBulb('green');

redBulb.color; // 'red'
yellowBulb.color; // 'yellow'
greenBulb.color // 'green'

greenBulb.switchOn() // the cars can go now... or something
```

<div class="lesson-note" markdown="1">

#### Why Constructors?

You might be asking yourself why you couldn't just return an object you want from a function, instead of using `new` and some seemingly-magical `this` syntax. That's a valid question!

Constructor functions provide built-in features for easily implementing inheritance in JavaScript, a concept we'll be covering next!
</div>

### Object Inheritance

The term **Object Inheritance** describes common methods in which the interfaces of two (or more) types of objects are 'composed together' to create a new type of object. This is most commonly understood be be done through some kind of a 'parent -> child' relationship.

Inheritance allows you to *utilize the existing functionality objects you already have* when creating new or similar ones. This can open up some neat organizational and code re-use strategies.

The way that many people use the term 'inheritance' often stems from backgrounds in languages that create and manage these relationships differently than JavaScript, often with the idea "Classes". JavaScript didn't have any concept of a "Class" until it's 2015 standard, and it wasn't widely supported until 2016. Under the hood, classes *still use* JavaScript's mechanism of "Prototypical Inheritance", and it's important to understand how it works.

If you've been exposed to the concepts of "Classical Inheritance" before, try to keep an open mind! JavaScript's prototypical inheritance model is pretty cool!

### Prototypical Inheritance In JavaScript

JavaScript has a pretty unique method of implementing inheritance, called **Prototypical Inheritance**. The primary idea behind prototypical inheritance is the ability inherit from *existing objects*. JavaScript manages this through something called the **Prototype Chain**. The Prototype Chain allows you to 'link' any object you want to an arbitrary parent object. By linking the objects together, the child will have access to the properties and methods of the parent.

When doing this, the parent object is often reffered to as the child's **Prototype Object**, or **prototype** for short. Each object can inherit from only one prototype, though that 'parent' object can also inherit from *it's own* prototype, forming a 'chain' of inheritance that might look somtheing like so:

```txt
Grandparent Object -> Parent Object -> Child Object
```

The chain here starts with the 'GrandParent Object', which would inherit nothing. The next object in the chain, the 'Parent Object', would have access to the 'Grandparent Object's properties and methods, and finally the 'Child Object' would have all access to all the properties and methods available to the 'Parent Object'.

In addition, all objects you create with the `Object` constructor (`new Object()`) or with Object Literal syntax (`{}`) inherit from an object on the `Object` constructor function by default called `Object.prototype`.

It's important to understand `Object.prototype` is a real object that exists in JavaScript, and not anything magical. Go ahead, and open the developer console to log the `Object` constructor function. If your developer console just shows something like `ƒ Object() { [native code] }` when you log it, use `console.dir()` to see it's various properties and methods (functions are objects too, in JavaScript!). In that list, you'll see a property called `prototype` that contains all the default methods that exist on every object like [`Object.prototype.hasOwnProperty()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty) and [`Object.prototype.toString()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString).

Try using each of those methods on an object you create in the console!

```js
let myObj = {
  name: 'myObj'
}

// default properties our object inherits from Object.prototype!
myObj.hasOwnProperty('name') // true
myObj.toString() // "[object Object]" 
```

We can 'prove' to ourselves that `Object.prototype` is really the **prototype object** of our object by using the [`Object.prototype.isPrototypeOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf) method:

```js
// is Object.prototype the prototype of our object?
Object.prototype.isPrototypeOf(myObject) // true!
```

Another way to view this relationship is through the JavaScript development console in your browser. In Chrome, log out `myObj` and inspect it's properties. In the output, you'll see a strange property called `[[Prototype]]` on it (in fact, *all* objects in JavaScript have this property, even `Object.prototype`. `Object.prototype` is, however, the only default object that doesn't have a prototype object. It's `[[Prototype]]` property instead points to `null`):

```js
Object {  }
  name: "child"
  [[Prototype]]: Object
    // Here, you'll see all the properties of Object.prototype
```

You cannot access the `[[Prototype]]` property of objects directly in JavaScript. It's kind of like a private property (remember those?) that JavaScript browsers and runtimes use to keep track of the prototype chain, but make easy for us to view in the console through developer magic. You can access the **prototype object** of any object itself, however, by using the [`Object.getPrototypeOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf) method.

<div class="lesson-note lesson-note--warning" markdown="1">
#### Object.getPrototypeOf() vs. .__proto__ vs. [[Prototype]]

Use Object.getPrototypeOf() to access an object’s prototype. You may have seen that the same thing can also be done using the `.__proto__` property of an object. However, this is a non-standard way of doing so, and deprecated.

`[[Prototype]]` and `.__proto__` are both commonly used to refer internal/private `[[Prototype]]` property of an object, but `someObject.[[Prototype]]` is the notation that is used in the ECMAScript standard and should be preferred.
</div>

The simplest example of assigning your own `[[Prototype]]` object to an object can be shown using the [`Object.setPrototypeOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf) method:

<div class="lesson-note lesson-note--warning" markdown="1">

#### Demonstration

It is generally not recommended to set the prototype of individual objects willy-nilly like this. The following code is being used strictly as a bare-bones demonstration of how prototypical inheritance in JavaScript works

</div>

```js
const parent = {
  name: 'parent',
  greet() {
    console.log(`Hello, I'm the ${this.name}`)
  }
}

const child = {
  name: 'child'
}

Object.setPrototypeOf(child, parent)
child.greet() // logs "Hello, I'm the child" to the console! We inherited a method!

parent.isPrototypeOf(child) // true!
```

In Chrome, if you were to log the `child` object in the Developer Console and inspect it, you would see something like this, allowing you to inspect the entire prototype chain:

```txt
Object {  }
  name: "child"
  [[Prototype]]: Object
    // the `parent` object
    greet: function greet()
    name: "parent"
    [[Prototype]]: Object { … }
      // Object.prototype
```

You may notice that the `greet` method said the name of the `child` object, and not the `parent`! That's because the object the `this` keyword points to *depends on the object a method is called from*. This is not a problem, and it's actually *really* useful functionality.

This is the fundamental way that inheritance works in JavaScript. We encourage you to play around with with the concept, and experiment with how it works! The lesson has been pretty long and in-depth so far, and could have felt like a lot. Take a breather before moving on!

### Classical Inheritance

The ideas of Classical Inheritance is pretty simple. You have a 'Class', or a type of object, that inherits from another 'Class'.

Imagine you have a type of object you use a lot throughout your code. Perhaps this is the 'lightbulb' object we created earlier. Now, in your 'Lightbulb Simulator App', you want to be able to create new types of lightbulbs.

A Standard Lightbulb may simply have:

- A color
- A 'lit' status
- Methods to turn the lightbulb on and off

RGB Lighbulbs may additionally have:

- A method to change the color of the bulb on a whim

Instead of re-implementing the standard lightbulb functionality for RGB lightbulbs, you might have the RGB Lightbulbs *inherit* their basic interface from the Standard Lighbulb.

So, as an example, if the RBG Lightbulb object type inherits from Standard Lightbulb object type, it might have a public interface that looks like this:

- A color property
- A lit property
- A method to turn the lightbulb on and off
- A method to change the color of the bulb on a whim.

This idea can be seen pretty clearly demonstrated using JavaScript's Class Syntax, so let's take an early peek at it now:

```js
class Lightbulb {
  constructor(color) {
    this.color = color
    this.lit = false
  }

  // object methods
  switchOn() {
    if(this.lit === true) return false
    this.lit = true
    return true
  }

  switchOff() {
    if (this.lit === false) return false
    this.lit = false
    return true
  }
}

// this class 'extends', or inherits, from `Lighbulb`
class RGBLightBulb extends Lightbulb {
  changeColor(newColor) {
    this.color = newColor
  }
}

let rgbLightbulb = new RGBLightBulb('green')

// The RGB lightbulb has all the properties and methods of a normal lightbulb
// and it also has a new method called `changeColor`
rgbLightbulb.color // green
rgbLightbulb.changeColor('red')
rgbLightbulb.color // red
rgbLightbulb.lit // false
rgbLightbulb.switchOn() // true
```
