# Regular Event Loop

This shows the execution order given JavaScript's Call Stack, Event Loop, and
any asynchronous APIs provided in the JS execution environment (in this example;
Web APIs in a Browser environment)

---

Given the code

```javascript
setTimeout(() => { 
  console.log('hi')
}, 1000)           
```

The Call Stack, Event Loop, and Web APIs have the following relationship

```text
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  |                   |              | |               |
    console.log('hi') |                   |              | |               |
  }, 1000)            |                   |              | |               |
                      |                   |              | |               |
```
To start, everything is empty

---

```text
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | <global>          |              | |               |
    console.log('hi') |                   |              | |               |
  }, 1000)            |                   |              | |               |
                      |                   |              | |               |
```
It starts executing the code, and pushes that fact onto the Call Stack (here named
`<global>`)

---

```text
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
> setTimeout(() => {  | <global>          |              | |               |
    console.log('hi') | setTimeout        |              | |               |
  }, 1000)            |                   |              | |               |
                      |                   |              | |               |
```
Then the first line is executed. This pushes the function execution as the
second item onto the call stack.

Note that the Call Stack is a _stack_; The last item pushed on is the first
item popped off. Aka: Last In, First Out. (think; a stack of dishes)

---

```text
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
> setTimeout(() => {  | <global>          |              | | timeout, 1000 |
    console.log('hi') | setTimeout        |              | |               |
  }, 1000)            |                   |              | |               |
                      |                   |              | |               |
```
Executing `setTimeout` actually calls out to code that is _not_ part of JS.
It's part of a _Web API_ which the browser provides for us.
There are a different set of APIs like this available in node.

---

```text
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | <global>          |              | | timeout, 1000 |
    console.log('hi') |                   |              | |               |
  }, 1000)            |                   |              | |               |
                      |                   |              | |               |
```
`setTimeout` is then finished executing; it has offloaded its work to the Web
API which will wait for the requested amount of time (1000ms).

---

```text
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  |                   |              | | timeout, 1000 |
    console.log('hi') |                   |              | |               |
  }, 1000)            |                   |              | |               |
                      |                   |              | |               |
```

As there are no more lines of JS to execute, the Call Stack is now empty.

---

```text
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  |                   | function   <-----timeout, 1000 |
    console.log('hi') |                   |              | |               |
  }, 1000)            |                   |              | |               |
                      |                   |              | |               |
```
Once the timeout has expired, the Web API lets JS know by adding code to the
Event Loop.

It doesn't push onto the Call Stack directly as that could intefere with already
executing code, and you'd end up in weird situations.

The Event Loop is a _Queue_. The first item pushed on is the first
item popped off. Aka: First In, First Out. (think; a queue for a movie)

---

```text
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | function        <---function     | |               |
    console.log('hi') |                   |              | |               |
  }, 1000)            |                   |              | |               |
                      |                   |              | |               |
```
Whenever the Call Stack is empty, the JS execution environment occasionally checks
to see if anything is Queued in the Event Loop. If it is, the first item is moved
to the Call Stack for execution.

---

```text
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | function          |              | |               |
>   console.log('hi') | console.log       |              | |               |
  }, 1000)            |                   |              | |               |
                      |                   |              | |               |
```
Executing the function results in `console.log` being called, also pushed onto
the Call Stack.

---

```text
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | function          |              | |               |
    console.log('hi') |                   |              | |               |
  }, 1000)            |                   |              | |               |
                      |                   |              | |               |
> hi
```
Once finished executing, `hi` is printed, and `console.log` is removed from the
Call Stack.

---

```text
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  |                   |              | |               |
    console.log('hi') |                   |              | |               |
  }, 1000)            |                   |              | |               |
                      |                   |              | |               |
> hi
```
Finally, the function has no other commands to execute, so it too is taken off
the Call Stack.

Our program has now finished execution.

End.

# Starved Event Loop

Below is an example of how code running in the current Call Stack can prevent code on the Event Loop from being executed. aka; the Event Loop is _starved_.

---

Given the code

