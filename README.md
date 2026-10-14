# Frontend + Full Stack Interview Questions & Answers

A revision handbook covering **JavaScript, React, Node.js, Express.js, HTML, CSS** and other commonly asked topics (Git, REST, Web Security, Performance, Auth, DB basics).

## Table of Contents

1. [JavaScript](#1-javascript)
2. [React](#2-react)
3. [Node.js](#3-nodejs)
4. [Express.js](#4-expressjs)
5. [HTML](#5-html)
6. [CSS](#6-css)
7. [Other Important Topics](#7-other-important-topics)

---

# 1. JavaScript

### 1. Difference between `var`, `let` and `const`

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function | Block | Block |
| Hoisting | Hoisted, initialized as `undefined` | Hoisted, but in Temporal Dead Zone (TDZ) | Hoisted, in TDZ |
| Re-declare in same scope | Yes | No | No |
| Re-assign | Yes | Yes | No |
| Attaches to `window` (global) | Yes | No | No |

```js
console.log(a); // undefined
var a = 1;

console.log(b); // ReferenceError (TDZ)
let b = 2;

const user = { name: "A" };
user.name = "B";   // allowed: object is mutable
user = {};         // TypeError: reassigning the binding
```

`const` prevents **reassignment**, not **mutation**. Use `const` by default, `let` when you must reassign, avoid `var`.

---

### 2. What is hoisting?

Hoisting is JavaScript's behavior of moving **declarations** to the top of their scope during the creation phase of the execution context, before code runs.

- `var` is hoisted and set to `undefined`.
- `let`/`const` are hoisted but not initialized (TDZ), so accessing them early throws a `ReferenceError`.
- Function declarations are fully hoisted (name and body).
- Function expressions and arrow functions follow their variable's rule (`var`/`let`/`const`).

```js
sayHi();            // works
function sayHi() { console.log("hi"); }

sayBye();           // TypeError: sayBye is not a function
var sayBye = function () {};
```

---

### 3. Loose equality (`==`) vs strict equality (`===`)

- `==` compares values **after type coercion**.
- `===` compares **value and type**, with no coercion.

```js
0 == "0";           // true
0 === "0";          // false
null == undefined;  // true
null === undefined; // false
NaN == NaN;         // false (use Number.isNaN)
[] == false;        // true (quirky)
```

Prefer `===` almost always. `Object.is()` is similar but treats `NaN` as equal to `NaN` and distinguishes `+0` from `-0`.

---

### 4. Difference between `null` and `undefined`

| `undefined` | `null` |
|---|---|
| Variable declared but not assigned; missing property; missing function argument | Intentional "no value" assigned by the developer |
| Type: `"undefined"` | Type: `"object"` (a historical bug) |
| Set by the JS engine | Set by the programmer |

```js
let x;           // undefined
let y = null;    // null
typeof x;        // "undefined"
typeof y;        // "object"
```

---

### 5. Primitive vs reference data types

**Primitives** (immutable, stored by value): `string`, `number`, `bigint`, `boolean`, `undefined`, `null`, `symbol`.

**Reference types** (stored by reference): `object`, `array`, `function`, `Map`, `Set`, `Date`, etc.

```js
let a = 10; let b = a; b = 20;
console.log(a); // 10 (copy)

let o1 = { x: 1 }; let o2 = o1; o2.x = 99;
console.log(o1.x); // 99 (same reference)
```

Primitives live on the stack (conceptually), references point to objects on the heap. Comparing two objects with `===` compares references, not contents.

---

### 6. What is a closure and why is it useful?

A closure is a function that **remembers the variables of its lexical scope** even after the outer function has finished executing.

```js
function counter() {
  let count = 0;
  return () => ++count;
}
const inc = counter();
inc(); // 1
inc(); // 2
```

**Uses:** data privacy / encapsulation, function factories, memoization, currying, debounce/throttle, event handlers, module pattern, React hooks internals.

**Caveat:** closures can cause memory leaks if they hold references to large objects that are no longer needed.

Classic gotcha:

```js
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 3 3 3
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 0 1 2
```

---

### 7. `.call()`, `.apply()` and `.bind()`

All three set the value of `this` for a function.

| Method | Invokes immediately? | Arguments |
|---|---|---|
| `call(thisArg, a, b)` | Yes | Comma separated |
| `apply(thisArg, [a, b])` | Yes | Array |
| `bind(thisArg, a, b)` | No, returns a new function | Comma separated (can be partially applied) |

```js
function greet(greeting, punct) {
  return `${greeting}, ${this.name}${punct}`;
}
const user = { name: "Anuj" };

greet.call(user, "Hello", "!");     // "Hello, Anuj!"
greet.apply(user, ["Hello", "!"]);  // "Hello, Anuj!"
const fn = greet.bind(user, "Hi");
fn("?");                            // "Hi, Anuj?"
```

---

### 8. Arrow function vs regular function

| Feature | Regular | Arrow |
|---|---|---|
| `this` | Dynamic (depends on how called) | Lexical (inherited from enclosing scope) |
| `arguments` object | Yes | No (use rest params) |
| Can be a constructor (`new`) | Yes | No |
| Has `prototype` | Yes | No |
| Hoisting | Declarations are hoisted | Not hoisted (they are expressions) |
| Implicit return | No | Yes (`x => x * 2`) |

```js
const obj = {
  name: "A",
  regular() { return this.name; },   // "A"
  arrow: () => this.name,            // undefined (this is outer scope)
};
```

Avoid arrow functions as object methods or prototype methods. Use them for callbacks.

---

### 9. What is prototypal inheritance?

Every JS object has an internal link `[[Prototype]]` (accessible via `Object.getPrototypeOf(obj)` or `__proto__`) to another object. When a property is not found on the object, the engine walks up the **prototype chain** until it finds it or reaches `null`.

```js
const animal = { eats: true };
const dog = Object.create(animal);
dog.barks = true;

dog.eats;  // true (found on prototype)

function Person(name) { this.name = name; }
Person.prototype.hi = function () { return "Hi " + this.name; };
new Person("A").hi();

// ES6 classes are syntactic sugar over the same mechanism
class Student extends Person {}
```

---

### 10. Explain the JavaScript event loop

JavaScript is single-threaded. The event loop lets it handle async work without blocking.

Pieces:
- **Call stack**: where synchronous code runs.
- **Web APIs / Node APIs**: timers, fetch, DOM events (handled outside the stack).
- **Microtask queue**: Promise `.then/.catch/.finally`, `queueMicrotask`, `MutationObserver`.
- **Macrotask (callback) queue**: `setTimeout`, `setInterval`, I/O, UI events.

**Loop:** run the stack until empty, then drain **all** microtasks, then take **one** macrotask, repeat. Microtasks always run before the next macrotask.

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
// Output: 1 4 3 2
```

---

### 11. What are Promises and what states can they be in?

A Promise represents the eventual result of an async operation.

States:
1. **Pending**: initial state.
2. **Fulfilled**: operation succeeded (has a value).
3. **Rejected**: operation failed (has a reason).

Once settled (fulfilled or rejected) it can't change.

```js
const p = new Promise((resolve, reject) => {
  setTimeout(() => resolve("done"), 500);
});
p.then(console.log).catch(console.error).finally(() => console.log("end"));

Promise.all([p1, p2]);        // all fulfilled, or first rejection
Promise.allSettled([p1, p2]); // wait for all, never rejects
Promise.race([p1, p2]);       // first to settle
Promise.any([p1, p2]);        // first to fulfill
```

---

### 12. What is `async/await` and how does it relate to Promises?

`async/await` is syntactic sugar over Promises that makes async code read like synchronous code.

- An `async` function always returns a Promise.
- `await` pauses the function until the Promise settles and unwraps its value (the rest of the program is not blocked).
- Use `try/catch` for errors.

```js
async function getUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error("Failed");
    return await res.json();
  } catch (err) {
    console.error(err);
  }
}

// Run in parallel
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

---

### 13. What are higher-order functions?

Functions that **take a function as an argument** and/or **return a function**.

```js
[1, 2, 3].map(x => x * 2);          // map
[1, 2, 3, 4].filter(x => x % 2);    // filter
[1, 2, 3].reduce((s, x) => s + x, 0); // reduce

const multiplier = n => x => x * n; // returns a function
const double = multiplier(2);
```

Other examples: `forEach`, `sort`, `setTimeout`, `addEventListener`, React HOCs.

---

### 14. What is an IIFE?

**Immediately Invoked Function Expression**: a function that runs right after it is defined.

```js
(function () {
  const secret = "private";
  console.log("runs now");
})();

(() => console.log("arrow IIFE"))();
```

**Uses:** create a private scope (avoid polluting global scope), module pattern, run `async` code at top level in older environments: `(async () => { await x(); })();`

---

### 15. What are `Map` and `Set`, and how do they differ from Array and Object?

**Map**: key-value pairs where keys can be **any type**, keeps insertion order, has `.size`.
**Set**: collection of **unique** values.

| | Object | Map |
|---|---|---|
| Key types | string / symbol | any |
| Order | Mostly insertion (with integer-key quirks) | Insertion order guaranteed |
| Size | `Object.keys(o).length` | `map.size` |
| Iteration | Needs `Object.entries` | Directly iterable |
| Performance for frequent add/remove | Lower | Better |

| | Array | Set |
|---|---|---|
| Duplicates | Allowed | Not allowed |
| Index access | Yes | No |
| Lookup | `includes` O(n) | `has` ~O(1) |

```js
const m = new Map([["a", 1]]);
m.set({ id: 1 }, "obj key");
const s = new Set([1, 2, 2, 3]); // {1, 2, 3}
const unique = [...new Set(arr)];
```

`WeakMap` / `WeakSet` hold keys weakly (do not prevent garbage collection).

---

### 16. What is function currying?

Transforming a function with multiple arguments into a sequence of functions each taking **one** argument.

```js
const add = a => b => c => a + b + c;
add(1)(2)(3); // 6

// Generic curry
function curry(fn) {
  return function curried(...args) {
    return args.length >= fn.length
      ? fn.apply(this, args)
      : (...more) => curried.apply(this, [...args, ...more]);
  };
}
const sum = curry((a, b, c) => a + b + c);
sum(1)(2)(3); sum(1, 2)(3);
```

**Uses:** reusable/partially applied functions, function composition, cleaner config-style helpers.

---

### 17. Event delegation and event bubbling

**Event bubbling:** after an event fires on the target, it propagates **up** through its ancestors (target → parent → ... → `document`). The opposite direction is **capturing** (top-down, enabled with `{ capture: true }`).

**Event delegation:** attach **one** listener on a parent and use `event.target` to handle events from its children. This works because of bubbling.

```js
document.querySelector("#list").addEventListener("click", (e) => {
  const li = e.target.closest("li");
  if (!li) return;
  console.log("Clicked", li.textContent);
});
```

**Benefits:** fewer listeners (better memory), works for dynamically added elements.
Stop propagation with `e.stopPropagation()`; prevent default action with `e.preventDefault()`.

---

### 18. Debouncing and throttling

Both limit how often a function runs.

- **Debounce:** run only after the user **stops** triggering for `n` ms. Good for search input, window resize end.
- **Throttle:** run **at most once per `n` ms**. Good for scroll, mousemove, button spam.

```js
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

function throttle(fn, limit) {
  let waiting = false;
  return (...args) => {
    if (waiting) return;
    fn.apply(this, args);
    waiting = true;
    setTimeout(() => (waiting = false), limit);
  };
}
```

---

### 19. Shallow copy vs deep copy

**Shallow copy:** copies top-level properties; nested objects are still **shared** references.
**Deep copy:** copies everything recursively; no shared references.

```js
const original = { a: 1, nested: { b: 2 } };

const shallow1 = { ...original };
const shallow2 = Object.assign({}, original);
shallow1.nested.b = 99; // original.nested.b is also 99

const deep1 = structuredClone(original);          // modern, recommended
const deep2 = JSON.parse(JSON.stringify(original)); // loses functions, undefined, Date, Map, Set, circular refs fail
```

Libraries like Lodash's `_.cloneDeep` also work.

---

### Bonus JavaScript questions

**`this` keyword:** value depends on call site: global/`undefined` (strict) in plain calls, the object in method calls, new instance with `new`, explicit with `call/apply/bind`, lexical in arrow functions.

**Temporal Dead Zone:** the time between entering a scope and the `let`/`const` declaration line, during which access throws `ReferenceError`.

**Spread vs rest:** spread (`...arr`) expands values; rest (`function f(...args)`) collects values.

**Destructuring:** `const { a, b: renamed = 5 } = obj; const [x, , y] = arr;`

**`for...in` vs `for...of`:** `for...in` iterates enumerable keys (including inherited); `for...of` iterates values of iterables (arrays, strings, Maps, Sets).

**`map` vs `forEach`:** `map` returns a new array; `forEach` returns `undefined`.

**Memoization:** caching function results by arguments.
```js
const memo = fn => { const c = new Map(); return n => c.has(n) ? c.get(n) : (c.set(n, fn(n)), c.get(n)); };
```

**Generators:** functions declared with `function*` that can pause with `yield` and resume.

**Modules:** ES modules (`import/export`, static, async) vs CommonJS (`require/module.exports`, dynamic, sync).

**`typeof`, `instanceof`:** `typeof` returns a primitive type string; `instanceof` checks the prototype chain.

**Optional chaining and nullish coalescing:** `user?.address?.city`, `value ?? "default"` (only for `null`/`undefined`, unlike `||` which treats `0`, `""`, `false` as falsy).

**Event loop starvation / blocking:** long sync code blocks rendering and callbacks; split work using `setTimeout`, `requestIdleCallback` or Web Workers.

**Web Storage:** `localStorage` (persistent, ~5MB), `sessionStorage` (per tab), cookies (sent with every request, ~4KB, can be `HttpOnly`), IndexedDB (large structured data).

**Polyfill vs transpiler:** polyfill adds missing runtime APIs; transpiler (Babel) converts new syntax to older syntax.

---

# 2. React

### 1. What is JSX?

JSX (JavaScript XML) is a syntax extension that lets you write HTML-like markup inside JavaScript. It is compiled (by Babel/SWC) into `React.createElement()` / `jsx()` calls.

```jsx
const el = <h1 className="title">Hello, {name}</h1>;
// compiles to
const el2 = React.createElement("h1", { className: "title" }, "Hello, ", name);
```

Rules: one root element (or a Fragment), `className` instead of `class`, `htmlFor` instead of `for`, camelCase attributes, close all tags, JS expressions go inside `{}`.

---

### 2. Real DOM vs Virtual DOM

- **Real DOM:** the browser's actual tree of nodes. Updating it is relatively expensive (reflow/repaint).
- **Virtual DOM:** a lightweight JavaScript object tree that mirrors the real DOM. On state change, React builds a new virtual tree, **diffs** it against the previous one, and applies only the **minimal changes** to the real DOM.

Note: the benefit is not that the virtual DOM is "faster than DOM" but that it enables declarative UI with efficient, batched updates.

---

### 3. State vs Props

| Props | State |
|---|---|
| Passed **from parent** to child | Managed **inside** the component |
| Read-only (immutable) | Mutable via setter (`setState`) |
| Used to configure a component | Used for data that changes over time |
| Changing props re-renders the child | Changing state re-renders the component |

```jsx
function Child({ title }) { return <h2>{title}</h2>; }   // props
function Counter() { const [n, setN] = useState(0); }    // state
```

---

### 4. What are React Fragments?

Fragments let you return multiple elements without adding an extra DOM node.

```jsx
return (
  <>
    <h1>Title</h1>
    <p>Text</p>
  </>
);
// With key (needed in lists):
<React.Fragment key={id}>...</React.Fragment>
```

Useful in tables (`<tr>` must not be wrapped in a `<div>`), flexbox/grid layouts, and cleaner DOM.

---

### 5. What are Higher-Order Components (HOCs)?

A HOC is a function that takes a component and returns a **new enhanced component**. Used to reuse logic (auth, logging, theming).

```jsx
function withAuth(Component) {
  return function Wrapped(props) {
    const { user } = useAuth();
    return user ? <Component {...props} user={user} /> : <Navigate to="/login" />;
  };
}
const ProtectedDashboard = withAuth(Dashboard);
```

Today, **custom hooks** are generally preferred over HOCs.

---

### 6. What is prop drilling and how do you avoid it?

Passing props through many intermediate components that don't use them, just to reach a deeply nested child.

**Solutions:**
- **Context API** (`createContext` + `useContext`)
- State management libraries: **Redux Toolkit, Zustand, Jotai, Recoil**
- **Component composition** (pass JSX as `children`)
- Colocate state closer to where it is used

```jsx
const ThemeContext = createContext("light");
<ThemeContext.Provider value="dark"><App /></ThemeContext.Provider>
const theme = useContext(ThemeContext);
```

---

### 7. Controlled vs Uncontrolled components

**Controlled:** form input value is stored in React state; React is the "single source of truth".

```jsx
const [name, setName] = useState("");
<input value={name} onChange={e => setName(e.target.value)} />
```

**Uncontrolled:** the DOM keeps the value; you read it via a `ref` when needed.

```jsx
const ref = useRef();
<input defaultValue="x" ref={ref} />
// ref.current.value
```

Use controlled for validation and instant feedback; uncontrolled for simple forms, file inputs, or performance-sensitive large forms (or use libraries like React Hook Form).

---

### 8. What are React Hooks? Explain `useState`, `useEffect`, `useMemo`, `useCallback`

Hooks are functions that let function components use state and lifecycle features.

**Rules:** call hooks only at the top level (not in loops/conditions) and only from React functions or custom hooks.

**`useState`**: adds state.
```jsx
const [count, setCount] = useState(0);
setCount(prev => prev + 1); // functional update
```

**`useEffect`**: runs side effects (fetching, subscriptions, DOM work) after render.
```jsx
useEffect(() => {
  const id = setInterval(tick, 1000);
  return () => clearInterval(id);   // cleanup
}, [dep]);                           // [] = once, [dep] = when dep changes, none = every render
```

**`useMemo`**: memoizes a **computed value**.
```jsx
const sorted = useMemo(() => heavySort(items), [items]);
```

**`useCallback`**: memoizes a **function reference** (useful when passing callbacks to `React.memo` children).
```jsx
const handleClick = useCallback(() => doSomething(id), [id]);
```

Other important hooks: `useRef`, `useContext`, `useReducer`, `useLayoutEffect`, `useId`, `useTransition`, `useDeferredValue`.

---

### 9. What is a custom hook?

A JavaScript function whose name starts with `use` and that calls other hooks, used to **reuse stateful logic**.

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    fetch(url, { signal: controller.signal })
      .then(r => r.json())
      .then(setData)
      .catch(e => e.name !== "AbortError" && setError(e))
      .finally(() => setLoading(false));
    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}
```

---

### 10. How do you optimize React app performance? Explain `React.memo`

**Techniques:**
- `React.memo`, `useMemo`, `useCallback`
- Code splitting and lazy loading (`React.lazy`, `Suspense`)
- Stable, unique `key` props in lists (not array index for dynamic lists)
- List virtualization (`react-window`, `react-virtuoso`)
- Avoid unnecessary state / lift or colocate state appropriately
- Debounce expensive handlers; use `useTransition` / `useDeferredValue`
- Optimize images (WebP/AVIF, lazy loading), bundle analysis, tree shaking
- Use the React DevTools Profiler to find actual bottlenecks first
- Production build, SSR/SSG where applicable

**`React.memo`:** a HOC that skips re-rendering a component if its props are **shallowly equal** to the previous props.

```jsx
const Item = React.memo(function Item({ name }) {
  return <li>{name}</li>;
});
```

It is ineffective if you pass new object/function references every render (combine with `useMemo`/`useCallback`).

---

### 11. What is diffing and reconciliation?

**Reconciliation** is the process React uses to update the DOM by comparing the new virtual tree with the old one. The **diffing algorithm** has heuristics to make it O(n):

1. Elements of **different types** produce different trees (the old subtree is unmounted, the new one is mounted).
2. Elements of the **same type** are updated in place (only changed attributes/props).
3. **Keys** identify which items in a list changed, moved, were added or removed.

React Fiber (the current engine) splits rendering into units of work so it can pause, prioritize and resume (enabling concurrent features).

---

### 12. Explain code splitting in React

Splitting the bundle into smaller chunks loaded **on demand** to reduce initial load time.

```jsx
import { lazy, Suspense } from "react";
const Dashboard = lazy(() => import("./Dashboard"));

<Suspense fallback={<Spinner />}>
  <Dashboard />
</Suspense>
```

Common strategy: **route-based splitting** (each route lazily loaded), plus splitting heavy components (charts, editors, modals). Bundlers (Vite/Webpack) create separate chunks from dynamic `import()`.

---

### 13. Shadow DOM vs Virtual DOM

| Shadow DOM | Virtual DOM |
|---|---|
| Browser **standard** (Web Components) | **React concept/library technique** |
| Encapsulates DOM and CSS inside an element so styles don't leak in/out | In-memory representation used to compute efficient DOM updates |
| Real DOM subtree attached to a host | Plain JS objects |
| Used for custom elements, `<video>` controls, widgets | Used for rendering performance and declarative UI |

---

### 14. What is `StrictMode`? Explain Server Components

**`<React.StrictMode>`:** a development-only wrapper that highlights potential problems. It double-invokes render functions, effects (mount → unmount → mount) and warns about deprecated APIs, unsafe lifecycles and impure rendering. It has no effect in production.

```jsx
<React.StrictMode><App /></React.StrictMode>
```

**React Server Components (RSC):** components that render **only on the server** and send serialized output to the client, with **zero JS** shipped for them.

- Can access databases, file system and secrets directly; can be `async`.
- Cannot use state, effects or browser APIs.
- Client components are marked with `"use client"` at the top of the file.
- Benefits: smaller bundles, faster initial load, secure data access. Used in the Next.js App Router.

```jsx
// Server Component (default in Next.js App Router)
export default async function Page() {
  const posts = await db.post.findMany();
  return <PostList posts={posts} />;
}
```

---

### Bonus React questions

**Keys:** unique, stable identifiers for list items so React can reconcile efficiently. Avoid index keys when the list can reorder/insert/delete.

**Lifting state up:** move shared state to the closest common parent and pass down via props.

**`useRef`:** holds a mutable value that persists across renders without causing re-render; also used to access DOM nodes.

**`useReducer` vs `useState`:** `useReducer` suits complex state logic with many transitions (`dispatch({ type })`).

**`useEffect` vs `useLayoutEffect`:** `useLayoutEffect` runs synchronously after DOM mutation but before paint (measure layout); `useEffect` runs after paint.

**Why can't you mutate state directly?** React compares by reference; direct mutation won't trigger re-render and breaks memoization.

**Error Boundaries:** class components with `componentDidCatch` / `getDerivedStateFromError` that catch render errors in children and show fallback UI (they do not catch event handler or async errors).

**Context API drawbacks:** all consumers re-render when the value changes; split contexts or use selectors/state libraries.

**Redux flow:** UI dispatches **action** → **reducer** computes new state → **store** updates → UI re-renders. Redux Toolkit simplifies this (`createSlice`, `configureStore`).

**CSR vs SSR vs SSG vs ISR:** client-side rendering, server-side rendering per request, static generation at build time, incremental static regeneration (static + revalidate).

**React 18/19 features:** automatic batching, concurrent rendering, `useTransition`, `useDeferredValue`, Suspense improvements, `use()` hook, Actions, `useOptimistic`, `useActionState`.

**Portals:** `ReactDOM.createPortal(child, container)` renders children outside the parent DOM hierarchy (modals, tooltips).

**Refs forwarding:** `forwardRef` (or `ref` as a prop in React 19) passes a ref through a component to a child DOM node.

**Hydration:** attaching event listeners and state to server-rendered HTML on the client.

---

# 3. Node.js

### 1. How is Node.js single-threaded yet able to handle concurrent requests?

Your JavaScript code runs on **one main thread**, but Node uses an **event-driven, non-blocking I/O** model powered by **libuv**.

- Async operations (network, file system, DNS, crypto) are delegated to the OS kernel or to libuv's **thread pool** (default 4 threads, set with `UV_THREADPOOL_SIZE`).
- When they finish, callbacks are queued and the **event loop** executes them on the main thread.
- So one thread can juggle thousands of connections since it doesn't wait on I/O.

Weakness: CPU-heavy synchronous work blocks the loop. Offload it to worker threads / child processes / separate services.

---

### 2. `setImmediate()` vs `process.nextTick()`

| `process.nextTick()` | `setImmediate()` |
|---|---|
| Runs **before** the event loop continues (right after the current operation), in the nextTick queue | Runs in the **check phase** of the event loop (after the poll phase) |
| Higher priority than Promise microtasks | Runs after I/O callbacks |
| Overuse can starve I/O | Safe, lets I/O proceed |

```js
setImmediate(() => console.log("immediate"));
process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("promise"));
console.log("sync");
// sync, nextTick, promise, immediate
```

Inside an I/O callback, `setImmediate` always fires before `setTimeout(fn, 0)`.

---

### 3. What is callback hell and how do you solve it?

Deeply nested callbacks that make code hard to read and maintain (the "pyramid of doom").

```js
getUser(id, (err, user) => {
  getPosts(user.id, (err, posts) => {
    getComments(posts[0].id, (err, comments) => { /* ... */ });
  });
});
```

**Solutions:** Promises (chaining), **async/await**, modularizing functions into named ones, `Promise.all` for parallel tasks, `util.promisify` for legacy callback APIs.

```js
try {
  const user = await getUser(id);
  const posts = await getPosts(user.id);
  const comments = await getComments(posts[0].id);
} catch (e) { /* handle */ }
```

---

### 4. What are streams and why use them? Explain backpressure

Streams process data **in chunks** instead of loading everything in memory.

Types: **Readable**, **Writable**, **Duplex** (both), **Transform** (modify while passing through).

**Why:** low memory usage for big files, faster time-to-first-byte, composable pipelines.

```js
const fs = require("fs");
fs.createReadStream("big.mp4").pipe(res); // stream a video to the client
```

**Backpressure:** when a producer (readable) generates data faster than the consumer (writable) can handle, the internal buffer fills up. If ignored, memory grows. Writable's `write()` returns `false` when the buffer is full (`highWaterMark`); the producer should pause until the `'drain'` event.

`pipe()` and `pipeline()` handle this automatically.

```js
if (!writable.write(chunk)) {
  readable.pause();
  writable.once("drain", () => readable.resume());
}
```

---

### 5. `worker_threads` vs `child_process`

| `worker_threads` | `child_process` |
|---|---|
| Threads in the **same process** | Separate **OS process** |
| Share memory via `SharedArrayBuffer`; messaging via `postMessage` | Communicate via IPC / stdio |
| Lightweight, fast startup | Heavier, isolated memory |
| Best for **CPU-intensive JS** (image processing, hashing, parsing) | Best for running **external programs/scripts**, isolation (`spawn`, `exec`, `fork`) |

Use neither for I/O-bound work: the event loop already handles it.

---

### 6. How do you handle uncaught exceptions?

```js
process.on("uncaughtException", (err) => {
  logger.error(err);
  // state may be corrupt: do cleanup then exit
  process.exit(1);
});

process.on("unhandledRejection", (reason) => {
  logger.error(reason);
  process.exit(1);
});
```

Best practices:
- Prefer `try/catch` with async/await and proper error handling in promises.
- Treat `uncaughtException` as **last resort**: log, gracefully close server/DB, then exit.
- Use a process manager (PM2, systemd, Docker/Kubernetes) to restart.
- Use centralized error handling (Express error middleware) and monitoring (Sentry).

---

### 7. `pipe()` vs `pipeline()`

- `pipe()`: connects a readable to a writable and handles backpressure, but **does not forward errors** or destroy streams on failure (can leak resources). Needs manual error handlers on each stream.
- `stream.pipeline()`: chains multiple streams, **propagates errors**, **cleans up** (destroys all streams) and provides a single callback/promise.

```js
const { pipeline } = require("stream/promises");
const zlib = require("zlib");

await pipeline(
  fs.createReadStream("in.txt"),
  zlib.createGzip(),
  fs.createWriteStream("in.txt.gz")
);
```

Prefer `pipeline()`.

---

### 8. Explain the `cluster` module and how it differs from worker threads

`cluster` forks **multiple Node processes** (one per CPU core) that **share the same server port**, letting you use all cores for handling HTTP requests.

```js
const cluster = require("cluster");
const os = require("os");
if (cluster.isPrimary) {
  os.cpus().forEach(() => cluster.fork());
  cluster.on("exit", () => cluster.fork()); // restart
} else {
  require("./server"); // each worker runs the server
}
```

| `cluster` | `worker_threads` |
|---|---|
| Multiple **processes**, separate memory | Multiple **threads**, can share memory |
| Scales a server across cores / improves availability | Offloads CPU-heavy tasks inside one process |
| Heavier | Lighter |

PM2 cluster mode does this automatically.

---

### 9. What are memory leaks in Node.js and how do you detect them?

A leak is memory that is no longer needed but still referenced, so the GC can't free it, causing a steady rise in memory and eventual crash.

**Common causes:** global variables, unbounded caches/arrays/maps, forgotten timers/intervals, un-removed event listeners, closures holding large objects, unclosed DB connections/streams.

**Detection:**
- Monitor `process.memoryUsage()` (heapUsed, rss).
- Run with `node --inspect` and use Chrome DevTools → **Memory** tab → heap snapshots (compare over time), allocation timeline.
- Tools: `clinic.js` (doctor, heapprofiler), `heapdump`, `memwatch-next`, APM tools (Datadog, New Relic).
- Load test and watch for a heap that never shrinks after GC.

**Fixes:** remove listeners, clear timers, use `WeakMap`/`WeakRef`, bounded LRU cache, stream large data.

---

### 10. What is the use of Buffer in Node.js?

`Buffer` is a fixed-size chunk of memory (outside the V8 heap) for handling **raw binary data**: files, network packets, images, encryption, streams.

```js
const buf = Buffer.from("Hello", "utf8");
buf;                       // <Buffer 48 65 6c 6c 6f>
buf.toString("base64");    // "SGVsbG8="
Buffer.alloc(10);          // zero-filled, safe
Buffer.concat([buf1, buf2]);
```

`Buffer` is a subclass of `Uint8Array`. Prefer `Buffer.alloc()` over the unsafe `Buffer.allocUnsafe()` unless you overwrite it fully.

---

### 11. How do you secure a Node.js API?

- **HTTPS/TLS** everywhere; set secure headers with **helmet**
- **Authentication & authorization**: JWT/session, OAuth2, role-based access control, short-lived tokens + refresh tokens
- **Validate and sanitize input** (Zod/Joi/express-validator) to prevent injection (SQL/NoSQL), XSS
- **Rate limiting** and brute-force protection (`express-rate-limit`), request size limits
- **CORS** configured with an allow-list
- **Hash passwords** with bcrypt/argon2 (never store plain text)
- Use **parameterized queries**/ORM; avoid `eval`
- **Secrets** in environment variables / secret manager, never in git
- Keep dependencies updated (`npm audit`), lock files
- Secure cookies (`HttpOnly`, `Secure`, `SameSite`), CSRF protection for cookie auth
- Proper error handling (don't leak stack traces), logging and monitoring
- Run as non-root, least-privilege DB users

---

### 12. Explain the event loop in Node.js

The loop runs in **phases** (libuv):

1. **Timers**: `setTimeout`, `setInterval` callbacks
2. **Pending callbacks**: some system-level callbacks
3. **Idle, prepare**: internal
4. **Poll**: retrieve new I/O events; execute I/O callbacks (may block here waiting)
5. **Check**: `setImmediate` callbacks
6. **Close callbacks**: e.g. `socket.on('close')`

Between each phase (and after each callback), Node drains the `process.nextTick` queue first, then the **Promise microtask** queue.

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

---

### 13. Explain the request/response cycle in Node.js

1. Client sends an HTTP request (DNS → TCP → TLS → HTTP).
2. Node's `http` server receives it and creates `req` (IncomingMessage, a readable stream) and `res` (ServerResponse, a writable stream).
3. Request goes through routing/middleware (in frameworks), body is parsed, auth is checked.
4. Handler runs business logic, possibly async DB/API calls (non-blocking).
5. Handler sends the response: status code, headers, body (`res.writeHead`, `res.end`).
6. Connection is closed or kept alive.

```js
const http = require("http");
http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(JSON.stringify({ ok: true }));
}).listen(3000);
```

---

### 14. The module system (CommonJS vs ESM)

| CommonJS (CJS) | ES Modules (ESM) |
|---|---|
| `require()` / `module.exports` | `import` / `export` |
| Synchronous loading | Asynchronous, statically analyzable |
| Dynamic (`require` anywhere) | Static (top-level imports); dynamic via `import()` |
| `__dirname`, `__filename` available | Use `import.meta.url` / `import.meta.dirname` |
| Default in `.js` unless `"type": "module"` | Use `.mjs` or `"type": "module"` in package.json |
| Values are copied/cached exports | Live bindings |
| No top-level `await` | Top-level `await` supported |

```js
// CJS
const fs = require("fs");
module.exports = { a };
// ESM
import fs from "fs";
export const a = 1;
export default fn;
```

---

### 15. Explain garbage collection and the V8 engine

**V8** is Google's open-source JS engine (written in C++) used by Chrome and Node. It uses **Ignition** (interpreter producing bytecode) and **TurboFan** (optimizing JIT compiler producing machine code for hot functions).

**Garbage collection** automatically frees memory for unreachable objects (mark-and-sweep reachability from roots).

V8 uses a **generational** heap:
- **Young generation** (new space, small): objects start here. Collected often by the fast **Scavenge** (Minor GC, copying collector).
- **Old generation**: objects that survive two scavenges get promoted. Collected by **Mark-Sweep-Compact** (Major GC), with incremental, concurrent and parallel techniques to reduce pauses.

Tuning: `--max-old-space-size=4096`. Inspect with `--trace-gc`.

---

# 4. Express.js

### 1. What is middleware in Express?

Functions that have access to `req`, `res` and `next`. They run **in order** during the request cycle and can modify `req/res`, end the response, or call `next()` to pass control.

Types: application-level, router-level, error-handling (4 args), built-in (`express.json`), third-party (`cors`, `helmet`, `morgan`).

```js
const logger = (req, res, next) => {
  console.log(req.method, req.url);
  next();
};
app.use(logger);
```

---

### 2. `app.use()` vs `app.get()`

| `app.use(path?, fn)` | `app.get(path, fn)` |
|---|---|
| Mounts middleware for **all HTTP methods** | Handles **only GET** requests |
| Path matches as a **prefix** (`/api` matches `/api/users`) | Path matches **exactly** |
| Default path is `/` | Path is required |
| Typically calls `next()` | Typically ends the response |

---

### 3. How do you handle routing in Express?

```js
// routes/users.js
const router = require("express").Router();
router.get("/", listUsers);
router.get("/:id", getUser);
router.post("/", createUser);
router.route("/:id").put(updateUser).delete(deleteUser);
module.exports = router;

// app.js
app.use("/api/users", require("./routes/users"));
```

Use `express.Router()` to modularize, organize by feature (routes → controllers → services), version APIs (`/api/v1`), add a 404 handler at the end.

---

### 4. How do you handle errors in Express?

Error-handling middleware has **four arguments** and is placed **after** all routes.

```js
class AppError extends Error {
  constructor(message, status = 500) { super(message); this.status = status; }
}

app.use((req, res, next) => next(new AppError("Not found", 404)));

app.use((err, req, res, next) => {
  res.status(err.status || 500).json({ message: err.message });
});
```

For sync code, thrown errors are caught automatically. For async code in Express 4 you must call `next(err)` (or wrap handlers); Express 5 forwards rejected promises automatically.

---

### 5. Purpose of `express.json()` and `express.urlencoded()`

Built-in body-parsing middleware that populates `req.body`.

- `express.json()`: parses `application/json` bodies.
- `express.urlencoded({ extended: true })`: parses HTML form data (`application/x-www-form-urlencoded`). `extended: true` supports nested objects (via `qs`).

```js
app.use(express.json({ limit: "10kb" }));
app.use(express.urlencoded({ extended: true }));
```

Without them `req.body` is `undefined`. For file uploads (`multipart/form-data`) use `multer`.

---

### 6. How do you manage sessions or cookies?

**Cookies** with `cookie-parser`:
```js
const cookieParser = require("cookie-parser");
app.use(cookieParser());
res.cookie("token", value, { httpOnly: true, secure: true, sameSite: "strict", maxAge: 3600000 });
res.clearCookie("token");
```

**Sessions** with `express-session` (store the session in Redis/Mongo, not memory, in production):
```js
const session = require("express-session");
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: { httpOnly: true, secure: true, maxAge: 1000 * 60 * 60 },
}));
req.session.userId = user.id;
req.session.destroy();
```

Alternative: stateless **JWT** stored in an `HttpOnly` cookie.

---

### 7. What are sessions and cookies?

- **Cookie:** a small piece of data (≤ ~4KB) stored in the browser and sent automatically with every request to the matching domain. Can be configured with `HttpOnly`, `Secure`, `SameSite`, `Expires/Max-Age`, `Domain`, `Path`.
- **Session:** **server-side** storage of user data, identified by a **session ID** stored in a cookie. Data stays on the server (more secure for sensitive info), but requires server storage and scaling considerations.

| Cookie | Session |
|---|---|
| Stored on client | Stored on server (ID on client) |
| Less secure for sensitive data | More secure |
| Size-limited | Larger |

---

### 8. What is scaffolding in Express?

Generating a **ready-made project structure** and boilerplate to start quickly.

```bash
npx express-generator myapp
# or modern alternatives: npm init, create-express-app, NestJS CLI
```

Typical generated structure: `app.js`, `bin/www`, `routes/`, `views/`, `public/`, `package.json`.

A common scalable structure:
```
src/
  config/  controllers/  routes/  services/  models/  middlewares/  utils/
  app.js  server.js
