# ES6-notes
Simple ES6 notes.

## Content
* Tail Recursion
* Declarations
* Rest Parameters
* Spread Operators
* Destructuring

## Tail Recursion

### Context

**Tail Recursion:** Tail recursion is a special kind of recursion where the recursive call is the very last thing in the function. It's a function that does not do anything at all after recursing. 

When writing any recursive algorithm, you should consider the involved recursion depth because it may translate into stack space. You usually have a very good idea of that depth because any sane recursive  algorithm should be based on a well founded order (if not it is unlikely to work).

As a rule of thumb if it's O(log n) (typical value for balanced binary trees traversal for instance) recursion is very unlikely to raise any run-time problem. 

On the other hand if recursion depth is O(n) this could lead to some troubles with stack... and if it's worse than O(n), you are very likely to crash the stack, henceforth you should think again.

Some recursions can be easily removed by compilers: if the recursive call is the very last thing performed by the function before returning. These are called 'tail recursions' (or more generally tail calls, because you actually don't have to call the same function, it merely has to be the last action before returning result). These can be internally replaced by a mere goto (or loop) to the function... if the compiler/interpreter used is smart enough.

For some languages  this optimisation is mandatory (ie: most functional languages). Others may or may not provide it depending on the compiler. 

In the end if your algorithm recursion depth is O(n), and it's a tail call recursion all will be fine if compiler replace it by a loop... but the stack is likely to crash if it's not optimised away. If your algorithm is O(n) and your recursion is no proper tail call you'll have trouble wathever the compiler/language.

The nice thing is that if the compiler does not implements tail calls optimisation (TCO), it's usually easy enough to do it by hand and rewrite the algorithm to use a loop. For people used to recursion, the resulting algorithm looks harder to read and understand. It's not the direct answer to the problem any more but a transformation of it (and if you are not used to recursion, you likely didn't thought of the recusive solution anyway).

There was an hot debate on this subject in python community. Functional programmers asked for tail call, while others rejected it on the ground that the stack trace is too precious for debugging and that TCO implementation implied removing an explicit stack call and replacing it with a loop. The problem is that if we do such removal internally the function call becomes invisible in stack trace. Guido Von Rossum(python community member) decided there would be no TCO in Python  and that developpers should use loops or list comprehension instead. So avoid any O(n) recursion when programming python. O(log n) recursion for balanced trees or similar data structures is still fine TCO or no TCO.

[source for tail recursion definition](https://www.quora.com/What-is-tail-recursion-Why-is-it-so-bad)

**Tail Position:** Last instruction to fire before the return statement. There can be multiple tail positions in a function.

**Tail Call:** Call for tail recursion

``` js
  function sayHello() {
    return 'Hello'; // Tail Position, but not Tail Call
  }

  function getSalutation() {
    return sayHello(); // Tail Position and Tail Call 
  }

```

`getSalution` is tail call because when you call `sayHello`, compiler don't need to maintain the current function stack. All compiler need to care about is the next call stack i.e., `sayHello` and the previous function stack will be GC'd (garbage collected). So this way the memory consumption is not increasing linearly but remaining constant(approximately). 

**A close call looking like tail call**

``` js
  function factorial(num) {
    if (num === 1)
      return 1; // Tail Position
    else 
      return num * factorial(num - 1); // Tail Position, but not a Tail Call
  }
```
It is not tail call because the function `factorial` has to do something after the recursion as well, it need to multiply `num` with the result of `factorial(num - 1)` so in this case it is necessary to hold the function stack in memory for correct results. 

**Tail call implementation of factorial**

``` js
  function factorial(i, minFactorial):
    minFactorial = minFactorial || 1;
    if (i === 1)
      return minFactorial; // Tail Position
    else
      return factorial(i - 1, minFactorial * i); // Tail Position and Tail Call
  }
```
### Main Content

JavaSript is GC'd language unlike the low level languages like `C` which uses `malloc` and `free` for memory management. Though Javascript has GC(garbage collection), ES5 or normal js that we use doesn't have tail call optimization so the performance of recursive function is pretty bad. Compiler doesn't take constant memory for such functions, but the memory consumption is increased linearly on each recursive call so eventually we get an `RangeError: Maximum call stack size exceeded`. 

>> ES6 brings this new feature of `Tail Call Optimization` which makes tail call work in the way they were supposed to work. Tail call will make the previous stack of function GC'd and all that is left in memory is the stack for tail call method.


**Tail call but without Tail Call Optimization**
``` js
  function foo(num) {
    try {
      return foo( (num || 0) + 1 ); // Tail Call
    } catch (e) {
      return num;
    }
  }

  console.log(foo());
  // 10416 in chrome
  // 49993 in mozilla
```

**Fibbonacci Series with Tail Call in ES5(don't have TCO)**
``` js
  function fib(x, y, limit, index){
    if(arguments.length === 1){
      if(x)
        return fib(0, 1, x, 1);
      else
        return 0
    }else{
      if(index < limit)
        return fib(y, (x + y), limit, ++index);
      else
        return y;
    } 
  }
  
  console.log(fib(3));     // 2
  console.log(fib(5));     // 5
  console.log(fib(13000)); // RangeError
```


**Fibbonacci Series with Tail Call in ES6(have TCO)**
``` js
  // Same code as above
  function fib(x, y, limit, index){
    if(arguments.length === 1){
      if(x)
        return fib(0, 1, x, 1);
      else
        return 0
    }else{
      if(index < limit)
        return fib(y, (x + y), limit, ++index);
      else
        return y;
    } 
  }
  
  console.log(fib(3));     // 2
  console.log(fib(5));     // 5
  console.log(fib(13000)); // A big no. equivalent to infinity, but no RangeError!
```

### Conclusion
* Tail calls only work in strict mode
* Any method whether recursive or not, can be benifited by this strategy of tail call but at the cost of ugliness that it broughts to your code
* Tail calls are great but it removes the method from stack trace which makes code difficult for debugging.

## Declarations

### Context

**Variable Hoisting**
Pre-ES6 javascript is function scoped language and Hoisting is JavaScript's default behavior of moving all declarations to the top of the current scope (to the top of the current script or the current function). 


Variable hoisted to Global Scope Example
``` js
  function outer() {
    a = 0; // Error in 'strict' mode
    function inner() {
      b = 0; // Error in 'strict' mode
    }
    inner();
  }
  outer();
```
JS compiler thinks of the above code as this:
``` js
  var a = undefined; // hoisted to global scope
  var b = undefined; // hoisted to global scope
  function outer() {
    a = 0;
    function inner() {
      b = 0;
    }
    inner();
  }
  outer();
```
 
Varaible in local blocks hoisted
``` js
  if(true) {
    var a = 0;
  }
  function outer() {
    if(true) {
      var b = 0;
    }
  }
  outer();
```
JS compiler thinks of the above code as this:
``` js
  var a = undefined;
  if(true) {
    a = 0;
  }
  function outer() {
    var b = 0; // as es5 is function scoped
    if(true) {
      b = 0;
    }
  }
  outer();
```

Another example of variable hoisting:
``` js
  console.log(name);
  var name = "arshad";
  console.log(name);
```
JS compiler thinks of the above code as this:
``` js
  var name = undefined;
  console.log(name); // undefined
  name = "arshad";
  console.log(name); // arshad
```

**Function Hoisting**
Functions are also hoisted like variables but there is a catch. Only functions that are created by `function declaration` are hoisted like variables but functions created by `function expression` are not hoisted.


Simple example:
``` js
  isHoisted();
  
  // creating isHoisted by function declaration
  function isHoisted() {
    console.log("Yes!");
  }
```
JS compiler thinks of the above code as this:
``` js
  function isHoisted() {
    console.log("Yes!");
  }
  
  isHoisted(); // Yes!
```


Example having both function declaration and function expression:

``` js
  // Outputs: "Definition hoisted!"
  definitionHoisted();

  // TypeError: undefined is not a function
  definitionNotHoisted();
  
  // function declaration
  function definitionHoisted() {
      console.log("Definition hoisted!");
  }

  // function expression
  var definitionNotHoisted = function () {
      console.log("Definition not hoisted!");
  };
```

**Let**

`let` is another keyword to create variables like `var` but the variables created have following different properties:
* let variables can be reassigned
* let variables can't be redeclared
* let variables are supposed to reserve space in memory before their declaration so a TDZ(Temporal Dead Zone) is created which ables us to find errors when variable is used before its declaration. This feature is not implemented well in some of the browsers like Mozilla so instead of getting a `ReferenceError` on accessing let variables before their declaration, `0` or `undefined` is retrieved. 
* let introduces block scoping


Example for block scoping:
``` js
  let a = 0;
  if(true) {
    // block scoping
    let a = 1;
    let b = 2;
    console.log(a,b); // 1, 2
  }
  console.log(a); // 0
  console.log(a + b) // Error
```


Example for multiple declaration:
``` js
  let b = 0;
  let b = 1; // Error: can't declare variable twice
  
  var a = 0;
  let a = 1; // Error: can't declare variable twice
  
  var a = 0;
  var a = 1; // No Error (even in strict mode) but dont do it
```


Temporal Dead Zone Example:
``` js
  function foo() {
    // Browser should log a RefereceError here 
    // because variable is being accessed in TDZ
    console.log(a);
    let a = 1, 
    // Even this console.log statement will give error as 
    // variable 'a' can only be used after declaration line
    console.log(a); 
  }
```
In ECMAScript 6, accessing a let or const variable before its declaration (within its scope) causes a `ReferenceError`. The time span when that happens, between the creation of a variable’s binding and its declaration, is called the temporal dead zone.


**Let with Looping**
Let variables declared in loops are only loop scoped, they cant be accessed after the loop. Let also brings down the infamous closure problem that will be shown in the examples below.


Scoping in loops example:
``` js
  for (var i = 0; i < 5; i++) {
    if (i > 3) {
      var a = 'a';
    }
    console.log(i); // logs 1 - 4
  }
  
  for (let j = 0; j < 5; j++) {
    if (j > 3) {
      var b = 'b';
    }
    console.log(j); // logs 1 - 4
  }

  console.log(a); // a
  console.log(i); // 5
  console.log(b); // ReferenceError
  console.log(j); // ReferenceError
  
```

Infamous closure problem:

``` js
  var a = [];
  for(var i = 0; i < 3; i++) {
    a[i] = function() {
      return i;
    }
  }
  // [3, 3, 3] as each of anonymous function is bound to same variable
  // i that is outside of the function in global scope
  console.log(a); 
```


In ES6:
``` js
  var a = [];
  for(let i = 0; i < 3; i++) {
    a[i] = function() {
      return i;
    }
  }
  
  // [0, 1, 2] as anonymous found is bound to a seperate 
  // value on each iteration that is in for loop scope

  console.log(a); 
 ```


**CONST**
`const` keyword is used to create constant variable(a perfect example of oxymoron). Here are the few properties that they have:

* Can't be reassigned
* Respects block scopes, like let
* Can't be re-declared

``` js
  const a = 'a';
  a = 'b'; // Error: const variable cant be reassigned
```

**Block Functions**
They are used to create temporal scopes for variables.

``` js
  function foo() {
    if (true) {
      let a = 0; // temporal scope variable
    }

    { // Laydown new scope: Block Scope
      let b = 0; // another way to create temporal scope
    }
  }
```


## Rest Parameters
The rest parameter syntax allows us to represent an indefinite number of arguments as an array.

Syntax: `...args`

**Difference between rest parameter and arguments object**
* rest parameters are only the ones that haven't been given a separate name, while the arguments object contains all arguments passed to the function;
* the arguments object is not a real array, while rest parameters are Array instances, meaning methods like sort, map, forEach or pop can be applied on it directly;
* the arguments object has additional functionality specific to itself (like the callee property).


Rest parameters have been introduced to reduce the boilerplate code that was induced by the arguments
```js
  // Before rest parameters, the following could be found:
  function f(a, b){
    var args = Array.prototype.slice.call(arguments, f.length);

    // …
  }

  // to be equivalent of

  function f(a, b, ...args) {

  }
```

Rest parameters can be destructured, that means that their data can be extracted into distinct variables.
``` js
  function f(...[a, b, c]) {
    return a + b + c;
  }

  f(1)          // NaN (b and c are undefined)
  f(1, 2, 3)    // 6
  f(1, 2, 3, 4) // 6 (the fourth parameter is not destructured)
``` 

**Rules for Rest Parameters**
* One per function
* Must be the last parameter
* Can't use `arguments` in a function when using rest parameter
* No default values

**Hack to create temporary scope variables in ES5**
``` js
  if (true) {
    try {
      throw "here is your funky variable";
    } catch (e) {
      e = "my temporary scope variable";
      // code...
    }
  }
```

### Spread Operators

The spread syntax allows an expression to be expanded in places where multiple arguments (for function calls) or multiple elements (for array literals) or multiple variables  (for destructuring assignment) are expected.

**Note:** 
* Spread operators are only for iterable objects.
* Spread operators only go one level deep when used with arrays

Syntax
``` js
  myFunction(...iterableObj);
  For array literals:

  [...iterableObj, 4, 5, 6]
```
Examples:
``` js
  function myFunction(x, y, z) { }
  var args = [0, 1, 2];
  myFunction.apply(null, args);
  With ES2015 spread you can now write the above as:

  function myFunction(x, y, z) { }
  var args = [0, 1, 2];
  myFunction(...args);
```


Any argument in the argument list can use the spread syntax and it can be used multiple times.
``` js
function myFunction(v, w, x, y, z) { }
var args = [0, 1];
myFunction(-1, ...args, 2, ...[3]);
```


Spread operator can create a more powerful array literal.

Example: Without ES2015 spread, if you have an array and want to create a new array with the existing one being part of it, the array literal syntax is no longer sufficient and you have to fall back to imperative code, using a combination of push, splice, concat, etc. With spread syntax this becomes much more succinct:
  
```js
  var parts = ['shoulders', 'knees'];
  var lyrics = ['head', ...parts, 'and', 'toes']; // ["head", "shoulders", "knees", "and", "toes"]
```

Note: Typically the spread operators in ES2015 goes one level deep while copying an array. Therefore, they are unsuitable for copying multidimensional arrays. It's the same case with Object.assign() and Object spread operators. Look at the example below for a better understanding.


``` js
  var a =[[1], [2], [3]];
  var b = [...a];
  b.shift().shift();// a is now [[], [2], [3]]
 ```



Note that the spread operator can be applied only to iterable objects:
``` js
  var obj = {"key1":"value1"};
  var array = [...obj]; // TypeError: obj is not iterable
```


### Destructuring
The destructuring assignment syntax is a JavaScript expression that makes it possible to extract data from arrays or objects into distinct variables.

Advantages over old way of manual destructuring:
* it can have default values
* more readable
* less LOC
* inmethod signature

``` js
  var a, b, rest;
  [a, b] = [1, 2];
  console.log(a); // 1
  console.log(b); // 2

  [a, b, ...rest] = [1, 2, 3, 4, 5];
  console.log(a); // 1
  console.log(b); // 2
  console.log(rest); // [3, 4, 5]
   
  // puttin code in parenthesis to make it an expression that can be executed
  ({a, b} = {a:1, b:2});
  console.log(a); // 1
  console.log(b); // 2
  
  // Adding aliases in case of object destructuring
  // in this case a, b will be undefined or have their previous value
  // but one, two will have 1, 2 value respectively
  ({a: one, b: two} = {a: 1, b: 2});
  
  // inmethod signature
  try {
    throw "Error";
  } catch({type, message, fileName, lineNumber}) {
    // do somethin
  }
```


Default Values example: 
``` js
Setting a function parameter's default value

  //  ES5 version

  function drawES5Chart(options) {
    options = options === undefined ? {} : options;
    var size = options.size === undefined ? 'big' : options.size;
    var cords = options.cords === undefined ? { x: 0, y: 0 } : options.cords;
    var radius = options.radius === undefined ? 25 : options.radius;
    console.log(size, cords, radius);
    // now finally do some chart drawing
  }

  drawES5Chart({
    cords: { x: 18, y: 30 },
    radius: 30
  });
  
  //  ES6 version

  function drawES6Chart({size = 'big', cords = { x: 0, y: 0 }, radius = 25} = {}) {
    console.log(size, cords, radius);
    // do some chart drawing
  }

  drawES6Chart({
    cords: { x: 18, y: 30 },
    radius: 30
  });
```

**Note:** 
* In case of object destructuring the name of variables must be same as key of the object. You can set aliases in case you want different names.
* Right side must be an array or object in case of array destructuring or object destructuring respectively.
* In case the left hand side has a variable name that's not matching with any key on right side, then that variable will assigned `undefined`. There has been confusion about refutability patterns that can make distinction between `required` and `not-sure` v

For more, must goto [Destructuring syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) and [Computed Property Name](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names).

































