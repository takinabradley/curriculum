### Introduction

The previous lesson covered some basics of *why* and *how* you might organize data and functionality into objects.

The next important step is to learn about some strategies for creating duplicates of certain types of objects, and how you can use existing types of objects as a 'base' for creating new ones through a mechanism called *inheritance*.

The term **Object Inheritance** describes common methods in which the interfaces (read: *properties and methods*) of two or more types of objects are 'composed together' to create a new type of object. This is most commonly understood be be done through some kind of a 'parent -> child' relationship. i.e. The child 'inherits' properties from it's parent object.

Inheritance allows you to *utilize the existing functionality objects you already have* when creating new or similar ones. This can open up some neat organizational and code re-use strategies.

### Lesson overview

This section contains a general overview of topics that you will learn in this lesson.

- Object Inheritance
- Prototypical Inheritance
- Using `Object.create` and `Object.assign`.

### Prototypical Inheritance in JavaScript

<div class="lesson-note lesson-note--warning" markdown="1">

---

#### Confusion Point

The way that many people use the term 'inheritance' often stems from backgrounds in languages that create and manage these relationships differently than JavaScript, often with the idea of "Classes". JavaScript didn't have any concept of a "Class" until it's 2015 standard, and it wasn't widely supported until 2016. Under the hood, classes in JS *still use* JavaScript's mechanism of "Prototypical Inheritance", which you'll learn about shortly, and it's important to understand how it works.

If you've been exposed to the concepts of "Classical Inheritance" before, try to keep an open mind! JavaScript's prototypical inheritance model is pretty cool!

---

</div>

JavaScript has a pretty unique method of implementing inheritance, called **Prototypical Inheritance**. The primary idea behind Prototypical Inheritance that makes it different from how other languages might manage inheritance is the ability inherit properties and methods from other *real, existing objects* in your code.

Other languages often require you to write a "Class", or a declaration of all the properties and methods that might appear on a 'type' of object, before you're ever able to create one. You might also have to specify if a Class inherits from any other Class at that time. This is **not** the case in JavaScript. In JavaScript, you can create objects *on the fly*, anywhere, any time.

Instead of making you specify how your objects inherit from other objects up front, JavaScript allows you to manage this through something it calls the **Prototype Chain**. The Prototype Chain allows you to 'link' any object you want to an arbitrary parent object. By linking the objects together, the 'child' object will have access to the properties and methods of the parent.

When doing this, the parent object is often reffered to as the child's **Prototype Object**, or **prototype** for short. Each object can inherit from only one parent, though that 'parent' object can also inherit from *it's own* prototype, forming a 'chain' of inheritance that might look sometheing like so:

```txt
Grandparent Object -> Parent Object -> Child Object
```

The chain here starts with the "GrandParent Object", which would inherit nothing and only contain it's own properties. The next object in the chain, the "Parent Object", would have access to the "Grandparent Object's" properties and methods, and finally the 'Child Object' would have all access to *all* the properties and methods that exist on objects higher in the chain.

In addition, all objects you create with the `Object` constructor (`new Object()`) or with Object Literal syntax (`{}`) inherit from an object on the `Object` constructor function by default called `Object.prototype`.

It's important to understand `Object.prototype` is a real object that exists in JavaScript, and not anything magical. Go ahead, and open the developer console to log the `Object` constructor function. If your developer console just shows something like `ƒ Object() { [native code] }` when you log it, use `console.dir()` to see it's various properties and methods (functions are objects too, in JavaScript!). In that list, you'll see a property called `prototype` that contains all the default methods that exist on every object like [`Object.prototype.hasOwnProperty()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty) and [`Object.prototype.toString()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString).

Try using each of those methods on an object you create in the console!

```js
let myObj = {
  name: 'myObj'
}

// `myObj` already inherits default properties from Object.prototype!
myObj.hasOwnProperty('name') // true
myObj.toString() // "[object Object]" 
```

We can 'prove' to ourselves that `Object.prototype` is *really* the **prototype object** of `myObj` by using the [`Object.prototype.isPrototypeOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf) method:

```js
// is Object.prototype the prototype of our object?
Object.prototype.isPrototypeOf(myObject) // true!
```

Another way to view this relationship is through the JavaScript development console in your browser. In Chrome, log out `myObj` and inspect it's properties. In the output, you'll see a strange property called `[[Prototype]]` on it. This property's value contains the prototype object of `myObj`:

```js
Chrome Developer Tools Output:

Object {  }
  name: "myObj"
  [[Prototype]]: Object
    // Here, you'll see all the properties of Object.prototype
```

In fact, *all* objects in JavaScript have this `[[Prototype]]` property, even `Object.prototype`! `Object.prototype` is, however, the only default object that *cannot* have a prototype object. It's `[[Prototype]]` property instead points to `null` to represent the end of the prototype chain.

You cannot access the `[[Prototype]]` property of objects directly in JavaScript. It's kind of like a private property (remember those?) that JavaScript browsers and runtimes use to keep track of the prototype chain, but make easy for us to view in the console through developer magic. You can access the **prototype object** of any object itself, however, by using the [`Object.getPrototypeOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf) method.

