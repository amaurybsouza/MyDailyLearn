#### Notes

- When browser find the `<script>` tag to download, all other parallel downloading halts. (6 files can be downloaded by modern browser)
- if we add `async` in `<script>` tag then it would be downloaded asynchronously
    - e.g. `<script type="text/javascript" src="http://www.abc.com/test.js" async></script`
- Efficient choices for `string concatenation` is `+=`
    - e.g. const a = ''; a += 'b'; a += 'c';
- Use a document fragment to insert additions all at once. Fragments are invisible containers that hold multiple DOM elements without
  being a node itself.

```javascript
var list = document.getElementById('kotwList');
var kotw = ["Jenna Rangespike", "Neric Farthing", "Darkin Stonefield"];

var fragment = document.createDocumentFragment();
// First we create a fragment, which will function as a staging area to hold all of our new `li` elements

const kotwLen = kotw.length;
for (var i=0, len = kotwLen; i<len; i++) {
    var element = document.createElement('li');
    element.appendChild(document.createTextNode(kotw[i]));
    fragment.appendChild(element);
// now we add each new `li` element to the staging fragment, instead of to the document itself
}
list.appendChild(fragment);
// Finally, we append all of our new text nodes in one fell swoop, using only one DOM touch!
```

- Every `var` keywords adds a look-up for the JavaScript parser that can be avoided with comma extensions.
- For concatenation over an array's contents,`join()` method is faster (inherited from the Array prototype)
- `console.time('timer name')` and `console.timeEnd('timer name')` 
    - to unite timer boundaries into one timer, their parameter labels must match
    - output: timer name 0.036 ms
    - `console.time` automatically prefaces the time measurement with the label we passed in a `parameter, plus a colon`.
- The `triple-equal (===)` comparator compares both `type and contents`. `===` seeks a `strict` equality.
- The `instanceof` operator helps identity objects.
- `Exceptions`: are run time errors.
- JavaScript `with` keyword is somewhat unreliable and often expensive.
- Keyword `with` tries and technically does limit redundancy, but makes us rather unsure about scope.
- Use variables to cache objects (it is more clear than `with` and also no lengthy nested object names!)
- JavaScript's `eval` keyword may not be evil, but it can affect legibility, an ability to debug and performance.

#### `Closures` and `References`
- One of the JS most powerful features is closures.
- With `closures`, scopes `always` keep access to the outer scope, in which they were defined.
```js
// Emulating private variables
function Counter(start) {
  var count = start;
   return {
    increment: function() {
      count++;
    },
    get: function() {
      return count;
    }
   }
}
var foo = Counter(4);
foo.increment();
foo.get(); // 5
```
- Here, `Counter` returns `two closures`: `increment` & `get` functions.
- Both of these functions keep a `reference` to the scope of `Counter` and, therefore, always keep access to the `Count`
  variable that was defined in that scope.
```js
// Closures Inside Loops
for(var i = 0; i < 10; i++) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
}
```
- The above will not output the numbers `0 to 9`, but the number `10 ten times`
- The anonymous function keeps a reference to i. At the time 'console.log' gets called, the `for loop` has already finished, and
 the value of `i` has been set to `10`
- In order to get the desired behavior, it is necessary to create a `copy` of the value of `i`
```js
// 1. Avoiding the Reference Problem
for(var i = 0; i < 10; i++) {
  (function(e) {
    setTimeout(function() {
      console.log(e);
    }, 1000)
  })(i);
}
```
- The anonymous outer function gets called immediately with `i` as its first argument and will receive a copy of the value of `i` as it's parameter `e`
- The anonymous function that gets passed to `setTimeout` now has a reference to `e`, whose value does not get changed by the loop.

```js
// 2. Avoiding the Reference Problem
for(var i = 0; i < 10; i++) {
  setTimeout((function(e) {
    return function() {
      console.log(e);
    }
    })(i), 1000)
}
```
- There is another possible way of achieving this, which is to return a function from the `anonymous wrapper` that will then
  have the same behavior as the code above.
