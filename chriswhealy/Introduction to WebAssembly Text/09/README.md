# Introduction to WebAssembly Text
| Previous | | Next
|---|---|---
| [Loops](../08/README.md) | [Top](../README.md) | [WASM and Shared Memory](../10/README.md)

## 9: More About Functions

So far, we have only looked at a very simple WebAssembly function &mdash; one that takes no arguments and returns a single `i32` value.

> For the sake of simplicity, we will skip over certain details of function declarations such as interface types, as these are not required at the moment.

### Naming Functions

A function is declared using the keyword `func`.

> ***IMPORTANT***  
>WebAssembly functions can be given two, possibly different, human-readable names:[^1]
>
> 1. A name used when calling the function from **inside** the WebAssembly module
> 1. A name used when calling the function from **outside** the WebAssembly module

If we wish to call a WebAssembly function from inside the module, then the internal name is declared as follows:

```wat
(func $my_func       ;; The function's internal name

  ;; Function body goes here
)
```

If a WebAssembly function only needs to be called from outside the module, then the internal name can be omitted.  The external name is defined using the `export` keyword followed by some string.  This string then becomes the function's external (or exported name):

```wat
(func
  (export "wasm_fn") ;; The function's external name

  ;; Function body goes here
)
```

if we need to call the WebAssembly module from both inside and outside the module, then specify both names:

```wat
(func $my_func       ;; The function's internal name
  (export "wasm_fn") ;; The function's external name

  ;; Function body goes here
)
```

### Function Arguments and Return Values

If a WebAssembly function needs to be passed any arguments, these must be specified immediately after the function name(s).  Similarly, if a function returns any values, these must be specified after any arguments have been defined.

Here's a real-life example.  Let's say we have a function that finds the magnitude (or hypotenuse length) of a complex number; that is, the distance from origin to the point on the complex plane.

This function takes a complex number as an argument in the form of two, 64-bit floating point numbers (`$real` and `$imag`) and returns a single 64-bit floating point number:

```wat
(func $mag           ;; Internal name
  (export "mag")     ;; External name
  (param $real f64)  ;; 1st argument is an f64 known as $real
  (param $imag f64)  ;; 2nd argument is an f64 known as $imag
  (result f64)       ;; One f64 will be left on the stack when the function terminates

  ;; Function body goes here
)
```

The implementation of this function simply uses Pythagoras' formula to work out the length of the hypotenuse of the triangle having sides `$real` and `$imag`

[09-single-return-value.wat](09-single-return-value.wat)
```wat
(func $mag           ;; Internal name
  (export "mag")     ;; External name
  (param $real f64)  ;; 1st argument is an f64 known as $real
  (param $imag f64)  ;; 2nd argument is an f64 known as $imag
  (result f64)       ;; One f64 will be left on the stack when the function terminates

  ;; Find the square root of the top value on the stack, then push the result
  ;; back onto the stack
  (f64.sqrt
    ;; Pop the top two values off the stack, add them up and push the result back
    ;; onto the stack
    (f64.add
      ;; Square the real part and push the result onto the stack
      (f64.mul (local.get $real) (local.get $real))

      ;; Square the imaginary part and push the result onto the stack
      (f64.mul (local.get $imag) (local.get $imag))
    )
  )
  
  ;; The square root operation leaves a single f64 value on the stack
  ;; We now exit and this becomes the function's return value
)
```

After the `f64.sqrt` instruction has been executed, it pushes its result onto the stack and this implicitly becomes the function's return value.

You can run this function using `wasmer`:

```bash
wasmer 09-single-return-value.wat -i mag 3 4
5
```

### Functions with Multiple Return Values

WebAssembly functions that return multiple values can be invoked from JavaScript running in the browser or via `wasmer`

Here's a simple example in which calculate the conjugate of complex number.  This is a very simple operation that transforms a complex number `a + bi` into `a - bi`.

The WebAssembly function must be passed a complex number in the form of two, 64-bit floating point numbers, and it returns another complex number, also in the form of two, 64-bit floating point numbers.

[`09-multiple-return-values.wat`](09-multiple-return-values.wat)
```wat
;; Conjugate of a complex number
;; conj(a+bi) => (a-bi)
(func $conj                     ;; Internal name
  (export "conj")               ;; External name
  (param $a f64)                ;; 1st argument is an f64 known as $a
  (param $b f64)                ;; 2nd argument is an f64 known as $b
  (result f64 f64)              ;; Two f64s will be left on the stack

  (local.get $a)                ;; Push $a. Stack = [$a]
  (f64.neg (local.get $b))      ;; Push $b then negate its value.  Stack = [-$b, $a]
)
```

The host environment then pops these two values off the stack to obtain the result of the function call

You can test this by running using `wasmer` to run [`09-multiple-return-values.wat`](09-multiple-return-values.wat)

```bash
wasmer 09-multiple-return-values.wat -i conj -- -5 3
-5 -3
```

> Notice the double hyphens `--` between `-i conj` and the function arguments.
> This is necessary to prevent the shell from interpreting the minus sign in front of `-5` an shell option

### NodeJS Limitation

At the time of writing (Nov 2021), NodeJS cannot yet invoke WebAssembly functions that return multiple values.  NodeJS currently throws a runtime error if you attempt to instantiate a WebAssembly module that exports a function with multiple return values.

Just to demonstrate this problem, try running [`09-multiple-return-values.js`](09-multiple-return-values.js) from NodeJs.

```bash
node 09-multiple-return-values.js
(node:49384) UnhandledPromiseRejectionWarning: CompileError: WebAssembly.instantiate(): return count of 2 exceeds internal limit of 1 @+15
(Use `node --trace-warnings ...` to show where the warning was created)
```

However, the exact same `.wasm` file can be executed successfully both by `wasmer` and from within a browser.

[^1]: All functions (and local variables) are referenced by their index number, so you *could* choose not to use any human-readable function names; however, its now down to you to remember what function number `2` or `7` or `21` does.  Good luck with that one...