the creation of the Global execution context
prominent

how the global execution context work

	theres a variable -> store it in the global memory
	function declaration -> store it in the global memory too

global memory == global scope == global variable env

js engine needs to keep track of whats happing -> relies on the call stack for that

call stack means pop and push 

when you run a function -> the engine power pushes that function into the call stack


first, the function appears in the global execution context, another mini context appears alongside the function, that little box call local execution context

