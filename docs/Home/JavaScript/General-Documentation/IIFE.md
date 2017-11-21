## Helpful Links
["IIFE Pattern"](https://app.pluralsight.com/player?course=advanced-javascript&author=kyle-simpson&name=advanced-javascript-m2&clip=7)
* This link is to a video within an awesome Pluralsight course "Advanced JavaScript" by Kyle Simpson

[Ben Alman on IIFE](http://benalman.com/news/2010/11/immediately-invoked-function-expression/)
* This link is to an article by Ben Alman who pioneered the name IIFE for this pattern
* There are some great links in this article as well

[[JavaScript Compiler]]
* This is a link to another page within this github wiki

---

## IIFE: Immediately-Invoked Function Expression
Syntax Options:
```javascript
// The two most common syntactical approaches in no particular order:

(function(){ /* code */ })();    // the executing parentheses are placed outside the grouping parentheses

(function(){ /* code */ }());    // the executing parentheses are placed within the grouping parentheses
```
* The IIFE does not have to be an anonymous function expression; in fact, as with all function expressions, naming them is best practice.
* There are other syntax options, and they are included in the [Ben Alman on IIFE](http://benalman.com/news/2010/11/immediately-invoked-function-expression/) article
  * The 2 syntax options above, seem to be most common
  * Also, the other syntax options are considered by some to be less preferred for different reasons such as readability and even potentially unwanted side effects depending on which alternative syntax is used

#### How the IIFE Pattern Works

_What does the IIFE pattern do?_
1. It wraps a function expression in parentheses
    > <strong>`(`</strong><sub><sup>`function foo(){ /* code */ }`</sub></sup><strong>`)`</strong><sub><sup>`()`</sub></sup>
    * This prevents the compiler from treating it as a function declaration
    * See [[JavaScript Compiler]]
1. It executes that function expression immediately with the set of parentheses that follow the expression
    > <sub><sup>`(function foo(){ /* code */ })`</sub></sup><strong>`()`</strong>
1. It allows passing arguments into the scope of the IIFE
    > <sub><sup>`(function foo(`</sub></sup><strong>`privateFoo`</strong><sub><sup>`){ /* code */ })`</sub></sup><sub><sup>`(`</sub></sup><strong>`foo`</strong><sub><sup>`)`</sub></sup>

Let's look at some example code that _tries_ (and fails) to achieve what the IIFE pattern achieves. This may help clarify how the IIFE pattern works.
```javascript
// Example 1
function foo(){ /* code */ }(); // SyntaxError: Unexpected token )

// Example 2
function foo(){ /* code */ }( 1 );

// Example 3
function foo(){ /* code */ }
( 1 );
```
_Examples Explained_ (see [[JavaScript Compiler]]):
* Example 1
  * This _looks_ like an attempt to execute a function. But, the compiler sees a function declaration followed by a grouping operator `()`
  * The compiler _expects_ an expression in the grouping operator, which is why the syntax error occurs
* Example 2
  * This _looks_ like an attempt to execute a function, passing in the expression `1` as an argument
  * But, the compiler sees a function declaration followed by a completely unrelated expression
    * Example 2 works exactly the same as Example 3, which may be a little more clear visually
  * Example 2 (and Example 3) won't throw an error, because the compiler's expectation of an expression in the grouping operator `(1)` is fulfilled; but, it does not accomplish what was intended

#### Why Use the IIFE Pattern?

It is a powerful scoping mechanism:
* Hide code from the global scope
* Maintain private internal state via closure
* Encapsulate and selectively expose an api
  * E.g. the module pattern, shown in a code example below
* Avoid polluting global scope with unnecessary, limited-use references

#### Effective Uses of IIFE

_Maintaining State with Closures: Example 1_

```javascript
var foo = function fooFn(msg){console.log(msg)};
foo("  Pre-IIFE foo");

(function bar(cpy){
    cpy("   In-IIFE cpy");        // "imported" foo
    foo("   In-IIFE foo");        //     global foo

    setTimeout(function p3(){     // ... pause 3 seconds ...

        cpy("3s In-IIFE cpy");    // "imported" foo - NOT changed
        foo("3s In-IIFE foo");    //     global foo -  IS changed (prints "3s In-IIFE foo |CHANGED|")
    },3000)

})(foo);

foo("After-IIFE foo");

foo = function fooFn2(msg){console.log(msg + " |CHANGED|")};
foo("   changed foo");

console.log();
console.log("pause 3 seconds ...");
console.log();

// RESULT:
// 
//   Pre-IIFE foo
//    In-IIFE cpy
//    In-IIFE foo
// After-IIFE foo
//    changed foo |CHANGED|
// 
// pause 3 seconds ...
// 
// 3s In-IIFE cpy
// 3s In-IIFE foo |CHANGED|
```
This above code example is intended to demonstrate linearly that the IIFE _protected_ its copy (`cpy`) of `foo` from the change that occurred outside the IIFE. After the IIFE is executed (which kicks off the 3 second timeout) `foo` is changed to point to a new function that appends "|CHANGED|" to the passed in `msg` argument before printing it out. This change to `foo` occurs well before the 3 second timeout completes. So by the time the IIFE's `cpy` and `foo` are executed after that timeout, `foo` has been changed--yet the `cpy` of `foo` is unaffected.

_Maintaining State with Closures: Example 2_

Perhaps a better way to demonstrate the value of this scoping mechanism is the below example where the IIFE provides the same protection within the context of a loop.
```javascript
// Loop finishes (way) before 3 second setTimeout elapses
// After the 3 seconds, console.log(i) is called 5 times, but i already equals 6
// RESULT: prints five 6s
for(var i = 1; i < 6; i++){
    setTimeout(function(){
        console.log(i);
    }, 3000);
}

// Using IIFE Wrapper
// During each loop, the current "state" of i is imported into and protected within the IIFE scope
// RESULT: prints out 1-5
for(var i = 1; i < 6; i++){
    (function keep(iState){
        setTimeout(function(){
            console.log(iState);
        }, 3000);
    })(i);
}
```

_The Module Pattern_

_(This code example is straight from [Ben Alman on IIFE](http://benalman.com/news/2010/11/immediately-invoked-function-expression/))_
```javascript
// Create an anonymous function expression that gets invoked immediately,
// and assign its *return value* to a variable. This approach "cuts out the
// middleman" of the named `makeWhatever` function reference.
//
// As explained in the above "important note," even though parens are not
// required around this function expression, they should still be used as a
// matter of convention to help clarify that the variable is being set to
// the function's *result* and not the function itself.

var counter = (function(){
    var i = 0;

    return {
        get: function(){
            return i;
        },
        set: function( val ){
            i = val;
        },
        increment: function() {
            return ++i;
        }
    };
}());

// `counter` is an object with properties, which in this case happen to be
// methods.

counter.get(); // 0
counter.set( 3 );
counter.increment(); // 4
counter.increment(); // 5

counter.i; // undefined (`i` is not a property of the returned object)
i; // ReferenceError: i is not defined (it only exists inside the closure)
```