```js
// 3. Avoiding the Reference Problem
for(var i = 0; i < 10; i++) {
  setTimeout(function(e) {
    console.log(e);
  }, 1000, i)
}
```
- The other popular way to achieve this is to add an additional argument to the `setTimeout` function, which passes these arguments to the callback
```js
// 4. Avoiding the Reference Problem
for(var i = 0; i < 10; i++) {
  setTimeout(console.log.bind(console, i), 1000);
}
```
- There's yet another way to accomplish this by using `.bind`, which can bind a `this` context and arguments to function.

#### Closures Help in Function "Construction Zones"
- A closure wraps up an entire environment, binding necessary variables from other scopes.
- A closure can make the creation of very similar functions untra-efficient.

```js
function buildCoveTicketMaker(transport) {
  return function (name) {
    alert ('Here transport is ', + transport + 'and name is ' + name);
  }

var getSubmarineTicket = buildCoveTicketMaker('Submarine');       | return same, function (name) {
var getSubmarineTicket = buildCoveTicketMaker('Battaleship');     |   alert ('Here transport is ', + transport + 'and name is ' + name);
var getSubmarineTicket = buildCoveTicketMaker('Gaint Seagull');   | }
                                                                  | but every time each function has a closure 'buildCoveTicketMaker' function
                                                                  | with transport = Submarine/Battaleship/Gaint Seagull
}
```

#### Loops with Closure: A Cautionary Tale (Be A Closure Master!)
```js
function assignTorpedo (name, passengerArray) {
  var torpedoAssignment;
  for (var i = 0; i < passengerArray.length; i++) {
    if (passengerArray[i] == name {
      torpedoAssignment = function () {
        alert("Ahoy, " + name + "!\n" +
        " Man your post at Torpedo # " + (i+1) + "!");
      }
    }
  }
  return torpedoAssignment;
}

var subPassengers = ["Luke", "Leia", "Han", "Chewie", "Yoda", "Boba"];
var giveAssignment = assignTorpedo("Chewie", subPassengers);

gitveAssignment();   // it shows "... Torpedo #6!" instead of "... Torpedo #4!"
```
```js
// solutions #1:
function assignTorpedo (name, passengerArray) {
  for (var i = 0; i < passengerArray.length; i++) {
    if (passengerArray[i] == name {
      return function () {                                // immediately return the function
        alert("Ahoy, " + name + "!\n" +                   // so that i variable don't get the
        " Man your post at Torpedo # " + (i+1) + "!");    // chance of increment
      }
    }
  }
}
```

```js
// solutions #2:
function assignTorpedo (passengerArray) {
  return function(name) {
    for (var i = 0; i < passengerArray.length; i++) {       // since we've put the loop inside the returned function,
        if (passengerArray[i] == name {                     // i variable will come directly from that local scope
          alert("Ahoy, " + name + "!\n" +
          " Man your post at Torpedo # " + (i+1) + "!");   
        }
      }
  }
}
``


#### Change the `tooltip` value with js when button is clicked
```
// html part
  <a class="btn" data-toogle="tooltip" data-placement="top" data-original-title="Copy">Click me</span></a>
/*
  N.B. `data-original-title` is changing(Copy/Copied) dynamically by js,
 can also use `title`, `title` originally converted to `data-original-title`
*/

// Trigger this code for `click` event
  const html = e.target.closest('.tooltip-show');
  $(html).removeAttr('data-original-title').attr('data-original-title', 'Copied');
  $(html).tooltip('show');

// Trigger this code for `mouseleave` event
  $(('.tooltip-show')).removeAttr('data-original-title').attr('data-original-title', 'Copy');
```

#### Check if specific html portion is visible or not!
 ``` 
 var html = $(this).closest('.box');
  html.find('.file-data').toggle();
  const btnTake = html.find('.show-hide-btn');
  if(html.find('.file-data').is(':visible')) {
     btnTake.text('Hide');
  } else {
     btnTake.text('Show');
  }