```

---

### 9. How do you handle async code in Express?

Use `async/await` and make sure errors reach the error handler.

```js
const asyncHandler = fn => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get("/users", asyncHandler(async (req, res) => {
  const users = await User.find();
  res.json(users);
}));
```

Or use `express-async-errors`, or Express 5 which handles rejected promises natively. Use `Promise.all` for parallel calls.

---

### 10. `res.send()` vs `res.json()` vs `res.end()`

| Method | Behavior |
|---|---|
| `res.send(body)` | Sends string, Buffer, object, array. Auto-sets `Content-Type` (`text/html` for strings, JSON for objects) and `ETag` |
| `res.json(obj)` | Serializes with `JSON.stringify`, sets `Content-Type: application/json`. Also converts `null`/`undefined` properly |
| `res.end([data])` | Native Node method, ends the response with **no** automatic headers or content-type handling. Use for no-body responses or streams |

```js
res.status(204).end();
res.json({ ok: true });
res.send("<h1>Hi</h1>");
```

---

### 11. Route parameters vs query string

- **Route params** (`req.params`): part of the path, identify a **specific resource**. `GET /users/42` → `/users/:id`.
- **Query string** (`req.query`): after `?`, used for **filtering, sorting, pagination, search**. `GET /users?role=admin&page=2`.

```js
app.get("/users/:id", (req, res) => res.json({ id: req.params.id }));
app.get("/users", (req, res) => {
  const { page = 1, limit = 10, q } = req.query;
});
```

---

### 12. What is `app.param()` and why is it used?

Defines **callback triggers for route parameters**: runs before any route handler that has that param, so you can validate or preload data once.

```js
app.param("userId", async (req, res, next, id) => {
  const user = await User.findById(id);
  if (!user) return next(new AppError("User not found", 404));
  req.user = user;
  next();
});

