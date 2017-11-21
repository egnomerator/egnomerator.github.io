## Helpful Links
["Prototypes Explained, Part 1"](https://app.pluralsight.com/player?course=advanced-javascript&author=kyle-simpson&name=advanced-javascript-m4&clip=1)
* This link is to a video within a Pluralsight course "Advanced JavaScript" by Kyle Simpson

[ES 2016: `__proto__`](https://www.ecma-international.org/ecma-262/7.0/index.html#sec-object.prototype.__proto__)

[ES 2016: `getPrototypeOf()`](https://www.ecma-international.org/ecma-262/7.0/index.html#sec-object.getprototypeof)

[ES 2016: `[[Prototype]]`](https://www.ecma-international.org/ecma-262/7.0/index.html#sec-ordinary-object-internal-methods-and-internal-slots)

[[Objects and Prototypes]]
* This is a link to another page within this github wiki which uses the `getObjectInfo()` function discussed in this wiki page

[[Prototype Chain]]
* This is a link to another page within this github wiki

---

##### This wiki page is only intended to cover a small number of built-in properties as ancillary information to other wiki pages here.

### "Internal Prototype Linkage" \[\[Prototype\]\]
What is this phrase "internal prototype linkage" used throughout this page?

[From ES 2016](https://www.ecma-international.org/ecma-262/7.0/index.html#sec-ordinary-object-internal-methods-and-internal-slots)
> All ordinary objects have an internal slot called \[\[Prototype\]\]. The value of this internal slot is either null or an object and is used for implementing inheritance

* This is a link between one object and the prototype object it's linked to
* This link cannot be accessed directly which is why it is called internal
  * There are built-in publicly exposed accessors for these internal linkages
* When an internal prototype linkage is returned, this linkage automatically returns the linked prototype object (you are never actually able to get the internal property \[\[Prototype\]\])
* This can produce a null value, and this happens only when getting the internal prototype linkage of the final object in the prototype chain (which is Object.prototype: `Object.getPrototypeOf(Object.prototype)` returns null)

### Some Built-in JavaScript Properties
(it's helpful to reference the diagram and related explanations [[here|Prototype Chain]]):

* `.prototype`: a built-in property of functions which points to the calling function's corresponding prototype object
  * Every function has its own corresponding prototype object
  * Every function can be used as a constructor to create a brand new object linked to that function's prototype object
  * Calling `.prototype` on a non-function object will result in `undefined`

* `Object.getPrototypeOf([object])`: returns the passed-in object's internal prototype linkage

* `[object].__proto__`: alternative to `Object.getPrototypeOf([object])`
  * `.__proto__` is a getter/setter function of the root JavaScript prototype object, Object.prototype; when `.__proto__` is called on an object, this causes a traversal up the prototype chain to Object.prototype (because `.__proto__` is a property that exists only on Object.prototype) where `Object.prototype.__proto__` is called which returns the internal prototype linkage of the object that originally made a call to `.__proto__`
  * E.g. `o.__proto__` causes the following behavior:
    * `.__proto__` is discovered not to exist on `o`
    * Traversal up prototype chain to `Object.prototype`
    * `.__proto__` exists on `Object.prototype` so `Object.prototype.__proto__` is called
    * Return internal linkage of `o`
  * The use of `.__proto__` is generally discouraged. Although it is supported in recent versions of all browsers and was even standardized in ES6, it was added in ES6 as a deprecated feature to support legacy code 

* `.constructor`: prototype objects have a `.constructor` property which points to their associated function object. Calling `.constructor` on a non-prototype object causes traversal of the prototype chain to the calling-object's prototype object on which `.constructor` is called returning the associated function object

* `Function.prototype`:
  * This is a strange outlier in the language (see [ES 2016 section 19.2.3](https://www.ecma-international.org/ecma-262/7.0/index.html#sec-properties-of-the-function-prototype-object))
    * A couple key quotes from this documentation:
      >NOTE The Function prototype object is specified to be a function object to ensure compatibility with ECMAScript code that was created prior to the ECMAScript 2015 specification.

      >The Function prototype object does not have a prototype property.
  * This is technically an exception to the "Every function ..." blanket statement bullet points under `.prototype` above, but practically and functionally speaking, this `Function.prototype` object is the prototype object of `Function` (even though it is technically a function object)