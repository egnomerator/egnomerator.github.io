## Helpful Links
["Prototypes Explained, Part 1"](https://app.pluralsight.com/player?course=advanced-javascript&author=kyle-simpson&name=advanced-javascript-m4&clip=1)
* This link is to a video within a Pluralsight course "Advanced JavaScript" by Kyle Simpson
* This page uses information from his course including his diagram as well as some example code and information from his lectures

[ECMAScript on Prototype Chain](https://www.ecma-international.org/ecma-262/7.0/index.html#sec-objects)

[[Keyword this]]
* This is a link to another page within this github wiki which covers the keyword `this`

[[Built-In JavaScript Properties]]
* This is a link to another page within this github wiki which discusses certain built-in JavaScript object properties integral to the prototype chain

---

### What is the Prototype Chain

From [the ES 2016 spec](https://www.ecma-international.org/ecma-262/7.0/index.html#sec-objects):
> Every object created by a constructor has an implicit reference (called the object’s prototype) to the value of its constructor’s "prototype" property. Furthermore, a prototype may have a non-null implicit reference to its prototype, and so on; this is called the prototype chain. When a reference is made to a property in an object, that reference is to the property of that name in the first object in the prototype chain that contains a property of that name. In other words, first the object mentioned directly is examined for such a property; if that object contains the named property, that is the property to which the reference refers; if that object does not contain the named property, the prototype for that object is examined next; and so on.

Any given object has a link to a prototype object which is linked to another prototype object and so on, until the end of the chain is reached. The prototype chain ends with `Object.prototype` who's internal property `[[Prototype]]` value is `null`--I'll call this the "root object".

The prototype chain facilitates inheritance; the specific term "prototype-based inheritance" is used in the ECMAScript spec ("prototypal inheritance" is a commonly used term; as a side note, some debate that the term delegation should be used rather than inheritance).

In graph form, the prototype chain resembles a tree (see [ES graph](https://www.ecma-international.org/ecma-262/7.0/index.html#sec-objects) and the graphs in this wiki page). The prototype chain has branches that link upward toward the trunk (and ultimately the root object). The linkages are linear; an object is not linked to all other objects in the prototype chain, it is linked to _only_ the objects that are on that same branch leading back to the root object.

#### How does the prototype chain work?

Consider this pseudo-code as an example for illustration purposes:
```
new object A { 
    sayHi(){ print("HI!"); } 
}

new object B {} linked to A

B.sayHi();    // prints HI!
```
##### What this code does:

Here, object A is created and given a method `sayHi`. Then object B is created and linked to object A (making A the prototype object that B is linked to). Object B is not given any method when it is created; it is an empty object. And yet, we are able to call `B.sayHi()` as though `sayHi` were a method on object B.

##### How B can call method sayHi:

When `B.sayHi()` is executed, the method `sayHi` is not found on object B--this immediately triggers a search for method `sayHi` via the prototype chain. Object A is the prototype of object B, so after the attempt to call `sayHi` via object B, there is an attempt to call `sayHi` via object A. Since object A does have that method, the method is called.

##### The Prototype Chain and `this`
Let's take the above pseudo-code one step further:
```
new object A { 
    name: "Dennis"
    sayHi(){ print("HI " + this.name + "!"); } 
}

new object B { name: "Mac" } linked to A

B.sayHi();    // prints HI Mac!
```
Here, the code works just the same as the above code with this additional behavior:
* After traversing the prototype chain from object B to object A, when `sayHi` is executed, `this` points to object B because object B was the calling object

Here is JavaScript code using the prototype chain and the `this` keyword:
```javascript
function A(){
    name = "Dennis";
}
A.prototype.sayHi = function(){ console.log("HI " + this.name + "!");};
B = Object.create(A.prototype);
B.name = "Mac";

B.sayHi();      // prints HI Mac!
```
##### How it works:
* Setup
  * Function A has its own prototype object (`A.prototype`) which becomes the prototype object that object B is linked to when object B is created.
* Use of Prototype Chain
  * Object B does not have function `sayHi` so object B's prototype chain link is traversed and `sayHi` is called on `A.prototype`
* Use of `this` Keyword
  * When `sayHi` is called on `A.prototype`, `this` points to B since B originally called `sayHi`

## Deep Dive

![Prototype Linkages Deep Dive Sections](Images/js_diagram_prototype_linkages_v2_brokenCtorProp_sections.png)

This diagram (a modified version of Kyle Simpson's) shows the linkages that are created when creating functions and objects in JavaScript. Let's take a section-by-section approach to this diagram (2 main sections, blue and red).

### The blue section illustrates _non_-built-in object creation:

Beginning with this [video](https://app.pluralsight.com/player?course=advanced-javascript&author=kyle-simpson&name=advanced-javascript-m4&clip=1), Kyle Simpson's course on Pluralsight includes a thorough explanation of what happens when an example code snippet is executed--his diagram is integral to the explanation (Kyle's original diagram is at the end of this wiki page). For this deep dive area, I have created a modified version of his diagram; I'm using his example code; and I'm reproducing his step-by-step explanation.

![Prototype Linkages Deep Dive Object Creation](Images/js_diagram_prototype_linkages_v2_brokenCtorProp_obj_creation.png)

Let's work through this code snippet and observe how the diagram is formed.
```javascript
 1. function Foo(who){
 2.     this.me = who;
 3. }
 4. Foo.prototype.identify = function(){
 5.     return "I am " + this.me;
 6. }
 7. 
 8. function Bar(who){
 9.     Foo.call(this,who);
10. }
11. // Bar.prototype = new Foo();
12. Bar.prototype = Object.create(Foo.prototype);
13. // Bar.prototype.constructor = Bar;
14. // Line 13 is commented out to match the diagram
15. // -this will be explained
16. 
17. Bar.prototype.speak = function(){
18.     console.log("Hello, " + this.identify() + ".");
19. }
20. var b1 = new Bar("b1");
21. var b2 = new Bar("b2");
22. 
23. b1.speak();     // Hello, I am b1.
24. b2.speak();     // Hello, I am b2.
25. 
26. console.log(b1.constructor === Foo);
27. console.log(b1.constructor === Bar);
28. console.log(b1.__proto__ === Bar.prototype);
29. console.log(b1.__proto__ === b2.__proto__);
```
Pretend the diagram is simply blank initially. We will "draw" this diagram as we go through this code snippet line-by-line to see what's happening.

#### Step Zero (built-in objects):
* Code (n/a)
* Diagram
  * Before line 1 in the code executes, the top 4 objects (`Object()`, `Object.prototype`, `Function()`, and `Function.prototype`) and their linkages have been drawn/created
  * These objects will be discussed later, when we cover the red section of the diagram
#### Step One (Foo):
* Code
  * On line 1 Function `Foo` is declared
* Diagram
  * Function `Foo()` is drawn/created
    * Has a property `.prototype` which points to (dotted arrow) its prototype object `Foo.prototype`
    * Has an internal property `[[Prototype]]` which is its link to the prototype chain
      * This internal link is from `Foo()` directly to (solid arrow) `Function.prototype`
  * `Foo.prototype` is drawn/created
    * Has a `.constructor` property which points to (dotted arrow) `Foo()`
    * Has an internal property `[[Prototype]]` which is its link to the prototype chain
      * This internal link is from `Foo.prototype` directly to (solid arrow) `Object.prototype`
#### Step Two (function on Foo.prototype):
* Code
  * On line 4 a property `identify` is created on `Foo.prototype` and is assigned a function expression
* Diagram
  * "identify()" is drawn inside `Foo.prototype`
#### Step Three (Bar):
* Code
  * On line 8 Function `Bar` is declared
* Diagram
  * Function `Bar()` is drawn/created
    * Has a property `.prototype` which points to (dotted arrow) its prototype object `Bar.prototype`
    * Has an internal property `[[Prototype]]` which is its link to the prototype chain
      * This internal link is from `Bar()` directly to (solid curved arrow) `Function.prototype`
  * `Bar.prototype` is drawn/created
    * This object is not on the diagram, because it will be replaced by a new object created on line 12; but, if this object were on the diagram, it would be the same pattern as seen for `Foo()` and `Foo.prototype` as shown in these next two sub-steps
    * Has a `.constructor` property which points to (dotted arrow) `Bar()`
    * Has an internal property `[[Prototype]]` which is its link to the prototype chain
      * This internal link is from `Bar.prototype` directly to (solid arrow) `Object.prototype`
#### Step Four (Explicit call to Foo):
* Code
  * On line 9 an explicit call to `Foo` passing the `this` context and the `who` argument is written
* Diagram
  * `Foo.call(this,who)` is drawn inside `Bar()`

Note: Line 11 is included as a comment just to note that it would accomplish the exact same thing as line 12

#### Step Five (explicitly set Bar.prototype):
* Code
  * On line 12 a new object is created passing in `Foo.prototype` which causes the newly created object's internal `[[Prototype]]` property to point to `Foo.prototype`
    * This new object is assigned to `Bar.prototype`
* Diagram
  * As shown, the object labeled `Bar.prototype` is drawn/created
    * Has an internal property `[[Prototype]]` which is its link to the prototype chain
      * This internal link is from the object labeled `Bar.prototype` directly to (solid arrow) `Foo.prototype`
  * `Bar`'s prototype property (dotted arrow) now points to the object labeled `Bar.prototype`
    * The object can now be referred to as `Bar.prototype`
  * ~~Bar.prototype has a .constructor property which points to (dotted arrow) Bar()~~
      * **note**: since this object was not created automatically along with a function object (e.g. like `Foo.prototype` was when `Foo()` was declared--which is how prototype objects are normally created), this object does not have a `constructor` property
      * This would be different [if line 13 were not commented out](#significance-of-line-13)
#### Step Six (function on Bar.prototype):
* Code
  * On line 17 a property `speak` is created on `Bar.prototype` and is assigned a function expression
* Diagram
  * "speak()" is drawn inside `Bar.prototype`
#### Step Seven (create objects):
* Code
  * On line 20 and 21, objects `b1` and `b2` are created via the `new` keyword on `Bar()`
* Diagram
  * Objects b1 and b2 are drawn/created
  * Each object is given a `me` property via the explicit call to `Foo()` within function `Bar()`
  * Each object has an internal property `[[Prototype]]` which is its link to the prototype chain
      * These internal links are from `b1` and `b2` directly to (solid arrow) `Bar.prototype`
  * Thanks to each object's link to the prototype chain, they each have access to `Bar.prototype`'s speak() function and `Foo.prototype`'s identity() function

##### Now the code is prepared for the remaining lines of code to execute using the functionality that was setup--the diagram is finished and illustrates this prepared functionality; the finished diagram can be used to predict the results of the remaining lines of code.

#### Step Eight (call speak):
* Code
  * On lines 23 and 24, objects `b1` and `b2` call the speak function
  * The speak function contains a call to the identify function
* Diagram
  * The finished diagram predicts what will occur (the same behavior occurs for both `b1` and `b2`)
    * The object does not have a speak property
    * Traversal of the prototype chain to `Bar.prototype` (via solid arrow internal `[[Prototype]]` linkage)
    * `Bar.prototype` has a speak property which is called
    * From the code we know that function speak begins preparing a message to print to the console calling property identify on the `this` keyword
    * The object that `this` points to (b1/b2) does not have an identify property
    * Traversal of the prototype chain to `Bar.prototype` (via solid arrow internal `[[Prototype]]` linkage)
    * `Bar.prototype` does not have an identify property
    * Traversal of the prototype chain to `Foo.prototype` (via solid arrow internal `[[Prototype]]` linkage)
    * `Foo.prototype` has an identify property which is called
    * From the code we know that function identify begins preparing a string to return calling property me on the `this` keyword
    * The object that `this` points to (b1/b2) has a me property
      * `b1.me` was set to "b1" and `b2.me` was set to "b2"

#### Step Nine (logging Boolean values):
* Line 26: `b1.constructor` points to `Foo`
  * Result: true
  * Diagram: `b1` does not have a constructor property -> nor does `Bar.prototype` -> `Foo.prototype.constructor` points to function `Foo`
* Line 27: `b1.constructor` points to `Bar`
  * Result: false
  * Diagram: There is no way to reach `Bar` from `b1`'s prototype chain
* Line 28: `b1`'s internal prototype linkage points to `Bar.prototype`
  * Result: true
  * Diagram: `b1` does not have a \_\_proto\_\_ property -> nor does `Bar.prototype` -> nor does `Foo.prototype` -> `Object.prototype.__proto__` returns the calling object's (`b1`'s) internal `[[Prototype]]` linkage (solid arrow) which points to `Bar.prototype`
* Line 29: `b1`'s internal prototype linkage points to the same thing as `b2`'s internal prototype linkage
  * Result: true
  * Diagram: `b1`'s internal `[[Prototype]]` linkage (solid arrow) and `b2`'s internal `[[Prototype]]` linkage (solid arrow) each point to `Bar.prototype`

#### Significance of Line 13<a id="significance-of-line-13"></a>
* As explained above in step five, `Bar.prototype` does not have a constructor property, and this is due to the fact that `Bar()`'s automatically created prototype object which _did_ have the constructor property was explicitly replaced by a new object--this replacement was to accomplish linking `Bar.prototype` to `Foo.prototype` in the prototype chain
* Line 13, when not commented out, explicitly sets `Bar.prototype`'s constructor property to function `Bar()`
  * Doing this means that `b1.constructor` and `b2.constructor` result in the expected `Bar()` rather than `Foo()`

##### What could go wrong?
Running the code snippet as is (with line 13 commented out) demonstrates that it technically still works. However, not taking the step of explicitly setting the constructor property, and thus leaving that prototype object without a constructor property, can potentially lead to issues.
* E.g. Say you're using an API, and you need more objects of the same kind as the one provided by the API, but the API does not provide a way to get more of these objects. You could create more using that object's constructor.
```javascript
var myAPI = (function(){
    function Earthling(){}
    function Person(){}
    Person.prototype = Object.create(Earthling);
    // Person.prototype.constructor = Person;                   // NEED THIS!
    Person.prototype.sayHi = function(){console.log("HI!")};

    var p1 = new Person();
    return {
        getP1: function(){ return p1; }
    }
})();

var p1 = myAPI.getP1();
// I want another person, but don't have access via the API
// -so I will use the constructor
var p2 = new p1.constructor();

p1.sayHi();     // HI!
p2.sayHi();     // Error: "p2.sayHi is not a function"
```
`p1.sayHi()` works as expected, however, `p1.constructor` does _not_ behave as we might expect. We expect `p1.constructor` to cause traversal of the prototype chain to `Person.prototype` and then to call that prototype object's constructor property which would point to `Person()` which would construct a new Person object for `p2`.

But since `Person.prototype.constructor` was never set after setting `Person.prototype` to the new object, `Person.prototype` has no constructor, which causes traversal further up the prototype chain to `Earthling.prototype` which _does_ have a constructor property. This means `p2` is an Earthling object which has no access to `sayHi()`, not even via the prototype chain.

This is obviously a contrived scenario with equally contrived code, but the point is that the constructor property makes a difference, and it's good to be aware of potential related issues. Whenever, you manually set a function's prototype object, not explicitly setting the constructor property of that prototype object could lead to debugging headaches and broken code.

* **note:** the version of the diagram at the end of this wiki page includes the `constructor` from `Bar.prototype` to `Bar` to illustrate how it should look with line 13 not commented out

### The red section illustrates the top level built-in object linkages:

![Prototype Linkages Deep Dive Root Objects](Images/js_diagram_prototype_linkages_v2_brokenCtorProp_root_objs.png)

As stated earlier, whenever a JavaScript program runs, before a single line of your code executes, all of these built-in objects and their linkages are created (`Object()`, `Object.prototype`, `Function()`, and `Function.prototype`).
  * The function `Object()` is drawn/created
    * Has an internal property `[[Prototype]]` which is its link to the prototype chain
      * This internal link is from `Object()` directly to (solid arrow) `Function.prototype`
    * Has a property `prototype` which points to (dotted arrow) `Object.prototype`
  * `Object.prototype` is drawn/created
    * The built-in object's `[[Prototype]]` value is null and thus is the end of the prototype chain
    * Has some base functions on it which all objects have access to via the prototype chain
    * Has a `constructor` property which points to (dotted arrow) `Object()`
    * Has a `__proto__` property which is a getter/setter accessible by all objects via the prototype chain
      * This property can be used by any object to reference the prototype object to which that calling object is linked
      * E.g. From the diagram: `b2.__proto__` would retrieve `Bar.prototype`
  * The function `Function()` is drawn/created
    * Has an internal property `[[Prototype]]` which is its link to the prototype chain
      * This internal link is from `Function()` directly to (solid arrow) `Function.prototype`
    * Has a property `prototype` which points to (dotted arrow) `Function.prototype`
  * `Function.prototype` is drawn/created
    * Has an internal property `[[Prototype]]` which is its link to the prototype chain
      * This internal link is from `Function.prototype` directly to (solid arrow) `Object.prototype`
    * Has some base functions on it which all function objects have access to via the prototype chain
    * Has a `constructor` property which points to (dotted arrow) `Function()`

From the diagram, there is a resemblance of a an incomplete circuit of linkages from `Object()` going counterclockwise to `Object.prototype`. In fact, you can access any of these objects by calling properties on any one of them and chaining their properties:
* E.g. #1 `Object.__proto__` points to `Function.prototype` and
* E.g. #2 `Object.__proto__.constructor` points to `Function()`
* E.g. #3 `Object.__proto__.constructor.prototype` points to `Function.prototype`
* E.g. #4 `Object.__proto__.constructor.prototype.__proto__` points to `Object.prototype`
* E.g. #5 `Object.__proto__.__proto__` also points to `Object.prototype`
* E.g. #6 `Object.__proto__.__proto__.constructor` points to `Object()`

These built-in objects are at the top of the prototype chain with `Object.prototype` being the top-most (`Object.prototype.__proto__` is `null`). New functions are linked to prototype object `Function.prototype`; new objects are linked to prototype object `Object.prototype`; and all objects whether function objects or non-function objects are either directly or indirectly linked to `Object.prototype`.

## The Clean Diagrams

Below is Kyle's original diagram who's design appeared to be driven by a specific approach. And then my version is below Kyle's. My attempt was to focus on clarity of the illustration of the prototype chain. It could be very helpful to compare these diagrams carefully studying the differences in order to understand the prototype chain as well as see what Kyle was emphasizing.

#### Some Differences of Note
##### A Distracting Apparent Intent of Kyle's: Debunk Common Misconceptions
* Given certain emphases in his diagram, and his lecture, Kyle seemed focused on debunking common JS misconceptions
  * E.g. he wanted to emphasize that all of those objects do _not_ have their own `constructor` property--they actually access the `constructor` property of the prototype object they are linked to via the prototype chain
* Without the context Kyle provides with his lecture, I think the diagram on its own could be misleading
  * His lecture goes perfectly with his diagram and is very clear and thorough, so I highly recommend his lecture, and my deep dive above is my attempt to reproduce the points in his lecture
##### My Singular Intent: Clearly Illustrate the Prototype Chain
* My version of Kyle's diagram differs from his in a number of ways
  * I do not include visualization of implied constructor property relationships (his dotted arrows)
    * I think including all these constructor property dotted arrows is confusing
      * It makes it look like each of those properties exist on the object the arrow is coming from, but they do not
      * Yes, the legend is intended to clarify this, but I just don't think it's clear
        * (as mentioned above, however, in combination with Kyle's lecture, it is clear)
    * It is not necessary. The express purpose of the diagram is to illustrate how the prototype chain works
      * If you focus on the prototype chain, which is my focus in the diagram, you can follow the prototype chain and see how the objects have access to the constructor property of the prototype objects they are linked to
  * I don't put `__proto__`'s by every object
    * Again, this makes it look like each of those properties exist on the object the arrow is coming from, but they do not
  * I put the `[[Prototype]]` internal properties inside each object letting the legend denote the linkage
    * The internal `[[Prototype]]` linkage is a unique property type, and I think simply symbolizing this with the solid arrow makes sense; doing this also removes the need to clarify what the solid arrow means with a `[[Prototype]]` label on each arrow
    * Given that it is unnecessary to label each solid arrow with `[[Prototype]]`, it now makes most sense to visibly include it within each object
  * I use dotted arrows to visualize the property pointers between constructors and prototype objects

This is Kyle Simpson's original diagram from his [course](https://app.pluralsight.com/player?course=advanced-javascript&author=kyle-simpson&name=advanced-javascript-m4&clip=1)

![Prototype Linkages: Original](Images/js_diagram_prototype_linkages.png)

This diagram is a version of Kyle's that I made with a focus on clarity of the actual prototype chain linkages.

![Prototype Linkages Deep Dive Clean](Images/js_diagram_prototype_linkages_v2_brokenCtorProp.png)

#### For Practice
Here is the diagram reflecting the code from the deep dive _IF_ line 13 from that code had not been commented out (i.e. if `Bar.prototype`'s constructor property had been manually set). See if you can determine how this diagram is different from the deep dive diagram and what effects this difference has.

_Hint: the answer is explained [here](#significance-of-line-13) within the deep dive section._

![Prototype Linkages Clean](Images/js_diagram_prototype_linkages_v2.png)