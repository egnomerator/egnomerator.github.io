## Helpful Links
["this Keyword"](https://app.pluralsight.com/player?course=advanced-javascript&author=kyle-simpson&name=advanced-javascript-m2&clip=16&mode=live)
* This link is to a video within a Pluralsight course "Advanced JavaScript" by Kyle Simpson
* The 4 [rules](#theRules) and their 4 corresponding questions, and most of the code within the example code snippets come from the above linked Pluralsight course
* The linked video is the first of 5 consecutive videos that very clearly and thoroughly cover the behavior of `this`

[[Confusion with this and Globals in Node.js]]
* This is a link to another page within this github wiki

---

# Nuances of the Keyword `this` in JavaScript

**Caveat**: results of the below code examples will differ when ran in Node.js vs. browsers, see [[Confusion with this and Globals in Node.js]] for details

#### Dynamic Scope
The `this` keyword in JavaScript is an example of dynamic scope: the scope of `this` is determined at run time (whereas lexical scope is determined at compile time). Determining what the keyword `this` refers to can be confusing sometimes. But, thankfully, there are reliable, clear-cut `this` binding rules that can be very helpful in determining what `this` refers to in any execution context.

## <a name="theRules"></a>The Rules

### Call Site is Key:

_How can we know what `this` points to?_
 
It's all about the **call site**, the location in the code where the function _is called_ (_not_ where the function is declared). The function's call site dictates what `this` points to while the code within the function is being executed.

_Look at the function's call site and find which of the below four rules apply._

### FOUR RULES (in order of precedence!!)

_Each of these rules references one example; but, each of these rules are demonstrated in 1 or more of the examples._

1. `new` Keyword Binding Rule
    * if the `new` keyword is used in front of a function call (shown in Example 7), `this` refers to the new object
2. Explicit Binding Rule
    * if you use `.call()` or `.apply()` at the call site (shown in Example 3), `this` refers to the passed object argument
3. Implicit Binding Rule
    * if you call the function from an object property reference (shown in Example 1), `this` refers to that property's containing object
4. Default Binding Rule
    * if none of the above rules apply (shown in Example 1), then `this` will point to the global object by default
      * UNLESS in strict mode which will cause `this` to be undefined

_If more than 1 of the above rules apply to a given call site, then the rule with highest precedence applies._

### FOUR QUESTIONS <sub><sup>(to help clarify the 4 rules)</sup></sub>

1. Was the function called with the `new` keyword?
1. Was the function called with `.call()` or `.apply()`?
1. Was the function called via an object property?
1. Did none of the above apply?

Below, are a number of code examples that demonstrate the different types of `this` keyword binding that can occur based on the `this` binding rules.

### Code Examples To Demonstrate `this` behavior in Different Execution Contexts

#### This Nuances:  Example 1
```javascript
// illustrates default binding and implicit binding

function foo(){
    console.log(this.bar);
}

var bar = "bar";
var o2 = { bar: "bar2", foo: foo };
var o3 = { bar: "bar3", foo: foo };

foo();    // "bar"  (undefined if in strict mode)   <-- this is default binding
o2.foo(); // "bar2"                                 <-- this is implicit binding
o3.foo(); // "bar3"                                 <-- this is implicit binding
```

#### This Nuances:  Example 2
```javascript
// illustrates default binding and implicit binding

var o1 = {
    bar: "bar1",
    foo: function(){
        console.log(this.bar);
    }
};
var o2 = {bar: "bar2", foo: o1.foo};

var bar = "bar3";
var foo = o1.foo;

o1.foo();   // "bar1"   <-- this is implicit binding
o2.foo();   // "bar2"   <-- this is implicit binding
foo();      // "bar3"   <-- this is default binding
```

#### This Nuances:  Example 3
```javascript
// illustrates default binding and explicit binding

function foo(){
    console.log(this.bar);
}

var bar = "bar1";
var obj = { bar: "bar2"};

foo();          // "bar1" <-- this is default binding
foo.call(obj);  // "bar2" <-- this is explicit binding; `this` points to the passed object argument (`obj`)
```

#### This Nuances:  Example 4
```javascript
// illustrates default binding and "hard" binding (more predictable explicit binding)

function foo(){
    console.log(this.bar);
}

var obj = { bar: "bar" };
var obj2 = { bar: "bar2" };

var orig = foo;                         // hard binding consists of the
foo = function(){ orig.call(obj);};     // code in each of these 2 lines

foo();              // "bar"
foo.call(obj2);     // "bar"
```

#### Explanation on "Hard" Binding (a specific implementation of explicit binding)

The purpose of hard binding is to provide an implementation of a function that produces a more predictable result. Here in Example 4, hard binding is demonstrated to override the explicit binding `foo.call(obj2)` call's attempt to pass in a different object to produce different output--the output is unchanged due to the hard binding forcing the call to foo() to always be called passing in `obj`.

_Remember: `new` keyword binding has higher precedence than explicit binding (even hard binding!)_

#### This Nuances:  Example 5
```javascript
// illustrates "hard" binding via a little bind utility

function bind(fn,o){
    return function(){
        fn.call(o);
    };
}
function foo(){
    console.log(this.bar);
}

var obj = { bar: "bar" };
var obj2 = { bar: "bar2" };

foo = bind(foo,obj);

foo();              // "bar"
foo.call(obj2);     // "bar"
```

#### This Nuances:  Example 6
```javascript
// illustrates "hard" binding via a safer, more useful utility ("bind2" is used to differentiate from the bind in the ES5 spec version)
if(!Function.prototype.bind2){
    Function.prototype.bind2 = 
        function(o) {
            var fn = this; // the foo function
            return function(){
                return fn.apply(o,arguments);
            };
        };
}
function foo(baz){
    console.log(this.bar + " " + baz);
}
var obj = { bar: "bar" };
var obj2 = { bar : "bar2"}

foo = foo.bind2(obj);

foo("baz");             // "bar baz"
foo.call(obj2, "baz");  // "bar baz"
```

#### This Nuances:  Example 7
```javascript
// illustrates the affect of the `new` keyword on `this` binding (`new` causes `this` to be bound to a new object)

function foo(){
    this.baz = "baz";
    console.log(this.bar + " " + baz);
}

var bar = "bar";

// using the `new` keyword hijacks a function call into a constructor call
// we get "undefined" for `this.bar` because the new object has no such property
// we get "undefined" for `baz` because the new object has not yet been assigned to `baz`
var baz = new foo();    // "undefined undefined"

// at this point the new foo() object has been created and assigned to the variable `baz`
console.log(baz);       // "foo { baz: 'baz' }"
```

#### What happens when the `new` keyword is used in front of a function call?

(**4 things happen**: within these 4 items, we will refer to an example function `foo()` as the function being called with the `new` keyword in front of it--so what 4 things happen when `foo()` is called with the `new` keyword in front of it?)

1. A brand new object is created (`foo {}`)
1. The new object is linked to a different object (via the prototype chain)
1. The new object gets bound as the `this` keyword for this execution of `foo()`
1. **IF** `foo()` does not explicitly return either a function or an object, an implicit `return this` statement is inserted at the end of the code within `foo()`
    * This means that, after the existing code within `foo()` is executed, `foo()` will return `this`
    * (i.e. `foo()` is modified to return the new object implicitly)