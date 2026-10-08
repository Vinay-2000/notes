# JavaScript Interview Notes

## ES6+

-   `var` is function-scoped; `let`/`const` are block-scoped.
-   `let` can be reassigned; `const` cannot.
-   `let`/`const` are hoisted but have a Temporal Dead Zone.
-   Destructuring extracts object properties/array positions.
-   Spread expands; rest collects.
-   `?.` safely accesses nullish values.
-   `??` falls back only for `null`/`undefined`; `||` treats all falsy
    values as fallback.

``` js
const user = { id: 1, name: "Vinay" };
const { id, name } = user;

const copy = { ...user, name: "John" };
const city = user?.address?.city;
const pageSize = config.pageSize ?? 20;
```

## Equality

Prefer `===`.

``` js
5 == "5";  // true
5 === "5"; // false
```

Falsy: `false`, `0`, `-0`, `""`, `null`, `undefined`, `NaN`.

## Array methods

-   `map` → transform
-   `filter` → select
-   `reduce` → accumulate
-   `find` → first match
-   `some` → at least one
-   `every` → all

Java Streams have similar operations, but normal JS array methods
execute immediately while Java Streams are lazy.

## Objects/references

``` js
const a = { name: "A" };
const b = a;
b.name = "B";
console.log(a.name); // B
```

Spread is shallow:

``` js
const copy = { ...user };
```

For supported data, `structuredClone(user)` creates a deep clone.

## Callbacks

A callback is a function passed to another function. It is not
inherently asynchronous.

``` js
setTimeout(() => console.log("later"), 1000);
```

Callback-heavy nesting led to Promises and `async/await`.

## Promises

States: `pending`, `fulfilled`, `rejected`. First settlement wins.

``` js
fetchUser()
  .then(user => fetchOrders(user.id))
  .then(orders => console.log(orders))
  .catch(error => console.error(error));
```

Returning from `.then()` passes the value to the next `.then()`.
Throwing rejects the returned Promise.

## async/await

``` js
async function load() {
  try {
    const response = await fetch("/api/users");
    if (!response.ok) throw new Error("Request failed");
    return await response.json();
  } catch (e) {
    console.error(e);
  }
}
```

`await` pauses the current async function, not the entire JavaScript
thread. `async` functions always return Promises.

### Parallel requests

``` js
const [users, orders] = await Promise.all([
  fetchUsers(),
  fetchOrders()
]);
```

## Promise combinators

``` text
all        → all fulfill or reject
allSettled → wait for everyone
any        → first fulfilled
race       → first settled
```

## Event loop

Simplified order:

``` text
sync code → microtasks → next task → microtasks → ...
```

Promises and `await` continuations use microtasks; `setTimeout` uses a
task.

``` js
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");

// A D C B
```

## Closures

A closure is a function plus access to its lexical environment.

``` js
function createCounter() {
  let count = 0;
  return () => ++count;
}

const counter = createCounter();
counter(); // 1
counter(); // 2
```

Each call to `createCounter()` creates an independent closure.

## `this`

Regular function `this` depends on the call site. Arrow functions
capture lexical `this`.

``` js
const user = {
  name: "Vinay",
  greet() {
    setTimeout(() => console.log(this.name), 1000);
  }
};
```

-   `call` → invoke now, individual args
-   `apply` → invoke now, array args
-   `bind` → return a new bound function

## Events

-   Bubbling: child → parent
-   Capturing: parent → child
-   `stopPropagation()` stops propagation
-   Event delegation uses a parent listener plus bubbling.

## Debounce vs throttle

Debounce waits until events stop; useful for search.

``` js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

Throttle limits execution frequency; useful for scroll/resize.

``` js
function throttle(fn, interval) {
  let last = 0;
  return function (...args) {
    const now = Date.now();
    if (now - last >= interval) {
      last = now;
      fn.apply(this, args);
    }
  };
}
```

## Modules

Named exports can be multiple; default export is at most one.

``` js
export const add = () => {};
import { add } from "./math";
```

``` js
export default User;
import User from "./User";
```

## Search race conditions

Debouncing prevents unnecessary requests. Cancellation prevents stale
in-flight responses from overwriting newer results.

``` js
let controller;

async function search(query) {
  controller?.abort();
  controller = new AbortController();

  const response = await fetch(
    `/api/search?q=${query}`,
    { signal: controller.signal }
  );

  return response.json();
}
```