```

#### Check if an object has a key
```js
if (Object.prototype.hasOwnProperty.call('myObject', 'key') {
  return true;
}

Or,

if ('key' in 'myObject') {
  return true;
}
```
#### Why `Eval` should be avoided!
```
function regimentmotto(number, motto) {
    eval('regiment' + number + ".motto = '" + motto + "'");
}
// here the `eval` method will take a string as a parameter, start the JS compiler, and treat that string as if it were
// a line of code to execute  
regimentmotto(1, 'The king's motto');  // will give compile error for `king's` "'" comma, compiler treats as incomplete string. 

// Fix compile error
eval('regiment' + number).motto = motto;
// Try to minimize the operations that your new mini-program needs to engage in... which will also have the benefit of
// improving your debug compability.

```
- Eval is most often misused for just this kind of `mapping number's to objects`, but an is much more efficient. 
- Eval might be usefull for dynamic code or uncontrollable data, but it's still treating the string as a program to expensively compile 
- Use `JOSN,parse()` to ensure only json is accepted. JOSN.parse or a parser library that recognizes JSON, helps to avoid both th security
  and performance issues posed by `eval`
- Leaving of {}. Please use {} even when we have only one statement of `if` or `for`.

- Decimals number in js is `wacky`
    - js uses binary floating point values to handle all of its decimal based operations. e.g. console.log(0.1 + 0.2), output: 0.30000000000000004 
    - Method `toFixed()` allow us to select the exact amount of decimal places to display. 
    ```js
    var num = 0.1 + 0.2 + 0.3;
    console.log(num.toFixed(1))  // output: 0.6;
    console.log(num.toFixed(4))  // output: 0.6000;
    ```
    - `parseFloat()` Turns Strings with Decimals into Numbers.
    - `parseInt()` also converts numerical strings. But it seeks the first available integer at the front of a string.
    ```
    parseInt("88 keys on a piano"); // output: 88
    parseFloat("1.234 keys on a bag"); // output: 1.234
    parseInt("There are 88 keys on a piano"); // NaN, cause string does not begin with a acceptable value
    parseInt("7.89") // output: 7, trim up descimals without `rounding` 
    parseInt("1.23", 10) // parseInt(string, radix), it allows any `radix` value from `2 to 36`. 
    ```
    - typeof(NaN) // output: number
        - console.log(NaN === NaN) // output: false
        - js function isNaN() is used to check if a value is NaN. 
        - isNaN("34") // output: false
    - If unsure about `data type`, but highly reliant on a `Number` use `typeof` and `isNaN()` as a best practice
        - function checkNumber(data) { return typeof data === "number" && !isNaN(data); }
        - checkNumber(640) // output: true
        - checkNumber("640") // output: false, false by typeof data === "number"
        - checkNumber("NaN") // output: false, false by !isNaN()
    - An object that groups and protects related data and methods in JavaScript files is called a `Namespace`.
    - If built well, `namespaces` remain `agnostic` of other namespaces.
    - `Namespaces` reduce global `footprint` while also keeping data grouped around their intended functionality.
    - `Closure` of js is used to cause some properties to be private, bound only to a surrounding function's local scope,
       and some properties to be public, accessible by all holders of the namespace.
    - `Private` properties are `created` in the local scope of the `function expression`. `public` properties are built withtin
       the `object` which is then `returned` to become the `namespace`. Access to `private` data is thus prossible only
       because of `closure` within the larger `module`.
    - If a module reference globally-scoped variable, it's a best practice to bring them into the scope of anonymous closure
      through the use of a `imports` technique.
    - Our `imports` ensures clarity of `scope` within a `module`. By using a `parameter`, we protect the `global` data 
      that might have been `overwritten`. All imported data becomes `locally` scoped to the `anonymous function`, to be used in
      `closure`. Thus, when compared with `searching` the entire `scopre chain`, `imports` are both clearer and faster.
    
    - `Augmentation` provides extra properties for existing modules.
    - In simple `augmentation`, the `module` file and the augmentation file `module` share their `private state`.
      Augmentation module `properties` may only `access` the `private` data from their file's `closure`. `Private` data
      from the `original` closure `will not` be lost, and `will` be accessible to all original `properties`.

#### Scope Chain (how js works!)
- Everything is executed in an Execution Context
- Function invocation creates a new Execution Context
- Each Execution Context has: 
    - Its own Variable Environment
    - Special `this` object
    - Reference to its Outer Environment
- Global scope does not have an Outer Environment as it's the most outer there is.
- Execute flows -
```
Referenced (not defined) variable will be searched for in its current scope first.
If not found, then Outer Reference will be searched.
If not found, the Outer Reference's Outer Reference will be searched, etc.
This will keep going until the Global scope.
If not keep going scope, the variable is undefined.
```

#### JavaScript: The Good Parts
- 100 and 1e2 are the same number
- `NaN` is not equal to any value, including itself. Detect `NaN` with the `isNaN(number)` function.
- `Infinity` is any value greater that 1.79769313486231570e+308.
- JS does not have an integer type
    - integers are a subset of doubles instead of a separate data type
- JS defines `7` built-in types
    - `Object` and `6 Primitives`
- Object type is a collection of name/value pairs
- Primitive type can contain a `single, immutable` value
- Undefined means variable memory has been allocated but no value has ever been explicitly set yet.
- What is `False` to JS ?
    - `false`, `null`, `undefined`, `""`, `0`, `NaN`
- In JS, primitives are passed by value, `objects` are `passed by referenece`

```
var a = { x: 7 };
var b = a;
console.log(a);   // output: 7
console.log(b);   // output: 7

b.x = 5;
console.log('After b.x update:');
console.log(a);   // output: 5  (a is also changed!)
console.log(b);   // output: 5

```
- When we have an `inner function` within another function,
  this keyword starts pointing to the `global object (window)`.

```

// Object literals and `this`
var literalCircle = {
  radius: 10,

  getArea: function() {
    console.log(this);       // Object {radius: 10}

    var increaseRadius = function () {
      this.radius = 20;
      // Here, this is referring to global object `window` because of inner function [getArea() -> increaseArea()]
    };
    increaseRadius();
    console.log(this.radius);

    return Math.PI * Math.pow(this.radius, 2);
  }
};
console.log(literalCircle.getArea());
```

- Immediately Invoked Function Expression (`IIFE`)
```
(function () {
  console.log('This function will be invoked immediately');
})();
```

- `Function.prototype.call()` - The `call()` method calls a function with a `given this` value and arguments provided individually.
    - A different `this object` can be assigned when calling an existing function. this refers to the current object, the calling object.
    With `call`, you can write a method once and then inherit it in another object, without having to rewrite the method for the new object.
- With `call() or apply()` we can set the value of `this`, and invoke a function as a new method of an existing object.
- Using call to invoke a function and specifying the `context` for `this`. In below example, when we will call great the value of this
 will be bind to boject `i`.

```
function greet() {
    var reply = [this.person, 'Is An Awesome', this.role].join(' ');
    console.log(reoly);
}

var i = {
    person: 'Douglas Crockford',
    role: 'Javascript Developer'
}

greet.call(i); // output: Douglas Crockford Is An Awesome Javascript Developer
```

#### Common Misconception about JSON
- JSON is a lightweight data representation
- Great format for passing data from server to client & back
- Syntax is based on JS object literal
    - But JSON is NOT JS object literal
    - JSON is just a string
- Need to convert JSON into a JS object
- Converting JSON to String & Back to JSON
    - var obj = JSON.parse(jsonString);    // converts from json string to object
    - var str = JSON.stringify(obj);       // converts from object to JSON 
    
---
 
## Closures and Event Listeners
#### The Problem:

```js
var nums = [1, 2, 3];
var len = nums.length;

for (var i = 0; i < len; i++ ) {
  var nowNum = nums[i];
  
  // Creating a DOM element for the number                  // Specifically, we're alerting the `nowNum` variable that's defined
  var div = document.createElement('div');                  // outside of this inner function. Each of these inner functions are
  div.textContent = nowNum;                                 // pointing to the same `nowNum` variable... the one that changes on
                                                            // each iteration, and which equals 3 at the end of the loop.
  // when click, alert the value of `num`                   // Whenever the anonymous function is called on the click event, the
  div.addEventListener('click', function(){                 // function will reference the same `nowNum` (which now equals 3)
    alert(nowNum);                                          
  });                                                       
                                                            
  // finally add the `div` element to the document          
  document.body.appendChild(div);                           
}
#### Solve the problem:                                     // function(numCopy) is outer function and invoking immediately
Output: 3 3 3       (when we click on 1, 2, 3 !!)           // by wrapping it in parentheses and calling it right away, passing
                                                            // in `nowNum`. Inside the outer function the value is known as `numCopy` 
elem.addEventListener('click', (function(numCopy) {         // Now, if doesn't matter that `nowNum` changes later down the line.
```                                                         // We stored the value of `nowNum` in `numCopy` inside our outer function.
    return function() {                                     // Lastly, the outer function returns the inner function to the event listener.
      alert(numCopy);                                       // Because of the way JavaScript scope works, that inner function has access to
    };                                                      // `numCopy` which will never change.
})(nowNum));                                                



