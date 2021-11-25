# Introduction to WebAssembly Text
| Previous | | Next
|---|---|---
| [Local Variables](../05/README.md) | [Top](../README.md) | [Conditions](../07/README.md)

## 6: Arrangement of WAT Instructions
WebAssembly is a stack-based language which means that almost every instruction we use will typically pop one or more values off the stack, do something with them, then push the result back onto the stack.

Consequently, if you decompile a `.wasm` program using a tool such as `wasm2wat`, you will see the WAT instructions listed sequentially (that is, in [Reverse Polish Notation](https://en.wikipedia.org/wiki/Reverse_Polish_notation)).

### Sequential WAT Instructions
When presented in sequential order, if we wish add up two numbers, we must first push both values onto the stack *then* call the `add` operation.  The `add` operation then pops the top two values off the stack, adds them up and pushes the result back onto the stack.

```wat
i32.const 3    ;; Push 3.  Stack = [3]
i32.const 5    ;; Push 5.  Stack = [5,3]

i32.add        ;; Pop 2 values, add them and push the result.  Stack = [8]
```

In addition to the fact that we have to issue instructions in this apparently backwards manner, we must also explicitly identify the datatype upon which instructions operates.  In this case, we are working with 32-bit integers; therefore, each instruction is prefixed with `i32`.[^1]

Similar prefixes exist for 64-bit integers (`i64`), and 32- and 64-bit floating points (`f32` and `f64`)

### Folded WAT Instructions or "S-Expressions"

It is also possible to fold WAT instructions into what is known as an [S-Expression](https://en.wikipedia.org/wiki/S-expression).  If you have used any of the LISP family of languages such as [Clojure](https://clojure.org/), then you will have already come across this syntactical style.  The same add instructions shown above could also be written as an S-Expression like this:

```wat
(i32.add (i32.const 3) (i32.const 5))
```

Although the folded (or S-Expression) form of WAT is generally easier to read, you should get used to reading both the sequential and folded forms because tools such as the WebAssembly decompiler (`wasm2wat`) always outputs instructions sequentially.

[^1]: WebAssembly instructions cannot operate values of mixed datatype. The compiler will throw an error for instance if you attempt to use `i32.add` to add two values that are not both of type `i32`