app.get("/users/:userId", (req, res) => res.json(req.user));
app.delete("/users/:userId", (req, res) => { /* req.user already loaded */ });
```

Note: `app.param` is **deprecated-ish/discouraged in Express 5 docs** for some cases; a reusable middleware function is a common alternative.

---

### 13. How can you improve Express.js performance?

- Use **gzip/brotli compression** (`compression`) or let Nginx do it
- Run in **production mode** (`NODE_ENV=production`), which enables view caching and less verbose errors
- **Cluster/PM2** to use all CPU cores; scale horizontally behind a load balancer
- **Cache** (Redis, HTTP cache headers, ETag), CDN for static assets
- Avoid **synchronous** blocking code (`fs.readFileSync`, heavy loops)
- **Database**: indexes, connection pooling, pagination, `lean()`/projections, avoid N+1 queries
- Use a **reverse proxy** (Nginx) for static files, TLS, rate limiting
- Use efficient logging (pino rather than console.log)
- Stream large responses; keep middleware minimal and ordered sensibly
- Profile (clinic.js, `--prof`) and monitor

---

### 14. How do you implement CORS in Express?

```js
const cors = require("cors");

app.use(cors({
  origin: ["https://myapp.com", "http://localhost:5173"],
  methods: ["GET", "POST", "PUT", "DELETE"],
  allowedHeaders: ["Content-Type", "Authorization"],
  credentials: true,       // allow cookies
  maxAge: 600,
}));
```

Manual version:
```js
app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "https://myapp.com");
  res.header("Access-Control-Allow-Headers", "Content-Type, Authorization");
  if (req.method === "OPTIONS") return res.sendStatus(204);
  next();
});
```

Note: with `credentials: true` you cannot use `origin: "*"`.

---

### 15. What is CORS?

**Cross-Origin Resource Sharing** is a browser security mechanism that controls whether a web page from one **origin** (scheme + host + port) can request resources from a different origin. It relaxes the **Same-Origin Policy** using HTTP headers set by the server.

- **Simple requests** (GET/POST with basic headers) go straight through; the browser checks `Access-Control-Allow-Origin` in the response.
- **Preflight requests:** for PUT/DELETE, custom headers or JSON content-type, the browser first sends an `OPTIONS` request asking permission. The server replies with `Access-Control-Allow-Methods/Headers`.
- CORS is enforced **by the browser only**: Postman/curl and server-to-server calls ignore it.

Key headers: `Access-Control-Allow-Origin`, `-Allow-Methods`, `-Allow-Headers`, `-Allow-Credentials`, `-Max-Age`.

---

# 5. HTML

### 1. What is semantic HTML and why does it matter?

Using elements that describe their **meaning**: `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>`, `<figure>`, `<time>`, instead of generic `<div>`/`<span>`.

**Benefits:** accessibility (screen readers), SEO, readability/maintainability, better default browser behavior.

---

### 2. Difference between `<div>` and `<span>`; block vs inline elements

- **Block** (`div`, `p`, `h1`, `section`, `ul`): start on a new line, take full width, accept width/height/margins.
- **Inline** (`span`, `a`, `strong`, `em`): flow within text, only as wide as content, ignore width/height and vertical margins.
- **Inline-block**: flows inline but accepts width/height.

`<div>` is a block container, `<span>` is an inline container; both have no semantic meaning.

---

### 3. What is the difference between `<script>`, `<script async>` and `<script defer>`?

| Attribute | Download | Execution |
|---|---|---|
| none | Blocks HTML parsing | Immediately, blocking |
| `async` | In parallel | As soon as downloaded (order **not** guaranteed) |
| `defer` | In parallel | After HTML is parsed, in order, before `DOMContentLoaded` |

Use `defer` for most app scripts, `async` for independent scripts (analytics).

---

### 4. `localStorage` vs `sessionStorage` vs cookies

| | localStorage | sessionStorage | Cookies |
|---|---|---|---|
| Lifetime | Until cleared | Until tab closes | Expiry set by you |
| Size | ~5-10 MB | ~5 MB | ~4 KB |
| Sent to server | No | No | Yes, every request |
| Access | JS | JS | JS (unless `HttpOnly`) + server |

---

### 5. What are `data-*` attributes?

Custom attributes to store extra data on elements, accessible via JS `dataset`.

```html
<button data-user-id="42" data-role="admin">Edit</button>
<script>
  btn.dataset.userId; // "42"