## Object Oriented JavaScript (Udacity)
#### Extend vs Object.create(): Property Lookup of Delegated Objects

```js
const gold = {a: 1};
console.log(gold.a);                           // 1
console.log(gold.z);                           // undefined

const blue = extend({}, gold);                 // copy one time
blue.b = 2;
console.log(blue.b);                           // 2
console.log(blue.z);                           // undefined

var rose = Object.create(gold);                // ongoing lookup time delegation
console.log(rose.a);                           // 1
console.log(rose.b);                           // undefined, no `b` in both rose or gold
rose.b = 3;
console.log(rose.b);                           // 3
console.log(rose.z);                           // undefined, no `z` in both rose or gold

gold.z = 3;
console.log(blue.z);                           // undefined, no 'z' in blue, here 'extend' was one time copy from 'gold'
console.log(rose.z);                           // 3, since there is no 'z' in rose, delegation goes through to gold, which does have gold.z
```

```js
function theBridgeOfHoistingDoom() { |
                                     | Alrighty, here’s the hoisted version. The function looks for any variables to
                                     | create space for, finds ring and power, and sets them both to undefined. The
  function balrog() {                | order of declared functions is balrog, elf, balrog, wizard, and elf. When older
    return "fire";                   | versions of the loaded functions are replaced, we are left with balrog, wizard,
  }                                  | and then elf. The only executable code that actually ever runs are the lines
  var ring;                          | that precede and include the return of the call to wizard
  function elf() {                   |
    return "pointy ears";            |
  }                                  | function theBridgeOfHoistingDoom() {
  ring = wizard;                     |   var ring = undefined;
  wizard = balrog;                   |   var power = undefined;
  return wizard();                   |   function balrog() {
  function balrog() {                |     return "whip";
    return "whip";                   |   }
  }                                  |   function wizard() {
  function wizard() {                |     return "white";
    return "white";                  |   }
  }                                  |   function elf() {
  var power = ring();                |     return "immortal";
  return elf();                      |   }
  function elf() {                   |   ring = wizard;
    return "immortal";               |   wizard = balrog;
  }                                  |   return wizard();
}                                    | }
```
#### Object Decorator Pattern