setTimeout(() => { 
  console.log('bye')
}, 2)           
someSlowFn()
console.log('hi')

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  |                   |              | |               |
    console.log('bye')|                   |              | |               |
  }, 2)               |                   |              | |               |
  someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |
```

To start, everything is empty

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | <global>          |              | |               |
    console.log('bye')|                   |              | |               |
  }, 2)               |                   |              | |               |
  someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |
```

It starts executing the code, and pushes that fact onto the Call Stack (here named `<global>`)

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
> setTimeout(() => {  | <global>          |              | |               |
    console.log('bye')| setTimeout        |              | |               |
  }, 2)               |                   |              | |               |
  someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |
```

`setTimeout` is pushed onto the Call Stack

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
> setTimeout(() => {  | <global>          |              | | timeout, 2    |
    console.log('bye')| setTimeout        |              | |               |
  }, 2)               |                   |              | |               |
  someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |
```

`setTimeout` triggers the timeout Web API

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | <global>          |              | | timeout, 2    |
    console.log('bye')|                   |              | |               |
  }, 2)               |                   |              | |               |
  someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |
```

`setTimeout` is then finished executing, while the Web API waits for the requested amount of time (2ms).

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | <global>          |              | | timeout, 2    |
    console.log('bye')| someSlowFn        |              | |               |
  }, 2)               |                   |              | |               |
> someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |
```

`someSlowFn` starts executing. Let's pretend this takes around 300ms to complete. For that 300ms, JS can't remove it from the Call Stack

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | <global>          | function   <-----timeout, 2    |
    console.log('bye')| someSlowFn        |              | |               |
  }, 2)               |                   |              | |               |
> someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |
```

Meanwhile, the timeout has expired, so the Web API lets JS know by adding code to the Event Loop.

`someSlowFn` is still executing on the Call Stack, and cannot be interrupted, so the code to be executed by the timeout waits on the Event Loop for its turn.

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | <global>          | function     | |               |
    console.log('bye')| someSlowFn        |              | |               |
  }, 2)               |                   |              | |               |
> someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |
```

Still waiting for `someSlowFn` to finish...

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | <global>          | function     | |               |
    console.log('bye')|                   |              | |               |
  }, 2)               |                   |              | |               |
> someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |
```

`someSlowFn` finally finished!

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | <global>          | function     | |               |
    console.log('bye')| console.log       |              | |               |
  }, 2)               |                   |              | |               |
  someSlowFn()        |                   |              | |               |
> console.log('hi')   |                   |              | |               |
```

The next line is executed, pushing `console.log` onto the Call Stack

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | <global>          | function     | |               |
    console.log('bye')|                   |              | |               |
  }, 2)               |                   |              | |               |
  someSlowFn()        |                   |              | |               |
> console.log('hi')   |                   |              | |               |

> hi
```

We see `hi` output on the console thanks to `console.log`

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  |                   | function     | |               |
    console.log('bye')|                   |              | |               |
  }, 2)               |                   |              | |               |
  someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |

> hi
```

Nothing left to execute, so the special `<global>` is popped off the Call Stack.

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | function        <---function     | |               |
    console.log('bye')|                   |              | |               |
  }, 2)               |                   |              | |               |
  someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |

> hi
```

This frees up the JS execution environment to check the Event Loop for any code which needs to be executed.

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | function          |              | |               |
>   console.log('bye')| console.log       |              | |               |
  }, 2)               |                   |              | |               |
  someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |

> hi
```

Executing the function results in `console.log` being called, also pushed onto the Call Stack.

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  | function          |              | |               |
    console.log('bye')|                   |              | |               |
  }, 2)               |                   |              | |               |
  someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |

> hi
> bye
```

Once finished executing, `bye` is printed, and `console.log` is removed from the Call Stack.

Notice that by this point, it is at least 300ms _after_ the code originally requested the `setTimeout`. Meaning even though we asked for it to be executed after only 2ms, we still had to wait for the Call Stack to empty before the `setTimeout` code on the Event Loop could be executed

_Note: Even if we didn't have `someSlowFn`, `setTimeout` is clamped to 4ms as the mimimum delay allowed in some cases_

---

```
        [code]        |   [call stack]    | [Event Loop] | |   [Web APIs]  |
  --------------------|-------------------|--------------| |---------------|
  setTimeout(() => {  |                   |              | |               |
    console.log('bye')|                   |              | |               |
  }, 2)               |                   |              | |               |
  someSlowFn()        |                   |              | |               |
  console.log('hi')   |                   |              | |               |

