# Monadic sugar syntax

A proposal to use sugar syntax for monadic control flow

## Motivation

Error handling is hard, especially when it's not statically enforcable. Exceptions (`throw`) are one of such examples.
The only way to know if how specific function or class signals about error is to look at the docs.
E.g. [`WebSocket` constructor](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/WebSocket) will throw if port is blocked,
but everything else will go to `onerror` handler. 
Functional languages such as Haskell, OCaml or F# use monads as `Either` or `Result` to solve this problem.
But using straightwaway monads without prior knowledge, pipeline operator and user defined operators is also hard.
On the other hand we already with monad-ish syntax, e.g. `async/await`, null-conditional operator.
This proposal tries to to bring good ideas from monads without all functional burden.

## Description

WIP, don't look, please

```typescript

type Result<TValue = undefined, TError = Error> =
  | { success: false, error: TError }
  | TValue extends undefined
    ? { success: true }
    : { success: true, value: TValue }

interface MonadicSugar {
  unit<T>(): T
  bind<T>(container: T): T
}

// Monadic sugar class should implement a certain interface
class TryResult implements MonadicSugar {
  unit<TValue, TError = Error>(): Result<TValue, TError> {
    return { success: true, value }
  }

  static bind<TValue, TError = Error>(result: Result<TValue, TError>): Result<TValue, TError> {
    return result.success ? "continue" : "throw";
  }
}

fuction ok<TValue, TError = Error>(value: TValue): Result<TValue, TError> {
  return { success: true, value }
}
  
fuction fail<TValue, TError = Error>(error: TError): Result<TValue, TError> {
  return { success: false, error }
}

function createWebSocket(url: string): Result<WebSocket> {
  try {
    return ok(new WebSocket(url))
  } catch (error) {
    return fail(error)
  }
}

const result = TryResult {
  // exclamation mark denotes a special syntax
  // if createWebSocket return failure, remainder of the body won't be executed
  // just like with exception, but only inside the defined scope
  const! websocket = createWebSocket("ws://example.com")
  const readyState = weboscket.readyState
  console.log('ready state is', readyState)
}

// can be transpiled roughly to
const result = (function {
  const tryResult = new TryResult()
  const websocketResult = createWebSocket("ws://example.com")
  if (tryResult.bind(websocketResult) === "throw") {
    // returning failed result
    return websocketResult
  }
    
  const websocket = websocketResult.value
  const readyState = weboscket.readyState
  console.log('ready state is', readyState)
  
  // implicitly returning successful result
  return websocketResult
})()

```

## Prior art

* [Haskell do notation](https://en.wikibooks.org/wiki/Haskell/do_notation)
* [F# computational expressions](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/computation-expressions)
* [OCaml 4.08 binding operators](https://github.com/ocaml/ocaml/pull/1947)