```js                                       | var amy = carlike({}, 1);
var carlike = function(obj, loc){           | amy.move();
  obj.loc = loc;                            |
  obj.move = function(){                    | var ben = carlike({}, 9);
    obj.loc++;                              | ben.loc++;
  }                                         |
  return obj;                               |
}
```

#### Functional Classes
- Building functions within code execution rather than at program load time

```js
function funcName(a, b) {
 // this is declared function
 // this function is loaded in memory when the program/code is run and held there until we use it
}

var funcName = function(a, b) {
   // ^ the function keyword will now assign the following function to the variable
   // loads only when the program reaches the line of code
   // Note the semicolon, it assigns the entire function to a variable
};
```

## Hoisting
##### First, memory is set aside for all necessary variables and `declared functions`.

```js
functions getMysterNumber () { |    // loads like this
  function chooseMystery() {   |  function getMysterNumber() {
    return 12;                 |   function chooseMystery() {
  }                            |     return 12;
                               |   }
  return getMysterNumber();    |   function chooseMystery() {  // it replaced the above chooseMystery function
                               |     return 7;
  function chooseMystery() {   |   }
    return 7;                  |   return getMysterNumber(); // output: 7;
  }                            |
}                              |   }
```

##### Function Expressions are never hoisted! They are treated as assignments.

