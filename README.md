# ES6-notes
Simple ES6 notes from Aaron Frost.

## Content
* Tail Recursion
* Declarations

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

  function definitionHoisted() {
      console.log("Definition hoisted!");
  }

  var definitionNotHoisted = function () {
      console.log("Definition not hoisted!");
  };
```












