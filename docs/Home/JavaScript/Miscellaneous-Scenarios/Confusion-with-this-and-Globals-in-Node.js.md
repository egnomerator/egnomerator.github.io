## Helpful Links
* Node.js
  * [Node.js Module](https://nodejs.org/api/modules.html#modules_modules)
  * [Top-Level Scope: Browsers vs. Node.js](https://nodejs.org/api/globals.html#globals_global)
    * This documentation explains the difference between a node module's top-level scope versus the top-level scope in a browser
  * [Node.js Module Wrapper](https://nodejs.org/api/modules.html#modules_the_module_wrapper)
    * This documentation explains the module wrapper, the module object, and the module.exports object
  * [The Node REPL](https://nodejs.org/api/repl.html#repl_repl)
* [[Activation Object and Variable Object]]
  * This is a link to another page within this github wiki
  * It might be helpful to have it open in another tab
* [[Keyword this]]
  * This is a link to another page within this github wiki
  * It might be helpful to have it open in another tab

---

## The Scenario
#### Given a sample of code demonstrating general behavior of the `this` keyword in JavaScript, unexpected output was produced

_Note:_ It might be helpful to have read the other page [[Keyword this]]
  * The code in example.js is copied from Example 1 in [[Keyword this]] (added line numbers)

### example.js

```javascript
 1  // illustrates default binding and implicit binding
 2  
 3  function foo(){
 4      console.log(this.bar);
 5  }
 6  
 7  var bar = "bar";
 8  var o2 = { bar: "bar2", foo: foo };
 9  var o3 = { bar: "bar3", foo: foo };
10  
11  foo();    // "bar"  (undefined if in strict mode)   <-- this is default binding
12  o2.foo(); // "bar2"                                 <-- this is implicit binding
13  o3.foo(); // "bar3"                                 <-- this is implicit binding
```

## The Problem
* Per line 11 comment, output was expected to be "bar". However, when executed in Node.js, output was `undefined`

### The Quick Explanation <sub><sup>(certain background knowledge in JavaScript and Node.js needed)</sup></sub>
#### Facts
* The default binding mechanism IS applying as expected
  * Therefore, when the code within `foo()` is executing due to the call to `foo()` on line 11, `this` is pointing to the global object
* On line 7, where `bar` is declared, `bar` becomes attached to the Node.js module wrapper's `Variable Object` rather than the global object (see [[Activation Object and Variable Object]])
  * This is due to the way Node.js handles globals in modules (see [Node.js Module Wrapper](https://nodejs.org/api/modules.html#modules_the_module_wrapper))

#### Conclusion

* This means that when `foo()` (called on line 11) tries to access `this.bar`, the global object does not contain a property `bar`, because the declaration of `bar` attached `bar` to the module wrapper's variable object

_Note_: In example.js, if `bar` were declared without the `var` keyword, `bar` would be attached to the global object resulting in the expected output

### The Long Explanation

#### As the call site, line 11 from the example code is the main point of focus
* The comment on this line indicates that the call to `foo()` will output "bar"
* The comment on this line also indicates that the output would be `undefined` if in `strict mode`

#### Expected Output of `foo()` (based on line 11 comment): "bar" (or `undefined` if in `strict mode`)
* The line 11 comment is not arbitrary; it is an expectation based on the behavior of the `this` keyword mechanism in JavaScript
* Specifically, the expectation is based on the default binding mechanism--the 4<sup>th</sup> rule covered in [[Keyword this]] which says,

  > Default Binding Rule
  > * If none of the above rules apply, then `this` will point to the global object by default
  > * UNLESS in strict mode which will cause `this` to be undefined

* _So, based on the above information, we expect line 11 to output "bar"_
  * _... and that IS the output IF the code is ran in a browser (or if the code is ran in the [Node REPL](https://nodejs.org/api/repl.html#repl_repl)) ..._

### Where the Confusion Began

_... well, if we run the example file in Node.js (by calling `node example.js` from a command shell or by requiring example.js in another file we run), line 11 produces `undefined` as the output._

What is going on?
* The default binding rule does apply in example.js (if this is not clear, read [[Keyword this]])
* At this point, it seems that Node.js might default to `strict mode` (which would explain the output of `undefined`) ... is this true? No, Node.js does not default to `strict mode`.

What we know at this point:
* We know that the default binding rule applies
* We know that we are not in `strict mode`
* So, we know that when `foo()` is called from line 11, and the code within `foo()` is being executed, `this` refers to the global object

... And yet we are seeing the output of `undefined`

* So, this must mean that, contrary to expectations, `bar` is not attached to the global object

#### Key:
* Question: since, `bar` is not attached to the global object like we expected, **what is `bar` attached to?**
* Answer: `bar` is attached to the Variable object of the previous execution context

#### Key Core Terms

* **Variable Object:** see [[Activation Object and Variable Object]]
  * Every execution context has an internal container object, a Variable object, which holds the variables local to that execution context
* **Activation Object:** see [[Activation Object and Variable Object]]
  * Whenever a function is called, an Activation object is created as the function's execution context; this object is used as the Variable object for that function's execution context
* **Node.js Module:** see [Node.js Module](https://nodejs.org/api/modules.html#modules_modules)
  * Basically, a Node.js module is a file containing code. (e.g. our example.js file)
  * (it's not _just_ a file; there's more to this in the documentation)
* **Node.js Module Wrapper:** see [Node.js Module Wrapper](https://nodejs.org/api/modules.html#modules_the_module_wrapper)
  * Every module's code is automatically wrapped in a function called a module wrapper for scoping purposes

    _the Node.js module wrapper_
    ```
    (function (exports, require, module, __filename, __dirname) {
    // Your module code actually lives in here
    });
    ```

* **Top-Level Scope:** see [Top-Level Scope: Browsers vs. Node.js](https://nodejs.org/api/globals.html#globals_global)
  * In short, top-level scope is the outer-most scope (scopes are nested) in the JavaScript scope chain
  * Node.js's top-level scope is the module (i.e. the scope of the module wrapper function) rather than the global object

#### Explaining These Terms

In browsers (and in the [Node REPL](https://nodejs.org/api/repl.html#repl_repl)), the top-level scope is the global scope.

But in a Node.js module, the top-level scope is the module, thanks to the Node.js module wrapper function. What does that mean ("the top-level scope is the module")?

Remember, every function in JavaScript has its own execution context (i.e. its Activation object) and this Activation object is used as the Variable object for that function, the Node.js module wrapper function is no exception. When Node.js loads a module (e.g. when you execute a file from a command shell calling `node example.js`) Node.js wraps the module in a module wrapper function. When that module wrapper function is called, the Activation object is created. Then, that Activation object is used as the Variable object for that module wrapper function's execution context; so, the arguments passed to the module wrapper function including the loaded module (i.e. your file) are attached to this Variable object.

**To summarize,** the module wrapper's Variable object (rather than the global object) is the top-level scope--this is what is meant by the Node.js documentation ([Node.js Module Wrapper](https://nodejs.org/api/modules.html#modules_the_module_wrapper)) that says,

> ... It [the module wrapper] keeps top-level variables (defined with `var`, `const` or `let`) scoped to the module rather than the global object.

#### Applying These Concepts to the Example Code

So, after `foo()` is called on line 11, and the code within `foo()` is being executed, `this` points to the global object as expected (per the JavaScript default binding rule); but, the global object does not contain `bar`. In fact, `bar` is contained by the Variable object of the module wrapper. So, the call to `this.bar` in `foo()` returns `undefined`.

## Conclusion

Since, the top-level scope in a Node.js module is the module itself rather than the global scope, variables declared in the top-level scope of a Node.js module will not be accessible via the global object. This explains why, when we run example.js in Node.js, the output for `this.bar` is `undefined`.

_Note_: As mentioned in the "Quick Explanation" above, in example.js, if `bar` were declared without the `var` keyword, `bar` would be attached to the global object resulting in the expected output