# ES6-notes
Simple ES6 notes.

## Table of Contents
- [Tail Recursion](#tail-recursion)
- [Declarations](#declarations)
- [Rest Parameters](#rest-parameters)
- [Spread Operators](#spread-operators)
- [Destructuring](#destructuring)
- [Arrow Functions](#arrow-functions)
- [Default Parameters](#default-parameters)
- [Classes](#classes)
- [Computed Properties](#computed-properties)
- [Method Definitions](#method-definitions)
- [Collections](#collections)
- [Modules](#modules)
- [Promises](#promises)
- [Generators](#generators)
- [Template Literals](#template-literals)

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


<sup>[(Back to table of contents)](#table-of-contents)</sup>

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
<sup>[(Back to table of contents)](#table-of-contents)</sup>

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

## Spread Operators

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

**Lexical Scoping [Extra]:** 

<sup>[(Back to table of contents)](#contents)</sup>


## Destructuring
The destructuring assignment syntax is a JavaScript expression that makes it possible to extract data from arrays or objects into distinct variables.

Advantages over old way of manual destructuring:
* it can have default values
* more readable
* less LOC
* inmethod signature

**NOTE:** There is no performance improvement though.

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
* In case the left hand side has a variable name that's not matching with any key on right side, then that variable will assigned `undefined`. There has been confusion about refutability patterns that can make distinction between `required` and `optional` variables in destructuring but it looks like this feature is added by default into destructuring pattern.


For more, goto [Destructuring syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) and [Computed Property Name](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names).


<sup>[(Back to table of contents)](#table-of-contents)</sup>

## Arrow Functions

An arrow function expression has a shorter syntax compared to function expressions and does not bind its own this, arguments, super, or new.target. Arrow functions are always anonymous. These function expressions are best suited for non-method functions and they can not be used as constructors so don't use them for inheritance. They improve performance.

In normal js or es5, every new function defined its own this value (a new object in case of a constructor, undefined in strict mode function calls, the context object if the function is called as an "object method", etc.). This proved to be annoying with an object-oriented style of programming.


ES5 Way:
``` js
  function Person() {
    // The Person() constructor defines `this` as an instance of itself.
    this.age = 0;

    setInterval(function growUp() {
      // In non-strict mode, the growUp() function defines `this` 
      // as the global object, which is different from the `this`
      // defined by the Person() constructor.
      this.age++;
    }, 1000);
  }

  var p = new Person();
```

ES6 Way:
``` js
  function Person(){
    this.age = 0;

    setInterval(() => {
      this.age++; // |this| properly refers to the person object
    }, 1000);
  }

  var p = new Person();
```

**NOTE:** You cannot "rebind" an arrow function. It will always be called with the context in which it was defined. Just use a normal function.

From the ECMASCRIPT-6 Spec:

> Any reference to arguments, super, this, or new.target within an ArrowFunction must resolve to a binding in a lexically enclosing environment. Typically this will be the Function Environment of an immediately enclosing function.



**Parenthesis-Parameter Rule**
``` js
  var x;           
  x = () => {};    // No parameters, MUST HAVE PARENS
  x = (val) => {}; // One parameter w/ parens, OPTIONAL 
  x = val => {}    // One parameter w/o parens, OPTIONAL
  x = (y,z) => {}; // Two or more parameters, MUST HAVE PARENS
  x = y,z => {};   // Syntax Error: must wrap with parens when using multiple parameters
```

Some examples:

``` js
// implicit return: dont need curly braces and return keyword to wrap single line body
var foo = () => "foo"; 

// explicit return
var doo = () => { return "foo's brother doo"; };
```

**Note:** Remember to wrap the object literal in parentheses:
```js
  var func = () => ({ foo: 1 });
```

There is no prototype of arrow functions but even then we get `function` on checking prototype and arrow function cant be used as constructor because of no local binding to `this` and no prototype. 

``` js
console.log( Object.getPrototypeOf(()=>{}) ); // function

var Foo = () => {};
new Foo(); // Foo is not a constructor
```

Another important point is that you can't rebind `this` in case of arrow function:
``` js
  function Logger() {
    this.id = 123;

      this.log = ()=>{
        // this is lexically bound to parent this object
        console.log("ID:", this.id);
      }

      this.old_log = function() {
        // this can be overwritten
        console.log("ID:", this.id);
      }
  }

  var pseudoObj = { id: 321 };
  
  //Using call we are borrowing/stealing Logger's methods
  new Logger().old_log.call(pseudoObj); // 123
  new Logger().log.call(pseudoObj); // 321

```

A good example showing why we need **old function**

``` js
  var foo = {
      // anonymous function this is bound to global this
      log : ()=>{
        this.speak();
      },
      old_log: function() {
        this.speak();
      },
      speak : () => {
        console.log("Hi");
      }
  }
  
  console.log(foo.log()); // Error
  console.log(foo.old_log()); // Hi
```

Checkout [this](http://stackoverflow.com/questions/13441307/how-does-the-this-keyword-in-javascript-act-within-an-object-literal) answer for `this` role in object literal.


<sup>[(Back to table of contents)](#contents)</sup>


## Default Parameters
Default function parameters allow formal parameters to be initialized with default values if no value or undefined is passed. Errors will propogate if a method which throws error is called in default parameter assignment.

Some points to remember:
* Can't assign rest parameters a default value
* Errors will propogate if a method which throws error is called in default parameter assignment
* arguments object doesn't count default parameters, it counts only the no of parameters passed to a function. If you passed 2 arguments to a function having 100 default variables in a function, `arguments.length` will be `2`, not `100`.

What triggers default parameters:
* Empty string doesnt trigger default params. (falsy values except undefined doesnt trigger D.P.)
* `undefined` triggers
* Explicit `undefined` triggers

<sup>[(Back to table of contents)](#table-of-contents)</sup>


## Classes
JavaScript classes introduced in ECMAScript 6 are syntactical sugar over JavaScript's existing prototype-based inheritance. The class syntax is not introducing a new object-oriented inheritance model to JavaScript. JavaScript classes provide a much simpler and clearer syntax to create objects and deal with inheritance.

Some points to note:
* If there is a constructor present in sub-class, it needs to first call super() before using "this".
* Keep in mind that getters/setters cannot have the same name as properties that you set in the constructor. You will end up exceeding the maximum call-stack with infinite recursion when you try to use the setter to set that same property. Example: set foo (value) { this.foo = value 
* methods declared outside will be added to prototype object (prototype object is automatically created when declaring a class and assigned to all the instances of the class).
* if in a subclass, there is no constructor then `super` will be called automatically
* class dont hoist so `runtime error` will be thrown to make an obj of class before defining it
* new class syntx is just syntactical sugar.

``` js
  class Dog {
    // a method name 'constructor' that creates object with some properties
    constructor(name, breed, power) {
      this.name = name;
      this.breed = breed;
      this.power = power;
    }
    // method that object can call
    // Prtotype method - WILL BE ADDED TO PROPTOTYPE Object
    sayMyName () {
      return this.name;
    }
    // static function or class function 
    // that can only be called by class, not 
    // by object
    static bark() {
      console.log("Woof Woof !!")
    }
    
    
    // Infinite loop - resuls to max stack size exceeded error
    // get breed() {
    //  return this.breed;
    // }
    
    // health getter for an object
    get health() {
      return this.power > 20;
    }
    // health setter for an object
    set health(newhealth) {
      this.health = newhealth;
    }
  }
  // The only way to create prototype data properties is by
  // modifying the prototype outside of the declaration.
  // WILL BE ADDED TO PROTOTYPE OBJECT
  Dog.prototype.animalType = "mammals";
 
  // Enumerable prototype methods can also be added 
  // WILL BE ADDED TO PROTOTYPE OBJECT
  Dog.prototype.sniff = function() { return "sniffin!!";};
  
  // Immutable properties can be added with defineProperty.
  Object.defineProperty(Animal.prototype, 'lifeCycle', { value: '10-20 years' });
  
  //static/class properties can be added like this
  Dog.isDomestic = true;
```
Must READ this page => https://reinteractive.com/posts/235-es6-classes-and-javascript-prototypes

### Object.getPrototypeOf vs .prototype [Extra]
``` js
  var a = new Foo();
  Object.getPrototypeOf( a ) === Foo.prototype; // true
```
When a is created by calling new Foo(), one of the things that happens is that a gets an internal [[Prototype]] link to the object that Foo.prototype is pointing at.

## Private methods and Private properties
Private properties can be created by weakmaps, namespace, symbols, closure, and an anti-pattern(encapsulation in constructor). For weakmaps, namespace, closure checkout this [MDN page](https://developer.mozilla.org/en-US/Add-ons/SDK/Guides/Contributor_s_Guide/Private_Properties)

### Anti-Pattern or Enacapsulation in constructor
``` js
  class SomeClass{
    constructor(){
        //private property
        let property="test";
        //public getter
        this.getProperty2=function(){
            return property2;
        }
        //public getter
        this.getProperty=function(){
            return property;
        }
        //public setter
        this.setProperty=function(prop){
            property=prop;
        }
    }
  }

  var obj = new SomeClass();

  console.log(obj.property);// undefined
  console.log(obj.getProperty());// "test"
  obj.setProperty('new test');
  console.log(obj.getProperty());// "new test"
```
However, this will spawn each of these getters and setters on on each instace of `SomeClass` but they won't go on `SomeClass.prototype`.

For more, goto this [page](http://www.zipcon.net/~swhite/docs/computers/languages/object_oriented_JS/methods.html).

### Symbols 
A symbol is a unique and immutable data type. It may be used as an identifier for object properties. The Symbol object is an implicit object wrapper for the symbol primitive data type.

``` js
  var property = Symbol();
  class SomeClass {
      constructor(){
          this[property] = "test";
      }
  }

  var obj = new SomeClass();

  console.log(obj.property);
```

**NOTE**  Symbol doesn't imply any privacy as property easily can be got through `Object.getOwnPropertySymbols`.

## Classes MDN Stuff 

### Defining classes
Classes are in fact "special functions", and just as you can define function expressions and function declarations, the class syntax has two components: class expressions and class declarations.

### Class declarations

One way to define a class is using a class declaration. To declare a class, you use the class keyword with the name of the class ("Polygon" here).
``` js
  class Polygon {
    constructor(height, width) {
      this.height = height;
      this.width = width;
    }
  }
```
### Hoisting

An important difference between function declarations and class declarations is that function declarations are hoisted and class declarations are not. You first need to declare your class and then access it, otherwise code like the following will throw a ReferenceError:
``` js
  var p = new Polygon(); // ReferenceError

  class Polygon {}
```
### Class expressions

A class expression is another way to define a class. Class expressions can be named or unnamed. The name given to a named class expression is local to the class's body.
```js
  // unnamed
  var Polygon = class {
    constructor(height, width) {
      this.height = height;
      this.width = width;
    }
  };

  // named
  var Polygon = class Polygon {
    constructor(height, width) {
      this.height = height;
      this.width = width;
    }
  };
```
**Note:** Class expressions also suffer from the same hoisting issues mentioned for Class declarations.

Class body and method definitions
The body of a class is the part that is in curly brackets {}. This is where you define class members, such as methods or constructors.

> Strict mode
>
> The bodies of class declarations and class expressions are executed in strict mode(automatically).
> Class definition and declaration are automatically compiled in strict mode.

Constructor

The constructor method is a special method for creating and initializing an object created with a class. There can only be one special method with the name "constructor" in a class. A SyntaxError will be thrown if the class contains more than one occurrence of a constructor method.

A constructor can use the super keyword to call the constructor of a parent class.

**Prototype methods**

See also method definitions.
``` js
  class Polygon {
    constructor(height, width) {
      this.height = height;
      this.width = width;
    }

    get area() {
      return this.calcArea();
    }

    calcArea() {
      return this.height * this.width;
    }
  }

  const square = new Polygon(10, 10);

  console.log(square.area);
```
### Static methods

The static keyword defines a static method for a class. Static methods are called without instantiating their class and are also not callable when the class is instantiated. Static methods are often used to create utility functions for an application.
``` js
  class Point {
    constructor(x, y) {
      this.x = x;
      this.y = y;
    }

    static distance(a, b) {
      const dx = a.x - b.x;
      const dy = a.y - b.y;

      return Math.sqrt(dx*dx + dy*dy);
    }
  }

  const p1 = new Point(5, 5);
  const p2 = new Point(10, 10);

  console.log(Point.distance(p1, p2));
```
### Boxing with prototype and static methods

When a static or prototype method is called without an object valued "this" (or with "this" as boolean, string, number, undefined or null), then the "this" value will be undefined inside the called function. Autoboxing will not happen. The behaviour will be the same even if we write the code in non-strict mode.
``` js
  class Animal { 
    speak() {
      return this;
    }
    static eat() {
      return this;
    }
  }

  let obj = new Animal();
  let speak = obj.speak;
  speak(); // undefined

  let eat = Animal.eat;
  eat(); // undefined
```
If we write the above code using traditional function based classes, then autoboxing will happen based on the "this" value overwhich the function was called.
```js
  function Animal() { }

  Animal.prototype.speak = function(){
    return this;
  }

  Animal.eat = function() {
    return this;
  }

  let obj = new Animal();
  let speak = obj.speak;
  speak(); // global object

  let eat = Animal.eat;
  eat(); // global object
```
### Sub classing with extends
The extends keyword is used in class declarations or class expressions to create a class as a child of another class.
``` js
class Animal { 
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(this.name + ' makes a noise.');
  }
}

class Dog extends Animal {
  speak() {
    console.log(this.name + ' barks.');
  }
}

var d = new Dog('Mitzie');
d.speak();
```
** If there is a constructor present in sub-class, it needs to first call super() before using "this".**

One may also extend traditional function-based "classes":
``` js
function Animal (name) {
  this.name = name;  
}

Animal.prototype.speak = function () {
  console.log(this.name + ' makes a noise.');
}

class Dog extends Animal {
  speak() {
    console.log(this.name + ' barks.');
  }
}

var d = new Dog('Mitzie');
d.speak();
```
**Note** that classes cannot extend regular (non-constructible) objects. If you want to inherit from a regular object, you can instead use Object.setPrototypeOf():
``` js
  var Animal = {
    speak() {
      console.log(this.name + ' makes a noise.');
    }
  };
  class Dog {
    constructor(name) {
      this.name = name;
    }
    speak() {
      console.log(this.name + ' barks.');
    }
  }

  Object.setPrototypeOf(Dog.prototype, Animal);

  var d = new Dog('Mitzie');
  d.speak();
```
Species
You might want to return Array objects in your derived array class MyArray. The species pattern lets you override default constructors.

For example, when using methods such as map() that returns the default constructor, you want these methods to return a parent Array object, instead of the MyArray object. The Symbol.species symbol lets you do this:
``` js
  class MyArray extends Array {
    // Overwrite species to the parent Array constructor
    // using computed properties that is a new feature in ES6
    static get [Symbol.species]() { return Array; }
  }

  var a = new MyArray(1,2,3);
  var mapped = a.map(x => x * x);

  console.log(mapped instanceof MyArray); // false
  console.log(mapped instanceof Array);   // true
```
Super class calls with super
The super keyword is used to call functions on an object's parent.
```js
  class Cat { 
    constructor(name) {
      this.name = name;
    }

    speak() {
      console.log(this.name + ' makes a noise.');
    }
  }

  class Lion extends Cat {
    speak() {
      super.speak();
      console.log(this.name + ' roars.');
    }
  }
```
### Mix-ins
Abstract subclasses or mix-ins are templates for classes. An ECMAScript class can only have a single superclass, so multiple inheritance from tooling classes, for example, is not possible. The functionality must be provided by the superclass.

A function with a superclass as input and a subclass extending that superclass as output can be used to implement mix-ins in ECMAScript:
``` js
  var calculatorMixin = Base => class extends Base {
    calc() { }
  };

  var randomizerMixin = Base => class extends Base {
    randomize() { }
  };
```
A class that uses these mix-ins can then be written like this:
``` js
  class Foo { }
  class Bar extends calculatorMixin(randomizerMixin(Foo)) { }
  // calculatorMixin(randomizerMixin(Foo))
  // calculatorMixin --> randomizerMixin --> Foo
  // {calc: calc}
```


### Extend classes via Expression
``` js
  class a {
    constructor() {
        this.prop = 'a';
      }

  }

  class b {
    constructor() {
        this.prop = 'b';
      }

  }

  class c extends f() {
    constructor() {
        super();
      }
  }

  function f() {
    if (true)
        return a;
      else 
        return b;
  } 


  var cc = new c();

  console.log(cc.prop, cc instanceof a, cc instanceof b);
  // logs "a, true, false"
```

<sup>[(Back to table of contents)](#table-of-contents)</sup>


## Computed Properties
Starting with ECMAScript 2015, the object initializer syntax also supports computed property names. That allows you to put an expression in brackets [], that will be computed as the property name. This is symmetrical to the bracket notation of the property accessor syntax, which you might have used to read and set properties already. Now you can use the same syntax in object literals, too:

``` js
  // Computed property names (ES6)
  var i = 0;
  var a = {
    ["foo" + ++i]: i,
    ["foo" + ++i]: i,
    ["foo" + ++i]: i
  };

  console.log(a.foo1); // 1
  console.log(a.foo2); // 2
  console.log(a.foo3); // 3

  var param = 'size';
  var config = {
    [param]: 12,
    ["mobile" + param.charAt(0).toUpperCase() + param.slice(1)]: 4
  };

  console.log(config); // { size: 12, mobileSize: 4 }
```
For more, goto this [page](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer).

<sup>[(Back to table of contents)](#table-of-contents)</sup>


## Method Definitions

Starting with ECMAScript 2015 (ES6), a shorter syntax for method definitions on objects initializers is introduced. It is a shorthand for a function assigned to the method's name.

Syntax
``` js
  var obj = {
    property( parameters… ) {},
    *generator( parameters… ) {},
  // also with computed keys:
    [property]( parameters… ) {},
    *[generator]( parameters… ) {},
  // compare ES5 getter/setter syntax:
    get property() {},
    set property(value) {}
  };
```
Description
The shorthand syntax is similar to the getter and setter syntax introduced in ECMAScript 2015.

Given the following code:
```js
  var obj = {
    foo: function() {
      /* code */
    },
    bar: function() {
      /* code */
    }
  };
  You are now able to shorten this to:

  var obj = {
    foo() {
      /* code */
    },
    bar() {
      /* code */
    }
  };
```
**Note : **The shorthand syntax uses named function instead of anonymous functions (as in …foo: function() {}…). Named functions can be called from the function body (this is impossible for anonymous function as there is no identifier to refer to). For more details, see function.
Shorthand generator methods

Generator methods can be defined using the shorthand syntax as well. When using them,

the asterisk (*) in the shorthand syntax must be before the generator property name. That is, * g(){} will work but g *(){} will not;
non-generator method definitions may not contain the yield keyword. This means that legacy generator functions won't work either and will throw a SyntaxError. Always use yield in conjunction with the asterisk (*).
``` js
  // Using a named property (pre-ES6)
  var obj2 = {
    g: function*() {
      var index = 0;
      while(true)
        yield index++;
    }
  };

  // The same object using shorthand syntax
  var obj2 = { 
    * g() {
      var index = 0;
      while(true)
        yield index++;
    }
  };

  var it = obj2.g();
  console.log(it.next().value); // 0
  console.log(it.next().value); // 1
 ```
Method definitions are not constructable

All method definitions are not constructors and will throw a TypeError if you try to instantiate them.
```js
  var obj = { 
    method() {},
  };
  new obj.method; // TypeError: obj.method is not a constructor

  var obj = { 
    * g() {} 
  };
  new obj.g; // TypeError: obj.g is not a constructor (changed in ES2016)
```
Examples
Simple test case
```js
  var obj = {
    a : "foo",
    b(){ return this.a; }
  };
  console.log(obj.b()); // "foo"

  Computed property names

  The shorthand syntax also supports computed property names.

  var bar = {
    foo0 : function (){return 0;},
    foo1(){return 1;},
    ["foo" + 2](){return 2;},
  };

  console.log(bar.foo0()); // 0
  console.log(bar.foo1()); // 1
  console.log(bar.foo2()); // 2
```

## Collections

### Set
The Set object lets you store unique values of any type, whether primitive values or object references.

#### Syntax
new Set([iterable]);
Parameters

#### iterable
If an iterable object is passed, all of its elements will be added to the new Set. null is treated as undefined.
Description
Set objects are collections of values. You can iterate through the elements of a set in insertion order. A value in the Set may only occur once; it is unique in the Set's collection.

For iterable , goto this [page](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of).

**Imporant Point**
#### Difference between for...of and for...in

The for...in loop will iterate over all enumerable properties of an object.

The for...of syntax is specific to collections, rather than all objects. It will iterate in this manner over the elements of any collection that has a [Symbol.iterator] property.

The following example shows the difference between a for...of loop and a for...in loop.
```js
  Object.prototype.objCustom = function () {}; 
  Array.prototype.arrCustom = function () {};

  let iterable = [3, 5, 7];
  iterable.foo = "hello";

  for (let i in iterable) {
    console.log(i); // logs 0, 1, 2, "foo", "arrCustom", "objCustom"
  }

  for (let i of iterable) {
    console.log(i); // logs 3, 5, 7
  }
```


#### Value equality

Because each value in the Set has to be unique, the value equality will be checked. In an earlier version of ECMAScript specification this was not based on the same algorithm as the one used in the === operator. Specifically, for Sets, +0 (which is strictly equal to -0) and -0 were different values. However, this was changed in the ECMAScript 2015 specification. See "Value equality for -0 and 0" in the browser compatability table for details.

Also, NaN and undefined can also be stored in a Set. **NaN is considered the same as NaN (even though NaN !== NaN)**.

#### Properties
Set.length
The value of the length property is 0.
get Set[@@species]
The constructor function that is used to create derived objects.
Set.prototype
Represents the prototype for the Set constructor. Allows the addition of properties to all Set objects.
Set instances
All Set instances inherit from Set.prototype.

#### Properties

Set.prototype.constructor
Returns the function that created an instance's prototype. This is the Set function by default.

Set.prototype.size
Returns the number of values in the Set object.

#### Methods

Set.prototype.add(value)
Appends a new element with the given value to the Set object. Returns the Set object.

Set.prototype.clear()
Removes all elements from the Set object.

Set.prototype.delete(value)
Removes the element associated to the value and returns the value that Set.prototype.has(value) would have previously returned. Set.prototype.has(value) will return false afterwards.

Set.prototype.entries()
Returns a new Iterator object that contains an array of [value, value] for each element in the Set object, in insertion order. This is kept similar to the Map object, so that each entry has the same value for its key and value here.

Set.prototype.forEach(callbackFn[, thisArg])
Calls callbackFn once for each value present in the Set object, in insertion order. If a thisArg parameter is provided to forEach, it will be used as the this value for each callback.

Set.prototype.has(value)
Returns a boolean asserting whether an element is present with the given value in the Set object or not.

Set.prototype.keys()
Is the same function as the values() function and returns a new Iterator object that contains the values for each element in the Set object in insertion order.

Set.prototype.values()
Returns a new Iterator object that contains the values for each element in the Set object in insertion order.

Set.prototype[@@iterator]()
Returns a new Iterator object that contains the values for each element in the Set object in insertion order.

``` js
  mySet2 = new Set([1,2,3,4]);
  mySet2.size; // 4
  
  // equivalent to Array.from(mySet2)
  [...mySet2]; // [1,2,3,4]

  console.log([...mySet2], Array.from(mySet2))
  // logs same result
```

for examples on set, goto this [page](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Set);

### Array.from 

Look `Array.from` that can create array or iterable object from sets, string etc. The Array.from() method creates a new Array instance from an array-like or iterable object.

Syntax
`Array.from(arrayLike[, mapFn[, thisArg]])`
Parameters

#### arrayLike
An array-like or iterable object to convert to an array.
#### mapFn
Optional. Map function to call on every element of the array.
#### thisArg
Optional. Value to use as this when executing mapFn.
Return value

A new Array instance.

#### Description
Array.from() lets you create Arrays from:

array-like objects (objects with a length property and indexed elements) or
iterable objects (objects where you can get its elements, such as Map and Set).

``` js
  // Using an arrow function as the map function to
  // manipulate the elements
  Array.from([1, 2, 3], x => x + x);      
  // [2, 4, 6]


  // Generate a sequence of numbers
  Array.from({length: 5}, (v, k) => k);    
  // [0, 1, 2, 3, 4]
```

For more goto this [page](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/from).

### Map
The Map object is a simple key/value map. Any value (both objects and primitive values) may be used as either a key or a value.

#### Syntax
new Map([iterable])
Parameters


#### iterable
Iterable is an Array or other iterable object whose elements are key-value pairs (2-element Arrays). Each key-value pair is added to the new Map. null is treated as undefined.
Description
A Map object iterates its elements in insertion order — a for...of loop returns an array of [key, value] for each iteration.

It should be noted that a Map that is a map of an object, especially a dictionary of dictionaries, will only map to the object's insertion order -- which is random and not ordered.


####  Difference between object and map
``` js
  var arr = [1,3];
  var obj = {};
  // arr will auto convert into string form by arr.toString() and then it will save.
  obj[arr] = "asd";
  var map = new Map();
  // proper arr will be saved in map
  map.set(arr, "asd");
```
Some points:

* An Object has a prototype, so there are default keys in the map that could collide with your keys if you're not careful. This could be bypassed by using map = Object.create(null) since ES5, but was seldom done.
* The keys of an Object are Strings and Symbols, whereas they can be any value for a Map, including functions, objects, and or any primitive.
* You can get the size of a Map easily with the size property, while the size of an Object must be determined manually. 

Properties
Map.length
The value of the length property is 0.
get Map[@@species]
The constructor function that is used to create derived objects.
Map.prototype
Represents the prototype for the Map constructor. Allows the addition of properties to all Map objects.
Map instances
All Map instances inherit from Map.prototype.

Properties

Map.prototype.constructor
Returns the function that created an instance's prototype. This is the Map function by default.
Map.prototype.size
Returns the number of key/value pairs in the Map object.
Methods


#### Map.prototype.clear()
Removes all key/value pairs from the Map object.

#### Map.prototype.delete(key)
Removes any value associated to the key and returns the value that Map.prototype.has(key) would have previously returned. Map.prototype.has(key) will return false afterwards.

#### Map.prototype.entries()
Returns a new Iterator object that contains an array of [key, value] for each element in the Map object in insertion order.

#### Map.prototype.forEach(callbackFn[, thisArg])
Calls callbackFn once for each key-value pair present in the Map object, in insertion order. If a thisArg parameter is provided to forEach, it will be used as the this value for each callback.

#### Map.prototype.get(key)
Returns the value associated to the key, or undefined if there is none.

#### Map.prototype.has(key)
Returns a boolean asserting whether a value has been associated to the key in the Map object or not.

#### Map.prototype.keys()
Returns a new Iterator object that contains the keys for each element in the Map object in insertion order.

#### Map.prototype.set(key, value)
Sets the value for the key in the Map object. Returns the Map object.

#### Map.prototype.values()
Returns a new Iterator object that contains the values for each element in the Map object in insertion order.

#### Map.prototype[@@iterator]()
Returns a new Iterator object that contains an array of [key, value] for each element in the Map object in insertion order.


For examples goto this [page](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Map).

### WeakMaps

The WeakMap object is a collection of key/value pairs in which the keys are weakly referenced.  The keys must be objects and the values can be arbitrary values.

Syntax
`new WeakMap([iterable])`

#### Parameters

#### iterable
Iterable is an Array or other iterable object whose elements are key-value pairs (2-element Arrays). Each key-value pair will be added to the new WeakMap. null is treated as undefined.
Description
**Keys of WeakMaps are of the type Object only. Primitive data types as keys are not allowed (e.g. a Symbol can't be a WeakMap key).**

### WeakMap Properties and differences
* weak reference
* cant be iterated
* size cant be found coz of weak reference
* keys can be primitive data type like Symbols etc, keys must be of type object

### Why WeakMap?

The experienced JavaScript programmer will notice that this API could be implemented in JavaScript with two arrays (one for keys, one for values) shared by the four API methods. Such an implementation would have two main inconveniences. The first one is an O(n) search (n being the number of keys in the map). The second one is a memory leak issue. With manually written maps, the array of keys would keep references to key objects, preventing them from being garbage collected. In native WeakMaps, references to key objects are held "weakly", which means that they do not prevent garbage collection in case there would be no other reference to the object.

Because of references being weak, WeakMap keys are not enumerable (i.e. there is no method giving you a list of the keys). If they were, the list would depend on the state of garbage collection, introducing non-determinism. If you want to have a list of keys, you should use a Map.

For more goto this [page](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/WeakMap).

## Template Literals
Template literals are string literals allowing embedded expressions. You can use multi-line strings and string interpolation features with them. They were called "template strings" in prior editions of the ES2015 / ES6 specification.

Template literals are enclosed by the back-tick (` `) (grave accent) character instead of double or single quotes. Template literals can contain place holders. These are indicated by the Dollar sign and curly braces (${expression}). The expressions in the place holders and the text between them get passed to a function. The default function just concatenates the parts into a single string. If there is an expression preceding the template literal (tag here),  the template string is called "tagged template literal". In that case, the tag expression (usually a function) gets called with the processed template literal, which you can then manipulate before outputting. To escape a back-tick in a template literal, put a backslash \ before the back-tick.


## Modules


## Promises


## Generators


#### Multi-line strings

Any new line characters inserted in the source are part of the template literal. Using normal strings, you would have to use the following syntax in order to get multi-line strings:

```js
console.log("string text line 1\n"+
"string text line 2");
// "string text line 1
// string text line 2"
```

To get the same effect with multi-line strings, you can now write:

```js
console.log(`string text line 1
string text line 2`);
// "string text line 1
// string text line 2"
```
#### Expression interpolation

In order to embed expressions within normal strings, you would use the following syntax:
```js
var a = 5;
var b = 10;
console.log("Fifteen is " + (a + b) + " and\nnot " + (2 * a + b) + ".");
// "Fifteen is 15 and
// not 20."
```js

Now, with template literals, you are able to make use of the syntactic sugar making substitutions like this more readable:
``` js
var a = 5;
var b = 10;
console.log(`Fifteen is ${a + b} and\nnot ${2 * a + b}.`);
// "Fifteen is 15 and
// not 20."
```
#### Tagged template literals

A more advanced form of template literals are tagged template literals. With them you are able to modify the output of template literals using a function. The first argument contains an array of string literals ("Hello " , " world", and "" in this example). The second, and each argument after the first one, are the values of the processed (or sometimes called cooked) substitution expressions ("15" and "50" here). In the end, your function returns your manipulated string. There is nothing special about the name tag in the following example. The function name may be anything you want.
``` js
  var a = 5;
  var b = 10;

  function tag(strings, ...values) {
    console.log(strings[0]); // "Hello "
    console.log(strings[1]); // " world "
    console.log(strings[2]); // ""
    console.log(values[0]);  // 15
    console.log(values[1]);  // 50

    return "Bazinga!";
  }

  tag`Hello ${ a + b } world ${ a * b }`;
  // "Bazinga!"
```
Tag functions don't need to return a string, as shown in the following example.


#### Raw strings

The special raw property, available on the first function argument of tagged template literals, allows you to access the raw strings as they were entered.

function tag(strings, ...values) {
  console.log(strings.raw[0]); 
  // "string text line 1 \n string text line 2"
}

tag`string text line 1 \n string text line 2`;
In addition, the String.raw() method exists to create raw strings just like the default template function and string concatenation would create.

``` js
  String.raw`Hi\n${2+3}!`;
  // "Hi\n5!"

```

For more, goto this [page](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals).

































