## Helpful Links
[[Objects and Prototypes]]
* This is a link to another page within this github wiki which uses the `getObjectInfo()` function discussed in this wiki page

[[Built In JavaScript Properties]]
* This is a link to another page within this github wiki

---

#### A Simple Helper Function

In all the code examples within the [[Objects and Prototypes]] wiki page, each example will end with commented output--this output will be the result of running the below helper function on an object. The built-in JavaScript properties accessed in this function expose internal prototype linkages (and can be used to implement inheritance in JavaScript). These JavaScript properties constitute the prototype chain. The reason for printing these values is to demonstrate how JavaScript prototype linkage works and what linkage will occur depending on the approach used for creating objects. Kyle Simpson explains this linkage information very well in his [course](https://app.pluralsight.com/player?course=advanced-javascript&author=kyle-simpson&name=advanced-javascript-m4&clip=1).

_Browser developer tools typically expose this information in some fashion as a built-in browser console UI feature (e.g. traversable node structure), but these log statements can still be helpful, especially in a text-only console environment._
```javascript
function getObjectInfo(o){
    console.log("Below is \"meta info\" of this object \"o\" ...");
    console.log("o:");
    console.log(o);
    console.log("o.prototype");
    console.log(o.prototype);
    console.log("Object.getPrototypeOf(o)");
    console.log(Object.getPrototypeOf(o));
    console.log("o.constructor");
    console.log(o.constructor);
}
```