> hi
> bye
```

Finally, there are no other commands to execute, so it too is taken off the Call Stack.

Our program has now finished execution.

End.

_Note: It's also possible to [starve the event loop with Promises](https://gist.github.com/jesstelford/bbb30b983bddaa6e5fef2eb867d37678) via the "Microtask queue"_

GEC (it has 2 main functions create a global object, and attach the `this` value to the global object)
FEC (an execution context is created for every function invocation), when a function is called, its executed through the execution stack

GEC is initially pushed on the execution stack by default -> global object
FEC is pushed to the stack when a function is invoked
```js
function greeting() { 
   sayHi();
}

function sayHi() {
   return "Hi!";
}

// Invoke the `greeting` function
greeting();
```
![[Pasted image 20220802092618.png]]

the execution context is created in 2 stages
creation stage and execution stage


# Creation stage
a lexical environment is created in the creation stage of the execution context, it is of 2 types
lexical env: this is the primary lexical env
variable env: this is a lexical env used explicitly by the var 

```
ExecutionContext = {

2  LexicalEnvironment = <ref. to LexicalEnvironment in memory>,

3  VariableEnvironment = <ref. to VariableEnvironment in memory>  

4}
```

  
According to the [official ES6](http://ecma-international.org/ecma-262/6.0/) documentation, _A Lexical Environment is a specification type used to define the association of Identifiers to specific variables and functions based upon the lexical nesting structure of ECMAScript code. A Lexical Environment consists of an Environment Record and a possibly null reference to an outer Lexical Environment._

The lexical environment is a standard or specification type. It defines the identifier that represents a specific variable or function, based on its lexical nesting structure or scope. The lexical environment is simply a structure that contains variable identifier mapping.


   
According to the [official ES6](http://ecma-international.org/ecma-262/6.0/) documentation, _A Lexical Environment is a specification type used to define the association of Identifiers to specific variables and functions based upon the lexical nesting structure of ECMAScript code. A Lexical Environment consists of an Environment Record and a possibly null reference to an outer Lexical Environment._

The lexical environment is a standard or specification type. It defines the identifier that represents a specific variable or function, based on its lexical nesting structure or scope. The lexical environment is simply a structure that contains variable identifier mapping.

  
According to the official ES6 docs: _An [Environment Record](https://262.ecma-international.org/6.0/#sec-environment-records) records the identifier bindings created within the scope of its associated lexical environment._ The Environment Record is the location where variable and function declarations are stored in a lexical environment, and it is located in the memory heap of the JS engine.

The Environment Record is of two types:

-   Declarative Environment Record: This is mainly used by a Lexical Environment created in the Function Execution Context. It records (store) function declarations and variable declarations. The Declarative Environment Record also stores an `arguments` object, which contains the length(number) of argument(s) passed to a function, and a map of the argument(s) and its index.
-   Object Environment Record: This is mainly used by a Lexical Environment created in the Global Execution Context. It stores function declarations, variable declarations, and a global binding object(window object in browsers).

  
Every (inner) Lexical Environment reference an outer Lexical Environment. The outer Lexical Environment logically surrounds the inner Lexical Environment that it is referenced by. The JS engine can look for variables in the outer Lexical Environment if they are not found in the inner Lexical Environment. The Lexical Environment in the Global Execution Context does not reference an outer Lexical Environment but is referenced to `null`.

```js
const company = {

2  name: 'facebook',

3  yearFound: 2004,

4  getAge: function() {

5    const date = new Date();

6    let currentYear = date.getFullYear();

7    console.log(currentYear - this.yearFound);

8  }

9}

10company.getAge();

11

12const companyAge = company.getAge;

