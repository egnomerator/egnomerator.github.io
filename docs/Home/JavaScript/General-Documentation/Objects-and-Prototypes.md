## Helpful Links
["Demo: Classes"](https://app.pluralsight.com/player?course=javascript-practical-design-patterns&author=jonathan-mills&name=javascript-practical-design-patterns-m3&clip=6&mode=live)
* This link is to a video within a Pluralsight course "Practical Design Patterns in JavaScript" by Jonathan Mills
* Some of the information and code in this wiki page is from this course

[[Prototype Linkage Helper Function]]
* This is a link to another page within this github wiki
* This page has the definition of `getObjectInfo()` which is used throughout this wiki page

[[Built-In JavaScript Properties]]
* This is a link to another page within this github wiki
* This page covers some key properties that are accessed and printed in the example code
* The material covered below assumes an understanding of these properties

[[Prototype Chain]]
* This is a link to another page within this github wiki
* This page explains what the prototype chain is and how it works
* The material covered below assumes an understanding of the prototype chain

---

## Creating Objects

There are many different ways to create objects in JavaScript. Some of the more common ways are demonstrated below. With each approach to creating objects in JavaScript, I'll discuss some possible reasons to use or not to use the given syntax.

**Note:** It's important to have an understanding of the prototype chain and also the built-in JavaScript properties printed out in these code snippets. With this understanding, analyzing the commented output at the end of each of these code examples will be helpful in seeing how the different ways of creating objects can effect how the objects will behave generally and how they will interact.

#### Object Literal Syntax
```javascript
// Object literal
var objLiteral = {
    name: "my name",
    fn: function(){
        return "this is a function";
    }
}

getObjectInfo(objLiteral);
// var obj2 = new objLiteral();                     // error: objLiteral is not a constructor

// OUTPUT BELOW:
// Below is "meta info" of this object "o" ...
// o:
// { name: 'my name', fn: [Function: fn] }
// o.prototype
// undefined
// Object.getPrototypeOf(o)
// {}
// o.constructor
// [Function: Object]
```
This is a basic way to create an object in JavaScript. As the log statements show, there is not much to this object--all the built-in JavaScript properties are default values.

* The object literal syntax can be very handy when there is a need to pass data and/or behavior around via anonymous objects
  * Some common patterns do this (e.g. Module Pattern)
* A limitation of this syntax is that you cannot construct new objects with it which is something the Constructor Function syntax is for

#### Constructor Function Syntax
```javascript
// Constructor Function
function ctorFn(){
    this.name = "my name";
    this.fn = function(){
        return "this is a function";
    };
}

getObjectInfo(ctorFn);
console.log("_______________________________________________________");
var obj2 = new ctorFn();
getObjectInfo(obj2);

// OUTPUT BELOW:
// Below is "meta info" of this object "o" ...
// o:
// [Function: ctorFn]
// o.prototype
// ctorFn {}
// Object.getPrototypeOf(o)
// [Function]
// o.constructor
// [Function: Function]
// _______________________________________________________
// Below is "meta info" of this object "o" ...
// o:
// ctorFn { name: 'my name', fn: [Function] }
// o.prototype
// undefined
// Object.getPrototypeOf(o)
// ctorFn {}
// o.constructor
// [Function: ctorFn]
```
What happens:
* We declare a function (which is an object)
* Then we print out that object's information
* Then we use that function as a constructor to create a new object with the `new` keyword
  * See [[this|https://github.com/egnomerator/misc/wiki/Keyword-%60this%60#this-nuances--example-7]] explanation of what happens when the `new` keyword is used in front of a function to create an object
* Then we print out that second object's information

A clear benefit with this syntax is the ability to construct objects which are linked to the function's prototype object.

#### Object.create() Syntax
```javascript
// Object literal
var objLiteral = {
    name: "my name",
    fn: function(){
        return "this is a function"
    }
}

getObjectInfo(objLiteral);
console.log("_______________________________________________________");
var obj2 = Object.create(objLiteral);
getObjectInfo(obj2);
// OUTPUT BELOW:
// Below is "meta info" of this object "o" ...
// o:
// { name: 'my name', fn: [Function: fn] }
// o.prototype
// undefined
// Object.getPrototypeOf(o)
// {}
// o.constructor
// [Function: Object]
// _______________________________________________________
// Below is "meta info" of this object "o" ...
// o:
// {}
// o.prototype
// undefined
// Object.getPrototypeOf(o)
// { name: 'my name', fn: [Function: fn] }
// o.constructor
// [Function: Object]
```
The object passed into `Object.create()` becomes the prototype object that the new object is linked to in the prototype chain. In this example we have passed in that object literal.

The `Object.create()` syntax is an alternative to using the `new` keyword. It allows you to create objects linked to a prototype object that was simply created as an object literal, whereas the `new` keyword, as stated earlier does not work on object literals.

#### Object.create() Syntax - Example 2
```javascript
// Constructor Function
function ctorFn(){
    this.name = "my name";
    this.fn = function(){
        return "this is a function"
    };
}

getObjectInfo(ctorFn);
console.log("_______________________________________________________");
var obj2 = Object.create(ctorFn.prototype);
getObjectInfo(obj2);
// OUTPUT BELOW:
// Below is "meta info" of this object "o" ...
// o:
// [Function: ctorFn]
// o.prototype
// ctorFn {}
// Object.getPrototypeOf(o)
// [Function]
// o.constructor
// [Function: Function]
// _______________________________________________________
// Below is "meta info" of this object "o" ...
// o:
// ctorFn {}
// o.prototype
// undefined
// Object.getPrototypeOf(o)
// ctorFn {}
// o.constructor
// [Function: ctorFn]
```
What's different here from the first example of the `Object.create()` syntax is that we are specifically passing in an object that the function we declared points to via it's prototype property which is a fairly common pattern.

#### class Syntax (introduced in ES6)
```javascript
class Task {
    constructor(name){
        this.name = name;
        this.completed = false;
    };
    complete(){
        console.log("completing task: " + this.name);
        this.complete = true;
    };
    save(){
        console.log("saving task: " + this.name);
    };
}

getObjectInfo(Task);
console.log("_______________________________________________________");
var task1 = new Task("create a demo for constructors");
getObjectInfo(task1);

// additional output to show that the prototype complete and save properties on the prototype
// -the class syntax simply defines those prototype properties as not enumerable
// -the Object.getOwnPropertyNames(Task.prototype) line also prints non-enumerable properties
console.log("_______________________________________________________");
console.log("Task");
console.log(Object.keys(Task));
console.log(Object.getOwnPropertyNames(Task));
console.log("Task.prototype");
console.log(Object.keys(Task.prototype));
console.log(Object.getOwnPropertyNames(Task.prototype));

// OUTPUT BELOW:
// Below is "meta info" of this object "o" ...
// o:
// [Function: Task]
// o.prototype
// Task {}
// Object.getPrototypeOf(o)
// [Function]
// o.constructor
// [Function: Function]
// _______________________________________________________
// Below is "meta info" of this object "o" ...
// o:
// Task { name: 'create a demo for constructors', completed: false }
// o.prototype
// undefined
// Object.getPrototypeOf(o)
// Task {}
// o.constructor
// [Function: Task]
// _______________________________________________________
// Task
// []
// [ 'length', 'name', 'prototype' ]
// Task.prototype
// []
// [ 'constructor', 'complete', 'save' ]
```
This output should look very familiar, and that's because there is no class entity in JavaScript as a first-class citizen--it is just "syntactic sugar". In fact, this class syntax accomplishes the same thing as the below syntax which uses the function declaration syntax along with explicitly adding prototype properties:

```javascript
function Task (name){
    this.name = name;
    this.completed = false;
};
Task.prototype.complete = function(){
    console.log("completing task: " + this.name);
    this.complete = true;
};
Task.prototype.save = function(){
    console.log("saving task: " + this.name);
};

getObjectInfo(Task);
console.log("_______________________________________________________");
var task1 = new Task("create a demo for constructors");
getObjectInfo(task1);

// additional output to show that the prototype complete and save properties on the prototype
// -including this for direct comparison to output in the class syntax example
console.log("_______________________________________________________");
console.log("Task");
console.log(Object.keys(Task));
console.log(Object.getOwnPropertyNames(Task));
console.log("Task.prototype");
console.log(Object.keys(Task.prototype));
console.log(Object.getOwnPropertyNames(Task.prototype));

// OUTPUT BELOW:
// Below is "meta info" of this object "o" ...
// o:
// [Function: Task]
// o.prototype
// Task { complete: [Function], save: [Function] }
// Object.getPrototypeOf(o)
// [Function]
// o.constructor
// [Function: Function]
// _______________________________________________________
// Below is "meta info" of this object "o" ...
// o:
// Task { name: 'create a demo for constructors', completed: false }
// o.prototype
// undefined
// Object.getPrototypeOf(o)
// Task { complete: [Function], save: [Function] }
// o.constructor
// [Function: Task]
// _______________________________________________________
// Task
// []
// [ 'length', 'name', 'arguments', 'caller', 'prototype' ]
// Task.prototype
// [ 'complete', 'save' ]
// [ 'constructor', 'complete', 'save' ]
```
Theoretically, the advantage to the class syntax is readability--the idea here being that people who are used to working with classes in other languages may find the syntax more intuitive.