```js                                 | // loads look like
function getMysteryNumber() {         | function getMysteryNumber() {
  var chooseMystery = function(){     |   var chooseMystery = undefined;
    return 12;                        |   var chooseMystery = undefined;  // replace the above 'chooseMystery'
  }                                   |   chooseMystery = function() {
  return chooseMystery();             |     return 12;
                                      |   }
  var chooseMystery = function() {    |    return chooseMystery();        // return 12;
    return 7;                         |   chooseMystery = function() {  | // this section is unreachable
  }                                   |     return 7;                   | // because it is below return statement
}                                     |   }                             |
                                        }
```

##### Check if 'return' statement is at the top
```js                                 | // loads look like
function getMysteryNumber() {         | function getMysteryNumber() {
  return chooseMystery();             |
  var chooseMystery = function(){     |   var chooseMystery = undefined;
    return 12;                        |   var chooseMystery = undefined;  // replace the above 'chooseMystery'
  }                                   |   return chooseMystery();         // ERROR
                                      |
                                      |   chooseMystery = function() {  | // this section is unreachable
                                      |     return 12;                  | // because it is below return statement
                                      |   }                             |
  var chooseMystery = function() {    |            // return 12;
    return 7;                         |   chooseMystery = function() {  | // this section is also unreachable
  }                                   |     return 7;                   | // because it is below return statement
}                                     |   }                             |
                                        }
```

##### Analyzing Hoisting Load Order