<div class="lesson-note lesson-note--warning" markdown="1">

---

#### Object.getPrototypeOf() vs. .\_\_proto__ vs. \[\[Prototype]]

Use `Object.getPrototypeOf()` to access an object’s prototype. You may have seen that the same thing can also be done using the `.__proto__` property of an object. However, this is a non-standard way of doing so, and deprecated.

`[[Prototype]]` and `.__proto__` are both commonly used to refer internal/private `[[Prototype]]` property of an object in conversation, but `someObject.[[Prototype]]` is the notation that is used in the ECMAScript standard and should be preferred.

---

</div>

### Assigning your own Prototype Object

<div class="lesson-note lesson-note--warning" markdown="1">

---

#### Demonstration

It is generally not recommended to *change* the prototype of individual objects willy-nilly like this after they have been created. The following code is being used strictly as a bare-bones demonstration of how prototypical inheritance in JavaScript works.

We'll soon cover how to use `Object.create` to instantiate (or create), objects with their prototype already in-place.

---

</div>

The most straighfoward example of assigning your own Prototype Object to an object you create can be shown using the [`Object.setPrototypeOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf) method:

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

Log the `child` object in the Developer Console and inspect it. You should see something very similar to what you saw before, but this time the `parent` object is part of the `child` object's prototype chain!

```txt
Chrome Developer Tools Output:

Object {  }
  name: "child"
  [[Prototype]]: Object
    // the `parent` object
    greet: function greet()
    name: "parent"
    [[Prototype]]: Object { … }
      // Object.prototype
```

### Reading properties on the prototype chain

You may notice that the `greet` method said the name of the `child` object, and not the `parent`! There is a couple reasons for this:

- The object the `this` keyword points to *depends on the object a method is called from*. This is not a problem, and it's actually *really* useful functionality!
- When looking up a property on an object, the JavaScript engine starts looking for the property on the object you are accessing before searching upwards in the prototype chain, one object at a time. The first property with the name you're searching for will be given to you.

The prototype chain for our object looks like this, with the objects considered 'highest' or near the 'top' of the chain towards on the left:

```txt
null (end of chain) -> Object.prototype -> parent -> child 
```

So that means that any property you try to access will first be searched for on the object you're accessing- in this case the `child`, then `parent`, and finally `Object.prototype`. If the property doesn't exist on any of those objects, JavaScript will regognize that the chain has ended and you will recieve `undefined`.

### Assigning properties to objects

When using the assignment operator (`=`) to assign a value to an object's property, the key/value pair will *always* be assigned to the object reference you're assigning to, and not on any other object in the prototype chain.

This is useful, because often you'll want *many* objects to inherit from a prototype object, and you wouldn't want it to accidentally get modified.

```js
const parent = {
  name: ''
}

const child = {}

Object.setPrototypeOf(child, parent)
child.name // ''
child.name = 'child' // a 'name' property gets added to the child
child.name // 'child'
parent.name // '' -- The prototype object has not been modified
```

With the ending of this section, we've covered fundamental way that inheritance works in JavaScript. Really, that's all there is to it! Before you move on, we encourage you to play around with with the idea, and experiment with how it works, and take a breather if you need it.

### Creating and inheriting with Object.create and Object.assign

[`Object.create`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create) is a method used to... well... create a new object, using an existing object as it's prototype. Essentially, it combines the object creating process and setting of the prototype into one, simple step.

Object.create is a great first way to start creating multiples of objects of a specific 'type' in JavaScript, via prototypical inheritance.

Let's go back to our `lightbulb` object from the previous lesson, and explore how we might start doing something *useful* with inheritance.

```js
const lightbulb = {
  color: 'fluorescent white',
  lit: false,
  switchOn() {
    if(this.lit === true) return false
    this.lit = true
    return true
  },
  switchOff() {
    if (this.lit === false) return false
    this.lit = false
    return true
  }
}
```

Let's say this lightbulb is a basic type of object we know we'll want to use often and create many of. When we want to create *new* lightbulb, we can just use it as a prototype object!

That's super easy to do with `Object.create`:

```js
// newBulb is an empty object that has lightbulb as it's prototype object
let newBulb = Object.create(lightbulb)

// we inherit the properties of lightbulb!
newBulb.lit // false
newBulb.switchOn() // true
newBulb.lit // true
```

You'll might notice, upon inspection, that the `switchOn` method adds a `lit` property to our previously empty `newBulb` object with the value of `true`- but it *doesn't* change the value of the prototype we're inheriting from. That's great, because it means we can make as many individual lightbulbs as we'd like without effecting the object we're inheriting from!

Next, let's suppose we want a slightly different variant of our lightbulb. Maybe we want it to be *blue*. That's also super easy.

```js
let blueLightBulb = Object.create(lightbulb)
blueBulb.color = "blue"
blueBulb.color // blue
```

You may not like that you can't do this all in one statement with `Object.create` (well... you can... but it's very verbose and uses something called "property descriptors", a topic we'll not be covering here). But there is another useful method called [`Object.assign`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign). Essentially, all this method does is copy the properties of any 'source' objects you want into a target object. Let's take a look at how that works:

```js
let blueLightBulb = Object.assign(Object.create(lightbulb), {
  color: 'blue'
}) 

blueLightBulb.color // 'blue'
```

Here we use Object.assign and:

- Provide it a target object created using `Object.create`. That means the target object is now an empty object who's prototype object is `lightbulb`
- Provide it with a source object containing the additional information we want to assign to `blueLightBulb`
- The properties of the source object get copied to the target object, and we now have a blue 'lightbulb' object.

Finally, imagine we don't want just a simple variant of the base object `lightbulb`, but we want to create a new, but similar type of object with it. Perhaps an RGB lightbulb that allows us to change it's color whenever we want with a convenient method.

We can easily do that with `Object.create` and `Object.assign` as well!

```js
let rgbLightbulb = Object.assign(Object.create(lightbulb), {
  changeColor(newColor) {
    this.color = newColor
  }
})

// starts with the default color
rgbLightbulb.color // "fluorescent white" 

// but now it has a handy method to change it!
rgbLightbulb.changeColor('red')
rgbLightbulb.color // red
```

Let's take this same idea just a little bit further, using another object we created in the last lesson, the Rock Paper Scissors game.

Let's say we wanted to place a limit on how many rounds are in the game. We could pretty easily *inherit* from that object to extend it's basic functionality:

```js
const threeRoundRPS = Object.assign(Object.create(rps), {
  // add some new properties
  roundLimit: 3,
  currentRound: 1,

  // override the `playRound` function with new, but similar functionality
  playRound(playerChoice) {
    // error if someone tries to play past the round limit
    console.log('currentRound', this.currentRound, 'roundLimit', this.roundLimit)
    if(this.currentRound > this.roundLimit) throw new Error("No more rounds to play!")
    if(!this._options.includes(playerChoice)) throw new Error(`Expected 'rock', 'paper', or 'scissors', but got ${playerChoice}`)
    
    // and increment the round
    this.currentRound++
    const computerChoice = this._getRandomChoice()
    if(this._isTie(playerChoice, computerChoice)) {
      return "tie"
    } else if(this._isPlayerWinner(playerChoice, computerChoice)) {
      this.playerScore++
      return "player"
    } else {
      this.computerScore++
      return 'computer'
    }
  },

  // also override the `reset()` function
  reset() {
    this.playerScore = 0
    this.computerScore = 0
    this.currentRound = 1
  }
})

threeRoundRPS.playRound('rock')
threeRoundRPS.playRound('rock')
threeRoundRPS.playRound('rock')
threeRoundRPS.playRound('rock') // ERROR: No more rounds to play!
threeRoundRPS.getWinningPlayer() // someone...
threeRoundRPS.reset()

// Alternatively, we could pass this object to Object.create to create many individual three-round games
const anotherThreeRoundRPS = Object.create(threeRoundRPS)
```

Hopefully, this starts to highlight to you the *usefulness* of inheritance, and also the importance of having well thought-out interfaces. If we hadn't designed our base Rock Paper Scissors game the way we had, it may have been difficult to extend it's functionality when new features were required.

You'll notice that our extension added some new properties to our object, but it didn't actually change how we interact with what was already there very much. This is an important detail to think about when extending objects into the future like this.

### A problem appears

- While this pattern is pretty good for inheriting functionality, it can't easily be used to inherit the structure of objects containing data, and the objects can easily be misused if they are intended only to be used as a prototype object

- This pattern can't easily be used as an interface for a 'collection'- if you try to push to an array in the prototype, the array will change for all inherited objects

- This pattern could be improved by adding a `create` method to prototype objects allowing you to initialize properties

### More Content

~~This is the fundamental way that inheritance works in JavaScript. We encourage you to play around with with the concept, and experiment with how it works! The lesson has been pretty long and in-depth so far, and could have felt like a lot. Take a breather before moving on!~~

### Knowledge Check

The following questions are an opportunity to reflect on key topics in this lesson. If you can't answer a question, click on it to review the material, but keep in mind you are not expected to memorize or master this knowledge.

- Some check
