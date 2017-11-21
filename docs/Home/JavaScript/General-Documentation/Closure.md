## Helpful Links
["Closures to the Rescue"](https://app.pluralsight.com/player?course=structuring-javascript&author=dan-wahlin&name=structuring-javascript-module1&clip=3)
* This link is to a video within a Pluralsight course "Structuring JavaScript Code" by Dan Wahlin
* Some of the information and code in this wiki page is from this course

["Closures"](https://app.pluralsight.com/player?course=advanced-javascript&author=kyle-simpson&name=advanced-javascript-m3&clip=0)
* This link is to a video within a Pluralsight course "Advanced JavaScript" by Kyle Simpson
* Some of the information and code in this wiki page is from this course

[[IIFE]]
* This is a link to another page within this github wiki

---

Functions remember their original lexical environment (the scope in which they are declared). This is referred to as closure. With closure, a private reference can be encapsulated such that the closure provides controlled access to that reference from within another scope that would otherwise not be able to access that reference. An example use case might be an API (see the module pattern example in [[IIFE]]).

Douglas Crockford on closures:
> ...an inner function always has access to the vars and parameters of its outer function, even after the outer function has returned.


#### example.js
```javascript
function myNonClosure(){
    var date = new Date();
    console.log("nonClsr:", date.getMilliseconds());
}
function myClosure(){
    var date = new Date();
    return function (){
        console.log("Closure:", date.getMilliseconds());
    }
}

var myNon = myNonClosure;
myNon();
setTimeout(function(){myNon();},500);

var myClsr = myClosure();
myClsr();
setTimeout(function(){myClsr();},500);
// example output:
// nonClsr: 178
// Closure: 198
// nonClsr: 700
// Closure: 198
```
In example.js, there are 2 function declarations; one of them, `myNonClosure()`, is a function that does not implement closure, and the other, `myClosure()`, does implement closure with the nested anonymous function expression that it returns.

`myNon` is assigned a reference to the non-closure function, so anytime `myNon()` is called results in a new date value, a call to `.getMilliseconds()` on that new date value, and, therefore, a _different_ number of milliseconds each time (as shown in the commented out example output).

`myClsr` is assigned a reference to the returned anonymous function expression of `myClosure()`, so anytime `myClsr()` is called results in calling `.getMilliseconds()` on the single date value that was created when `myClosure()` was called and, therefore, the _same_ number of milliseconds each time (as shown in the commented out example output).

#### exampleInteresting.js
```javascript
var foo = (function(){
    var o = { bar: "bar"};
    return { obj: o};                           // Implementation 1: returns an object that provides access to `o`; NOT closure
    // return function(){return o;}             // Implementation 2: returns a function that provides access to `o`; IS closure
    // return { obj: function(){return o;}};    // Implementation 3: returns an object with function property that provides access to `o`; IS closure
})();

console.log(foo.obj.bar);           // Implementation 1
// console.log(foo().bar);          // Implementation 2
// console.log(foo.obj().bar);      // Implementation 3
```
Technically speaking, implementation 1 within exampleInteresting.js is not an example of closure. Closure refers specifically to functions (not objects) that remember their original context. The Advanced JavaScript course by Kyle Simpson (linked to at the top) includes the code with implementation 1 and discusses this.

Implementation 2 is an example of closure. Returning the function instead of the object implements closure. To be clear, we could still return an object and be implementing closure if that returned object contained a function property that returned the private variable `o`, and this is precisely what is done in implementation 3.

Each of the 3 implementations accomplish the same thing: expose controlled access to the private variable `var o` via the returned object (whether or not that object is specifically a function); whether the conduit is a nested object or a nested function could seem to be an irrelevant implementation detail.

#### Why Use Closure?

While you could technically achieve the same result with a regular object rather than a function, the closure syntax (using a nested function) is a pattern that takes advantage of the design of the JavaScript language. Function objects are first-class objects in JavaScript and have the advantage of being callable. This can give closures an edge of versatility, and even readability, over regular objects in JavaScript.