```js
function theBridgeOfHoistingDoom() { | Alrighty, here’s the hoisted version. The function looks for any variables to
  function fellowship() {            | create space for, finds sword, dwarf, fall, and ring, and sets them all to
    return "friends";                | undefined. There’s only one declared function, fellowship, so that comes next.
  }                                  | In this case, there are no replacement declared functions. The executable code
  var sword = "sting";               | that assigns new values or functions to variable has all var keywords popped off.
  var dwarf = function() {           | Any executable code after the first return of sword is excluded from the answer.
    return "axe";                    |
  };                                 | function theBridgeOfHoistingDoom() {
  var fall = "Fly you fools!";       |   var sword = undefined;
  fellowship = function() {          |   var dwarf = undefined;
    return "broken";                 |   var fall = undefined;
  };                                 |   var ring = undefined;
  ring();                            |   function fellowship() {
  return sword;                      |     return "friends";
  fellowship = function() {          |   }
    return "mines"                   |   sword = "sting";
  };                                 |   dwarf = function() {
  sword = function() {               |     return "axe";
    return "glamdring";              |   };
  };                                 |   fall = "Fly you fools!";
  var ring = function() {            |   fellowship = function() {
    return "precious";               |     return "broken";
  };                                 |   };
}                                    |   ring();
                                     |    return sword;
                                     |  }
```
```js
function theBridgeOfHoistingDoom() { |
  var ring = undefined;              | Alrighty, here’s the hoisted version. The function looks for any variables to
  power = undefined;                 | create space for, finds ring and power, and sets them both to undefined. The
  function balrog() {                | order of declared functions is balrog, elf, balrog, wizard, and elf. When older
    return "fire";                   | versions of the loaded functions are replaced, we are left with balrog, wizard,
  }                                  | and then elf. The only executable code that actually ever runs are the lines
  var ring;                          | that precede and include the return of the call to wizard
  function elf() {                   |
    return "pointy ears";            |
  }                                  | function theBridgeOfHoistingDoom() {
  ring = wizard;                     |   var ring = undefined;
  wizard = balrog;                   |   var power = undefined;
  return wizard();                   |   function balrog() {
  function balrog() {                |     return "whip";
    return "whip";                   |   }
  }                                  |   function wizard() {
  function wizard() {                |     return "white";
    return "white";                  |   }
  }                                  |   function elf() {
  var power = ring();                |     return "immortal";
  return elf();                      |   }
  function elf() {                   |   ring = wizard;
    return "immortal";               |   wizard = balrog;
  }                                  |   return wizard();
}                                    | }
```

###### Analyzing load order II

1. For all variable declarations, put the corresponding declarations at the top of the function. Assign them a value
   of undefined and maintain their order.

2. Now that variable declarations have been placed at the top, remove the original declarations, but leave any associated assignments.

3. Then, hoist all function declarations to immediately after your variable declarations, maintaining their order as well.

4. Any function expression assignment is treated here as executable code, and does not change the load order.

5. Remove any unreachable statements after the first return statement.


```js
function theBridgeOfHoistingDoom() { | Alrighty, here’s the hoisted version. The function looks for any variables to
  function fellowship() {            | create space for, finds sword, dwarf, fall, and ring, and sets them all to
    return "friends";                | undefined. There’s only one declared function, fellowship, so that comes next.
  }                                  | In this case, there are no replacement declared functions. The executable code
  var sword = "sting";               | that assigns new values or functions to variable has all var keywords popped off.
  var dwarf = function() {           | Any executable code after the first return of sword is excluded from the answer.
    return "axe";                    |
  };                                 | function theBridgeOfHoistingDoom() {
  var fall = "Fly you fools!";       |   var sword = undefined;
  fellowship = function() {          |   var dwarf = undefined;
    return "broken";                 |   var fall = undefined;
  };                                 |   var ring = undefined;
  ring();                            |   function fellowship() {
  return sword;                      |     return "friends";
  fellowship = function() {          |   }
    return "mines"                   |   sword = "sting";
  };                                 |   dwarf = function() {
  sword = function() {               |     return "axe";
    return "glamdring";              |   };
  };                                 |   fall = "Fly you fools!";
  var ring = function() {            |   fellowship = function() {
    return "precious";               |     return "broken";
  };                                 |   };
}                                    |   ring();
                                     |    return sword;
                                     |  }

```

###### Analyze Load Order III
Q. What is the output of theBridgeOfHoistingDoom()?

1. If the result is undefined, log an "undefined" string to the console.
2. If the function is unable to complete, log an "ERROR" string to the console.

```js
function theBridgeOfHoistingDoom() {   | Answer: console.log("ERROR");
  var sword = undefined;               | // cause ring() is not a function, it's a string.
  var dwarf = undefined;               |
  var fall = undefined;                |
  var ring = undefined;                |
  function fellowship() {              |
    return "friends";                  |
  }                                    |
  sword = "sting";                     |
  dwarf = function() {                 |
    return "axe";                      |
  }                                    |
  fall = "Fly you fools!";             |
  fellowship = function() {            |
    return "broken";                   |
  }                                    |
  ring();                              |
  return sword;                        |
}                                      |
```                                    |