13companyAge();
```


```
GlobalExectionContext = {

2  LexicalEnvironment: {

3    EnvironmentRecord: {

4      Type: "Object",

5      // Identifier bindings go here

6    }

7    outerEnv: <null>,

8    this: <global object>

9  }

10}

11

12FunctionExectionContext = {

13  LexicalEnvironment: {

14    EnvironmentRecord: {

15      Type: "Declarative",

16      // Identifier bindings go here

17    }

18    outerEnv: <Global or outer function environment reference>,

19    this: <depends on how function is called>

20  }

21}
```

## Variable Environment

The Variable Environment has been explained earlier in this section, as a type of Lexical Environment used specifically by the `var` _VariableStatement_ within an Execution Context. This means that the Environment Record of the **VariableEnvironment** only records bindings created by the `var` _VariableStatement_. Whereas the Environment Record of the **LexicalEnvironment** records both variable (`let` and `const`) bindings and function declarations.

**EXECUTION STAGE** The stage variables are assigned to their respective values, and the code is executed.

```js
const b = 20;

3var c;

4

5function add(e, f) {

6   var d = 30;

7   return e + f + d;

8}

9

10c = add(40, 50);
```

### Creation stage

```
GlobalExectionContext = {

2  LexicalEnvironment: {

3    EnvironmentRecord: {

4      Type: "Object",

5      // Identifier bindings go here

6      a: < uninitialized >,

7      b: < uninitialized >,

8      add: < func >

9    }

10    outerEnv: <null>,

11    ThisBinding: <Global Object>

12  },

13  VariableEnvironment: {

14    EnvironmentRecord: {

15      Type: "Object",

16      // Identifier bindings go here

17      c: undefined,

18    }

19    outerEnv: <null>,

20    ThisBinding: <Global Object>

21  }

22}
```

  
When the add(40, 50) function is called, a new Function Execution Context is created to execute the function code. The Function Execution Execution Context will look like this in the creation stage.


```
FunctionExectionContext = {

2LexicalEnvironment: {

3    EnvironmentRecord: {

4      Type: "Declarative",

5      // Identifier bindings go here

6      Arguments: {0: 40, 1: 50, length: 2},

7    },

8    outerEnv: <GlobalLexicalEnvironment>,

9    ThisBinding: <Global Object or undefined>,

10  },

11VariableEnvironment: {

12    EnvironmentRecord: {

13      Type: "Declarative",

14      // Identifier bindings go here

15      d: undefined

16    },

17    outerEnv: <GlobalLexicalEnvironment>,

18    ThisBinding: <Global Object or undefined>

19  }

20}
```


### Execution Stage

Here values are assigned to variables.

Global Code: In the Execution Stage, when variable assignments are done. This is what the Global Execution Context will look like.


```
GlobalExectionContext = {

2LexicalEnvironment: {

3    EnvironmentRecord: {

4      Type: "Object",

5      // Identifier bindings go here

6      a: 10,

7      b: 20,

8      add: < func >

9    }

10    outerEnv: <null>,

11    ThisBinding: <Global Object>

12  },

13VariableEnvironment: {

14    EnvironmentRecord: {

15      Type: "Object",

16      // Identifier bindings go here

17      d: undefined,

18    }

19    outerEnv: <null>,

20    ThisBinding: <Global Object>

21  }

22}
```


Function Code: After the Global Execution Context has gone through the execution stage, assignments to variables are done. This also includes variables inside every function declaration in the Global Execution Context. The Function Execution Context will look like this during the execution stage.

```
FunctionExectionContext = {

2LexicalEnvironment: {

3    EnvironmentRecord: {

4      Type: "Declarative",

5      // Identifier bindings go here

6      Arguments: {0: 40, 1: 50, length: 2},

7    },

8    outerEnv: <GlobalLexicalEnvironment>,

9    ThisBinding: <Global Object or undefined>,

10  },

11VariableEnvironment: {

12    EnvironmentRecord: {

13      Type: "Declarative",

14      // Identifier bindings go here

15      d: 30

16    },

17    outerEnv: <GlobalLexicalEnvironment>,

18    ThisBinding: <Global Object or undefined>

19  }

20}
```