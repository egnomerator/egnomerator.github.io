## Helpful Links
["Scope and the JavaScript Compiler"](https://app.pluralsight.com/player?course=advanced-javascript&author=kyle-simpson&name=advanced-javascript-m2&clip=0&mode=live)
* This link is to a video within an awesome Pluralsight course "Advanced JavaScript" by Kyle Simpson
* Most of the content of this wiki page including the example code comes from the above linked Pluralsight course
* The linked video is just the first of many consecutive videos that cover a number of topics related to scope in JavaScript

[](http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/#a-word-on-hoisting)
* This article is a clear concise explanation of 

---

With the JavaScript compiler, we don't distribute compiled code; we distribute source code. The compiler compiles the source code every time the code is ran.

### Multiple Code Passes
* The compiler makes multiple passes over the code before the pass where it actually executes the code
  * Initial pass to compile
  * _Another pass to scope variables and functions_
    * On this pass the compiler looks for declarations and creates and attaches these declared variables and functions to the scope they are declared in
  * ... Some number of passes for various purposes ...
  * _A final pass to execute the code (this includes assigning values to variables)_

#### Why Care?
* _We only care about two particular passes the compiler makes_ (for the purposes of this wiki page)
  1. The pass where the compiler scopes declarations (we'll call it "pass one")
  1. The pass where the compiler executes code which includes variable assignments (we'll call it "pass two")
  * (using "pass one" and "pass two" to emphasize that the scope pass does happen before the execution pass)
* The above outline of code passes is high-level and grossly glosses over details, barely scratching the surface of what the compiler does
* The point of the above outline of code passes is to emphasize that _variable and function declaration scoping occurs at a completely separate time in the compiler process from when variable assignments and execution occur_

#### example1.js
```javascript
 1. var foo = "bar";
 2. 
 3. function bar(){
 4.     var foo = "baz";
 5. }
 6. 
 7. function baz(foo){
 8.     foo = "bam";
 9.     bam = "yay";
10. }
```

#### Pass One: Declarations
The compiler's first pass over the code in example1.js _basically_ looks like this:
```
Line 1: a variable `foo` is declared; registering variable `foo` to the current scope (global scope)
Line 3: a function `bar` is declared; registering function `bar` to the current scope (global scope)
Line 4: a variable `foo` is declared; registering variable `foo` to the current scope (scope of `bar`)
Line 7: a function `baz` is declared; registering function `baz` to the current scope (global scope)
Line 7: a variable `foo` is declared; registering variable `foo` to the current scope (scope of `baz`)
    // (function parameters are considered variable declarations)
```
Let's _pretend_ that on Line 8, the variable `foo` had a `var` in front of it. What would the compiler do?
```
Line 8: a variable `foo` is declared; `foo` already exists in the scope of `baz`; continue
```
Why bother with this hypothetical? Again, this is all high-level and glosses over details, but here is the point:

_Once a variable has been declared within a scope, the compiler will ignore any future declarations of that variable in that scope._

#### Pass Two: Execute
The compiler's second pass over the code in example1.js _basically_ looks like this:
```
Line 1: global scope has LHS `foo`; assigning RHS value "bar" to LHS `foo`
    // (END OF PASS TWO: see note after this code snippet)
Line 4: scope of `bar` has LHS `foo`; assigning RHS value "baz" to LHS `foo`
Line 8: scope of `baz` has LHS `foo`; assigning RHS value "bam" to LHS `foo`
Line 9: scope of `baz` has no LHS `bam`; global scope creates LHS `bam`; assigning RHS value "yay" to LHS 'global.bam`
    // (global object just created LHS `bam` "on-demand"; strict mode prevents this behavior and throws an error)
```
_Note:_ For Lines 4, 8, 9, for the sake of discussion in this example1.js demo, we are _pretending_ that, elsewhere in the code, the compiler reached code calling to these functions which is why the compiler is processing these lines. Technically, Line 1 is all that the compiler would process on this pass, and it would skip all the rest of the code, because the rest of the code consists of function declarations. So, on this pass, the compiler would skip these declarations, and find itself at the end of example1.js. (the compiler cares about declarations _only_ on the first pass)

#### Undeclared Variables
There is some odd behavior in Line 9. The variable `bam` is an undeclared variable. Undeclared variables are those that are not declared with the `var` keyword; this prevents the compiler from scoping it during pass one; which in turn causes this situation during pass two: the compiler traverses the scope chain not finding a scope that is aware of the variable `bam`; then the global scope creates `bam` on-the-fly and passes a global reference to `bam` back down the scope chain. Undeclared variables are always global and never exist until/unless they are assigned to.

The behavior in Line 9 changes when in strict mode (as the comment indicates). Strict mode prevents assigning to undeclared variables by throwing an error at execution time. The error that would be thrown for Line 9 is "bam is not defined"

#### Assignments (LHS/RHS)

This is an assignment (in JavaScript): LHS = RHS
* LHS (Left-Hand Side reference); RHS (Right-Hand Side reference)
* LHS/RHS _of what?_ ... of an assignment (e.g. equals operator `=`).

Think in terms of _source_ and _target_, because the equals operator assignment is not the only assignment syntax.
* The LHS is the _target_ and the RHS is the _source_

#### example2.js
```javascript
 1. var foo = "bar";
 2. 
 3. function bar(){
 4.     var foo = "baz";
 5. 
 6.     function baz(foo){
 7.         foo = "bam";
 8.         bam = "yay";
 9.     }
10.     baz();
11. }
12. 
13. bar();
14. foo;
15. bam;
16. baz();
```

#### Pass One: Declarations
The compiler's first pass over the code in example2.js _basically_ looks like this:
```
Line 1: a variable `foo` is declared; registering variable `foo` to the current scope (global scope)
Line 3: a function `bar` is declared; registering function `bar` to the current scope (global scope)
Line 4: a variable `foo` is declared; registering variable `foo` to the current scope (scope of `bar`)
Line 6: a function `baz` is declared; registering function `baz` to the current scope (scope of `bar`)
Line 6: a variable `foo` is declared; registering variable `foo` to the current scope (scope of `baz`)
    // (function parameters are considered variable declarations)
```
There are quite a few more lines of code past Line 6 in example2.js, but the `foo` parameter in function `baz` is the last declaration. And, declarations are all that the compiler cares about on this pass.

#### Pass Two: Execute
The compiler's second pass over the code in example2.js _basically_ looks like this:
```
Line  1: global scope has LHS `foo`; assigning RHS value "bar" to LHS `foo`
Line 13: global scope has RHS `bar`; global scope provides function object `bar`; execute function `bar`
Line  4: scope of `bar` has LHS `foo`; assigning RHS value "baz" to LHS `foo`
Line 10: scope of `bar` has RHS `baz`; scope of `bar` provides function object `baz`; execute function `baz`
Line  7: scope of `baz` has LHS `foo`; assigning RHS value "bam" to LHS `foo`
Line  8: scope of `baz` has no LHS `bam`; global scope creates LHS `bam`; assigning RHS value "yay" to LHS 'global.bam`
    // (global object just created LHS `bam` "on-demand"; strict mode prevents this behavior and throws an error)
Line 14: global scope has RHS `foo`; global scope provides variable `foo`
Line 15: global scope has RHS `bam`; global scope provides variable `bam`
Line 16: global scope has no RHS `baz`; error, "baz is not defined"
```
#### Walk Through
* Line  1: already familiar with this
* Line 13:
  * We skip to Line 13, because the code between Lines 1 and 13 is a function declaration
    * The compiler does not care about declarations on this pass, because they were already handled on pass one
  * `bar` is an RHS reference to function `bar` within the global scope, so global scope provides that object
  * Then, the parentheses `()` cause the actual execution of that provided function object
* Line 4: already familiar with this
  * The function call on Line 13 brings us to Line 4, the first line to execute within function `bar`
* Line 10:
  * Basically the same situation as Line 13;
    * We skipped to Line 10 (passed the function declaration)
    * `baz` is an RHS reference to function `baz` within the scope of `bar`, so the scope of `bar` provides that object
    * Then, the parentheses `()` cause the actual execution of that provided function object
* Line 7: already familiar with this
  * The function call on Line 10 brings us to Line 7, the first line to execute within function `baz`
  * (we skipped the foo parameter, because no argument was passed for that parameter)
* Line 8: already familiar with this (the undeclared variable assignment)
* Line 14:
  * Works just like Line 13, minus the parentheses `()`
  * Since there are no parentheses `()`, we just move to the next line after the global scope provides the value
* Line 15:
  * Works just like Line 14
  * As already pointed out, as an undeclared variable, `bam` is an object that the global scope provides
* Line 16:
  * This line wants to work like Line 13, but it throws an error "baz is not defined", because `baz` is not in the global scope

### Function Declarations and Function Expressions

_Where are the function declaration(s) and where are the function expression(s) in this example?_

#### example3.js

```javascript
 1. var foo = function bar(){
 2.     var foo = "baz";
 3. 
 4.     function baz(foo){
 5.         foo = bar;
 6.         foo;    // function ...
 7.     }
 8.     baz();
 9. }
10. 
11. foo();
12. bar();          // Error
```

Function Declaration: Line 4 `function baz(foo){...}`

Function Expression: Line 1 `function bar(){...}`

_What Makes What What??_

To qualify as a function declaration, the keyword `function` must be the first word in the statement.
* On Line 1, `function bar(){...}` comes after a variable declaration; this function is assigned to the variable `foo`; so, this is a named function expression
* On Line 4, `function` is the first word in that statement; it is a function declaration

#### Anonymous Function Expressions vs. Named Function Expressions:
On Line 1, lets say that 

`var foo = function bar(){...}` were instead written as 

`var foo = function (){...}`

This `function (){...}` is an anonymous function expression.

With a named function expression, the name of the function is within its own scope--the name of the function expression is only accessible within the function itself (this is why Line 12 of example3.js produces an error; it attempts to call function `bar` but the global scope has no such identifier).

_Advantages of Named Function Expressions Over Anonymous Function Expressions_

Named Function Expressions:
* Allow recursion
  * An anonymous function expression has no way to reference itself and therefore cannot recurse
* Can help in debugging
  * The names of the named function expressions will show up on the stack trace
  * Anonymous function expressions all look the same in a debugger
* Self-documenting code and increased readability
  * It's always helpful to name a function so that it is easier to come back to and immediately recognize that function's purpose


### Hoisting

#### example4.js
```javascript
foo();

function foo(){
    console.log("bar");
}
```
Hoisting is the term used to refer to a result that naturally occurs from the compiler's process. Since the compiler handles the declarations during the first pass and executes code on the second pass, when `foo()` is called on the first line, it successfully executes the function foo declared below it.

The term Hoisting is used because it seems almost as though the function declaration was hoisted up to the top so that it could then be called successfully--this is not what happens, but that is why the term is used.

### Gotcha's

There are 2 key points about the compiler's process that have been ignored so far:
1. Function Declarations before Variables Declarations
    * As the compiler takes its first pass to handle declarations, it handles all function declarations and then handles all variable declarations
    * In all the examples above showing what the compiler does for each line, all the function declaration lines should really be at the top before all the variable declaration lines
    * E.g. the first pass for example1.js should look like this
    ```
    Line 3: a function `bar` is declared; registering function `bar` to the current scope (global scope)
    Line 7: a function `baz` is declared; registering function `baz` to the current scope (global scope)
    Line 1: a variable `foo` is declared; registering variable `foo` to the current scope (global scope)
    Line 4: a variable `foo` is declared; registering variable `foo` to the current scope (scope of `bar`)
    Line 7: a variable `foo` is declared; registering variable `foo` to the current scope (scope of `baz`)
    ```
    * And, a further point of note regarding this sequence is that JIT compilation is used in JS engines which means that a function declaration is not actually compiled until it is needed for execution
      * If a function is declared on line 1 and isn't called until line 100, the compiler will only register the name of that function to the global object; it will not compile the function body until it needs to at the time it is called on line 100
1. Duplicate Declarations
    * Of course this situation should be avoided, but here is how JavaScript handles situations where both a variable and a function have been declared with the same name (or 2 variables, or 2 functions):
      * If a function is declared with a name already used, then the reference is updated to point to this function
      * If a variable is declared with a name already used, then the variable declaration is ignored
        * The assignment operation will be executed as usual