</script>
```

---

### 6. `<img>` best practices: `alt`, `srcset`, `loading`

```html
<img
  src="photo-800.jpg"
  srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1600.jpg 1600w"
  sizes="(max-width: 600px) 100vw, 50vw"
  alt="Team working at a desk"
  width="800" height="600"
  loading="lazy" decoding="async" />
```

Always provide `alt` (empty `alt=""` for decorative), set width/height to prevent layout shift (CLS), use `<picture>` for format/art-direction switching (WebP/AVIF).

---

### 7. What is the DOCTYPE and why is it needed?

`<!DOCTYPE html>` tells the browser to render in **standards mode** (HTML5). Without it, browsers use **quirks mode** with inconsistent legacy behavior.

---

### 8. `<meta>` tags you should know

```html
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<meta name="description" content="Short page summary for SEO" />
<meta property="og:title" content="Title" />   <!-- Open Graph for social sharing -->
```

The viewport tag is essential for responsive design on mobile.

---

### 9. HTML accessibility (a11y) basics

- Use semantic elements and proper heading order (`h1` → `h2` ...)
- `alt` text for images; `<label for="id">` for inputs
- Keyboard accessibility (focus order, visible focus, `tabindex` only 0 or -1)
- **ARIA** (`role`, `aria-label`, `aria-expanded`, `aria-live`) only when native HTML isn't enough ("first rule of ARIA: don't use ARIA if a native element works")
- Sufficient color contrast (WCAG AA: 4.5:1 for normal text)
- Use `<button>` for actions and `<a>` for navigation

---

### 10. What are HTML5's new features?

Semantic tags, `<audio>`/`<video>`, `<canvas>` and SVG, form input types (`email`, `date`, `range`, `number`) and validation attributes (`required`, `pattern`), Web Storage, Geolocation API, Web Workers, drag and drop, `<dialog>`, `<template>`, Web Components.

---

### 11. `<b>` vs `<strong>`, `<i>` vs `<em>`

`<strong>` and `<em>` carry **semantic importance/emphasis** (read differently by screen readers). `<b>` and `<i>` are only **stylistic** (bold/italic with no extra meaning).

---

### 12. What happens when you type a URL and press Enter?

1. Browser checks cache; **DNS lookup** resolves the domain to an IP.
2. **TCP connection** (3-way handshake) and **TLS handshake** for HTTPS.
3. Browser sends an **HTTP request**; server responds with HTML.
4. Browser **parses HTML → DOM**, **CSS → CSSOM**, combines into the **render tree**.
5. **Layout** (reflow) → **Paint** → **Composite**.
6. JS executes (can modify the DOM); more resources (CSS, JS, images) are fetched.

---

# 6. CSS

### 1. Explain the CSS box model

Every element is a box: **content → padding → border → margin**.

`box-sizing: content-box` (default): `width` = content only.
`box-sizing: border-box`: `width` includes padding and border (easier to reason about).

```css
*, *::before, *::after { box-sizing: border-box; }
```

---

### 2. CSS specificity and the cascade

Specificity decides which rule wins (highest first):

1. `!important`
2. Inline styles (1000)
3. IDs (100)
4. Classes, attributes, pseudo-classes (10)
5. Elements, pseudo-elements (1)

If equal, the **later** rule wins. Also inheritance and origin matter. Modern CSS also has `@layer` to control cascade order, and `:where()` (zero specificity) / `:is()`.

---

### 3. Difference between `display: none`, `visibility: hidden`, `opacity: 0`

| Property | Takes space? | Accessible to screen readers? | Receives events? |
|---|---|---|---|
| `display: none` | No | No | No |
| `visibility: hidden` | Yes | No | No |
| `opacity: 0` | Yes | Yes | Yes |

---

### 4. `position` values

| Value | Behavior |
|---|---|
| `static` | Default, normal flow |
| `relative` | Offset from its normal position; still takes original space; creates a positioning context |
| `absolute` | Removed from flow; positioned relative to nearest **positioned** ancestor |
| `fixed` | Relative to the **viewport**; stays on scroll |
| `sticky` | Hybrid: relative until a scroll threshold, then sticks |

---

### 5. Flexbox: explain with key properties

One-dimensional layout (row or column).

```css
.container {
  display: flex;
  flex-direction: row;            /* main axis */
  justify-content: space-between; /* main axis alignment */
  align-items: center;            /* cross axis alignment */
  flex-wrap: wrap;
  gap: 1rem;
}
.item { flex: 1 1 200px; }        /* grow shrink basis */
```

Centering: `display:flex; justify-content:center; align-items:center;`

---

### 6. CSS Grid: explain with key properties

Two-dimensional layout (rows and columns).

```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}
.header { grid-column: 1 / -1; }
.layout { grid-template-areas: "header header" "sidebar main"; }
```

**Flexbox vs Grid:** Flexbox for content-driven 1D alignment (navbars, toolbars); Grid for page layout and 2D placement. They are often combined.

---

### 7. `em` vs `rem` vs `px` vs `%` vs `vh/vw`

- `px`: absolute pixels
- `em`: relative to the **parent's** font size (compounds when nested; for padding it's relative to the element's own font size)
- `rem`: relative to the **root** (`html`) font size (predictable, good for accessibility)
- `%`: relative to the parent's corresponding property
- `vw/vh`: 1% of viewport width/height (`dvh` handles mobile browser bars)

---

### 8. Responsive design and media queries

Responsive design adapts layout to screen size using fluid grids, flexible images and **media queries**, with a **mobile-first** approach.

```css
.card { padding: 1rem; }
@media (min-width: 768px) {
  .card { padding: 2rem; }
}
img { max-width: 100%; height: auto; }
```

Modern tools: `clamp()` for fluid type (`font-size: clamp(1rem, 2.5vw, 2rem)`), container queries (`@container`), `min()/max()`.

---

### 9. Pseudo-classes vs pseudo-elements

- **Pseudo-class** (single colon): a state of an element: `:hover`, `:focus`, `:nth-child(2)`, `:not()`, `:has()`.
- **Pseudo-element** (double colon): a part of an element: `::before`, `::after`, `::first-line`, `::placeholder`, `::selection`.

```css
.btn:hover { background: #333; }
.title::after { content: "→"; }
```

---

### 10. What is the difference between `relative`, `absolute` centering and how do you center a div?

```css
/* Flexbox */
.parent { display: flex; place-items: center; justify-content: center; }
/* Grid */
.parent { display: grid; place-items: center; }
/* Absolute */
.child { position: absolute; inset: 0; margin: auto; width: 200px; height: 100px; }
/* Horizontal only */
.child { margin: 0 auto; width: 300px; }
```

---

### 11. What is z-index and stacking context?

`z-index` controls stacking order of **positioned** elements (and flex/grid items). A **stacking context** is created by the root, `position` + `z-index`, `opacity < 1`, `transform`, `filter`, `isolation: isolate`, etc. A child's `z-index` only competes **within** its context, so a child can never go above an element outside its parent's context with a higher z-index.

---

### 12. CSS variables (custom properties)

```css
:root { --primary: #4f46e5; --gap: 1rem; }
.btn { background: var(--primary, blue); padding: var(--gap); }
[data-theme="dark"] { --primary: #818cf8; }
```

They cascade, can be changed at runtime with JS (`el.style.setProperty("--x", "1")`), and are great for theming.

---

### 13. Transitions vs animations; transforms

```css
.btn { transition: transform 0.3s ease, background 0.3s; }
.btn:hover { transform: translateY(-2px) scale(1.05); }

@keyframes spin { to { transform: rotate(360deg); } }
.loader { animation: spin 1s linear infinite; }
```

Animate `transform` and `opacity` for smooth 60fps (GPU-composited, avoids layout/paint). Respect `prefers-reduced-motion`.

---

### 14. Reflow vs repaint

- **Reflow (layout):** recalculating element sizes/positions (triggered by changing width, height, margin, adding/removing elements, reading `offsetHeight`). Expensive.
- **Repaint:** redrawing visuals without changing layout (color, background, visibility).

Reduce them by batching DOM changes, using `transform`/`opacity`, avoiding layout thrashing (interleaved reads/writes).

---

### 15. CSS preprocessors, CSS-in-JS and utility-first CSS

- **Sass/SCSS/Less:** variables, nesting, mixins, functions (compile to CSS). Native CSS now has variables and nesting too.
- **CSS Modules:** locally scoped class names per file.
- **CSS-in-JS** (styled-components, Emotion): styles in JS with props-based theming.
- **Tailwind CSS:** utility-first classes directly in markup; small production CSS via purging/JIT.

---

### 16. Other CSS questions

- **`inline` vs `inline-block` vs `block`**: see HTML section.
- **Margin collapsing:** vertical margins of adjacent block elements combine into the larger one (not in flex/grid).
- **`@import` vs `<link>`:** `<link>` loads in parallel; `@import` is blocking and slower.
- **BEM naming:** `block__element--modifier` (`card__title--large`).
- **`object-fit`:** `cover` / `contain` for images in fixed boxes.
- **`:is()`, `:where()`, `:has()`:** modern selectors for grouping and parent selection.
- **`rem`-based accessible typography and `prefers-color-scheme`** for dark mode.
- **Critical CSS:** inline above-the-fold CSS for faster first paint.

---

# 7. Other Important Topics

## Git

**`merge` vs `rebase`:** `merge` creates a merge commit, preserving history; `rebase` replays your commits on top of another branch for a linear history (don't rebase shared/public branches).

**`git fetch` vs `git pull`:** `fetch` downloads changes only; `pull` = `fetch` + `merge` (or rebase).

**`reset` vs `revert`:** `reset` moves the branch pointer (rewrites history; `--soft`, `--mixed`, `--hard`); `revert` creates a new commit that undoes a previous one (safe for shared branches).

**Other:** `git stash`, `cherry-pick`, `git bisect`, resolving merge conflicts, `.gitignore`, Gitflow/trunk-based workflows, pull requests.

```bash
git checkout -b feature/login
git add . && git commit -m "feat: add login"
git push -u origin feature/login
git rebase main
git stash pop
```

---

## HTTP & REST

**HTTP methods:** `GET` (read, idempotent, safe), `POST` (create), `PUT` (replace, idempotent), `PATCH` (partial update), `DELETE` (idempotent), `OPTIONS`, `HEAD`.

**Status codes:** `200 OK`, `201 Created`, `204 No Content`, `301/302` redirects, `304 Not Modified`, `400 Bad Request`, `401 Unauthorized` (not authenticated), `403 Forbidden` (not allowed), `404 Not Found`, `409 Conflict`, `422 Unprocessable`, `429 Too Many Requests`, `500 Internal Server Error`, `502/503/504`.

**REST principles:** client-server, stateless, cacheable, uniform interface (resource-based URLs, correct verbs), layered system. Example: `GET /api/users/42`, `POST /api/users`.

**REST vs GraphQL:** REST has multiple endpoints with fixed response shapes (risk of over/under-fetching); GraphQL has a single endpoint where the client specifies exactly the fields it needs.

**HTTP vs HTTPS, HTTP/1.1 vs HTTP/2:** HTTPS adds TLS encryption; HTTP/2 offers multiplexing, header compression, server push.

**Caching headers:** `Cache-Control`, `ETag`, `Last-Modified`, `Expires`.

---

## Authentication & Authorization

**Authentication** = who you are; **Authorization** = what you may do.

**JWT:** `header.payload.signature` (Base64URL). Stateless; verify with the secret/public key. Use short-lived access tokens + refresh tokens; store in `HttpOnly` cookies (safer against XSS than localStorage). Don't put secrets in the payload (it's only encoded, not encrypted).

**Session vs JWT:** sessions are server-stored and easy to revoke; JWTs are stateless and scalable but harder to revoke.

**OAuth 2.0 / OIDC:** delegated authorization ("Login with Google"), using authorization code flow with PKCE.

```js
const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: "15m" });
const decoded = jwt.verify(token, process.env.JWT_SECRET);
```

---

## Web Security

- **XSS (Cross-Site Scripting):** attacker injects script into your page. Prevent with output encoding/escaping, avoid `innerHTML`/`dangerouslySetInnerHTML` with untrusted data, sanitize (DOMPurify), **Content Security Policy**, `HttpOnly` cookies.
- **CSRF:** tricks a logged-in user's browser into sending an unwanted request. Prevent with CSRF tokens, `SameSite` cookies, checking origin.
- **SQL/NoSQL Injection:** use parameterized queries, validate input, avoid passing raw user objects to Mongo queries.
- **CORS misconfiguration, clickjacking** (`X-Frame-Options` / CSP `frame-ancestors`), **MITM** (HTTPS + HSTS), **dependency vulnerabilities** (`npm audit`).
- **OWASP Top 10** is the standard checklist.

---

## Web Performance

- **Core Web Vitals:** **LCP** (loading, < 2.5s), **INP** (interactivity, < 200ms), **CLS** (visual stability, < 0.1).
- **Techniques:** minify and compress (gzip/brotli), code splitting, lazy loading, image optimization, CDN, caching, preload/prefetch critical resources, reduce render-blocking CSS/JS, font optimization (`font-display: swap`), avoid layout shifts, Service Workers/PWA.
- **Tools:** Lighthouse, Chrome DevTools Performance tab, WebPageTest.

---

## Databases (MongoDB / SQL basics)

**SQL vs NoSQL:** SQL = tables, fixed schema, joins, ACID (PostgreSQL, MySQL). NoSQL (MongoDB) = flexible documents, horizontal scaling, denormalized.

**Indexes:** speed up reads, slow down writes, use memory; create them on frequently queried/sorted fields.

**Mongoose basics:**
```js
const userSchema = new Schema({
  name: { type: String, required: true },
  email: { type: String, unique: true },
}, { timestamps: true });
const User = model("User", userSchema);
await User.find({ role: "admin" }).select("name").limit(10).lean();
```

**Joins in Mongo:** `populate()` (Mongoose) or `$lookup` (aggregation).
**Normalization vs denormalization, transactions, pagination (offset vs cursor)**, SQL `JOIN` types (INNER, LEFT, RIGHT, FULL).

---

## TypeScript (commonly asked with React/Node)

- **`interface` vs `type`:** both describe shapes; interfaces can be extended/merged; types support unions, intersections and mapped types.
- **`any` vs `unknown`:** `unknown` is the type-safe version and requires narrowing.
- **Generics:** `function identity<T>(x: T): T { return x; }`
- **Utility types:** `Partial`, `Required`, `Pick`, `Omit`, `Record`, `ReturnType`.
- **Enums, union/literal types, type guards, `never`.**

```ts
interface User { id: number; name: string; email?: string; }
type Status = "idle" | "loading" | "error";
```

---

## Testing

- **Unit** (Jest/Vitest), **integration**, **end-to-end** (Cypress/Playwright).
- **React Testing Library:** test behavior as the user sees it (`getByRole`, `userEvent`), not implementation details.
- **API testing:** Supertest + Jest for Express routes.
- **Mock vs stub vs spy; TDD** (red → green → refactor).

---

## System Design / Architecture (basic)

- **Monolith vs microservices**, load balancing, horizontal vs vertical scaling
- **Caching layers** (browser, CDN, Redis), message queues (RabbitMQ, Kafka)
- **Rate limiting, pagination, idempotency, logging/monitoring**
- **CI/CD** (GitHub Actions), Docker basics (image vs container), environment variables, deployment (Vercel, Render, AWS)

---

## Common Coding Round Questions (practice)

```js
// 1. Reverse a string
const reverse = s => [...s].reverse().join("");

// 2. Palindrome
const isPalindrome = s => { const c = s.toLowerCase().replace(/[^a-z0-9]/g, ""); return c === [...c].reverse().join(""); };

// 3. Flatten an array
const flat = arr => arr.flat(Infinity);

// 4. Remove duplicates
const uniq = arr => [...new Set(arr)];

// 5. Two sum (O(n))
function twoSum(nums, target) {
  const seen = new Map();
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(nums[i], i);
  }
}

