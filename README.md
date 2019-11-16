# Synchronized function syntax

This proposal suggest to create a new kind of syntax and semantic of asynchronous functions, socalled synchronized function. It's syntax is opposite to async functions, it awaits promises automatically without using `await` keyword.

## Rationale

Async functions were created to reduce nesting and simplify code. Currently async functions are using to make call behav like a synchronous function and it's a very rare case when developers need to make async function not to wait a promise resolution received in some. Thus async functions are flooded by `await` usage.

Current async/await syntax has bunch of issues:

1. It couldn't solve chained promises and lead to weird expressions like this `await (await fetch(url)).json()`.
2. It makes harder to implement other language features like piped calls with things like this `x |> await fetch().then((res) => res.json())`.
3. It requires custom solutions for other language feauters like async for-loop.

Synchronized function solves this issues and reduces syntax complexity. It resolves all promises, even intermediate promises from chained calls. It's still asynchronous under the hood and return a promise like regular async function do, so it isn't a blocking too. In the case when developers still need to move a call to another event-frame, they can use `setImmediate/setTimeout`.

## Syntax

### Regular function

Synchromized functions requires `synced` keyword placed before the function.

Example:

```
synced function get(url) {
  return fetch(url).json()
}
```

### Arrow function

It seems logical to make this behavior the default due to its simplicity and to use `~>` construction for synced function definition of arrow function (instead of a keyword).

Example:
```
const get = (url) ~> fetch(url).json()
```

## Examples

## Chained call

Regular async function:
```js
async function requestJson(url) {
  const res = await fetch(url)
  const json = await res.json()
  return json
}
```

Synchronized function:
```js
synced function requestJson(url) {
  return fetch(url).json()
}
```

## Parallel call

Regular async function:
```js
async function requestJson(urls) {
  const responses = await Promise.all([
    fetch(urls[0]),
    fetch(urls[1]),
  ])

  return responses
}
```

Synchronized function:
```js
synced function requestJson(urls) {
  const responses = Promise.parallel(
    () => fetch(urls[0]),
    () => fetch(urls[1]),
  )
  
  return responses
}
```

Promise.parallel implementation:
```js
Promise.parallel = (...fs) => Promise.all(fs.map(f => f()))
```
