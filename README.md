# Synchronized function syntax

This proposal suggests creation of a new kind of syntax and semantic for asynchronous functions, socalled synchronized function. It's logic is opposite to async functions, it awaits promises automatically without using `await` keyword and use `nowait` to prevent call from resolution.

## Rationale

Async functions were created to reduce nesting and simplify code. Currently async functions are using to make call behav like a synchronous function and it's a very rare/specific case when developers need to make async function not to wait a promise resolution in current call. Thus async functions are flooded by `await` statements.

Current async/await syntax has bunch of issues:

1. It couldn't solve chained promises and lead to weird expressions like this `await (await fetch(url)).json()`.
2. It makes harder to implement other language features like pipeline operator which couldn't use asyncronous code `x |> async (url) => fetch(url).then((res) => res.json()) |> await`.
3. It requires custom solutions for other language feauters like async for-loop.

Synchronized function solves this issues and reduces syntax complexity. It resolves all promises returend by function call, even from chained calls. It's still asynchronous under the hood and returns a promise like regular async function do, so it isn't blocking too. In the case when developers still need to move a call to another event-frame, they can use `setImmediate/setTimeout` or `nowait` keyword.

## Syntax

### Regular function

Synchromized functions requires `synced` keyword placed before the function. Also it uses `nowait` keyword to prevent a promise resolution.

Example:

```
synced function request(url) {
  return fetch(url).json()
}
```

### Arrow function

It seems logical to make this behavior the default due to its simplicity and to use `~>` construction for synced function definition of arrow function (instead of a keyword).

Example:
```
const get = (url) ~> fetch(url).json()
```

## Resolution

Synced function resolves each function call or constructor call result. It doesn't resolve object properties.

Constructor resolution:
```
synced function example() {
  const result = new Promise((resolve) => setImmediate(resolve, true)) // would be resolved
  result // -> true
}
```

External promise resolution:
```
const promise = new Promise((resolve) => setImmediate(resolve, true))

synced function example() {
  const result = promise.then() // would be resolved
  result // -> true
}
```

## Examples

### Chained call

Regular async function:
```js
async function request(url) {
  const res = await fetch(url)
  const json = await res.json()
  return json
}
```

Synchronized function:
```js
synced function request(url) {
  return fetch(url).json()
}
```

### Nowait and promises

Regular async function:
```js
async function request(url) {
  const promise = fetch(url).then((res) => res.json())
  const json = await promise
}
```

Synchronized function:
```js
synced function request(url) {
  const promise = nowait fetch(url)
  const json = promise.then().json()
}
```

### Parallel resolution

Regular async function:
```js
async function request(urls) {
  const responses = await Promise.all([
    fetch(urls[0]),
    fetch(urls[1]),
  ])

  return responses
}
```

Synchronized function:
```js
synced function request(urls) {
  const responses = Promise.all([
    nowait fetch(urls[0]),
    nowait fetch(urls[1]),
  ])
  
  return responses
}
```

Synchronized function (without `nowait`):
```js
function parallel(fs) {
  return Promise.all(fs.map(f => f()))
}

synced function request(urls) {
  const responses = parallel([
    () => fetch(urls[0]),
    () => fetch(urls[1]),
  ])
  
  return responses
}
```