// 6. Promise.all polyfill
Promise.myAll = (promises) => new Promise((resolve, reject) => {
  const results = []; let done = 0;
  if (!promises.length) return resolve(results);
  promises.forEach((p, i) => Promise.resolve(p).then(v => {
    results[i] = v;
    if (++done === promises.length) resolve(results);
  }, reject));
});

// 7. Group by
const groupBy = (arr, key) => arr.reduce((acc, x) => ((acc[x[key]] ||= []).push(x), acc), {});

// 8. Deep clone (simple)
const clone = o => structuredClone(o);

// 9. Fibonacci with memoization
const fib = (n, memo = {}) => n <= 1 ? n : memo[n] ??= fib(n - 1, memo) + fib(n - 2, memo);
```

Also practice: `Array.prototype.map/filter/reduce` polyfills, `bind` polyfill, custom `debounce`, event emitter, LRU cache, a todo app, a debounced search with API, a infinite scroll, a modal/accordion/tabs component in React.

---

## Quick Tips for Interviews

- Explain with a **definition → example → use case/trade-off** structure.
- Be ready to write the code on a whiteboard/live (closures, debounce, promises, flatten, curry).
- Know your own **projects** deeply: architecture, challenges, performance and security decisions.
- Say "I don't know, but I'd approach it by..." rather than guessing.

---

*Good luck with your interviews!*
