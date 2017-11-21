## Helpful Links
["Block Scope in ES6"](https://app.pluralsight.com/player?course=advanced-javascript&author=kyle-simpson&name=advanced-javascript-m2&clip=9&mode=live)
* This link is to a video within an awesome Pluralsight course "Advanced JavaScript" by Kyle Simpson
* Most of the code within the example code snippets come from the above linked Pluralsight course

[[Keyword this]]
* This is a link to another page within this github wiki

---

## Lexical Scope
Lexical scope is determined at compile time (whereas dynamic scope is determined at run time). JavaScript's lexical scope gives functions their own private scope. Non-function code blocks (loops, if statements, switch statements) do not get their own private scope.

#### exampleLex.js
```javascript
// Example 1
// for loops do not have their own private scope
for(var i = 0; i < 3; i++){
    console.log(i);         // prints 0, 1, 2
}
console.log(i);             // prints 3

// Example 2
// switch statements do not have their own private scope
var n = 5;
switch(n){
    case 5:
        var woot = "woot"
        console.log(woot);  // prints woot
        break;
    default:
        break;
}
console.log(woot);          // prints woot

// Example 3
// function contains baz within its lexical private scope
function foo (bar){
    if(bar){                // if statements do not have their own private scope
        var baz = bar;
    }
    console.log(baz);       // prints "bar"
}
foo("bar");
console.log(baz);           // Error: baz is not defined
```

## Keyword `let` and Block Scope
The `let` keyword hijacks the scope of whatever block `{}` it appears in to be a private scope for the variable declared with `let`.

#### exampleLet.js
```javascript
// Example 1
// `let i` hijacks for loop as a private scope for i
for(let i = 0; i < 3; i++){
    console.log(i);         // prints 0, 1, 2
}
console.log(i);             // Error: i is not defined

// Example 2
// `let woot` hijacks switch as a private scope for woot
var n = 5;
switch(n){
    case 5:
        let woot = "woot"
        console.log(woot);  // prints woot
        break;
    default:
        break;
}
console.log(woot);          // Error: i is not defined


// Example 3
// function contains baz within its lexical private scope
function foo (bar){
    if(bar){                // `let baz` hijacks if statement as a private scope for baz
        let baz = bar;
    }
    console.log(baz);       // Error baz is not defined
}
foo("bar");
console.log(baz);           // Error: baz is not defined (same as in exampleLex.js)
```