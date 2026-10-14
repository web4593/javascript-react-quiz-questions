# JavaScript and React Quiz Questions, Explained

Twelve "predict the output" questions about how JavaScript and React actually behave, each with the answer, a short explanation and a link to the official documentation.

These questions are about runtime behaviour more than syntax: what `typeof` returns, when a `ReferenceError` fires, the order `await` resumes in, and why React sometimes skips a re-render. They are meant for self-assessment, revision before a technical interview, or starting a discussion in a study group.

> The questions in this repository are original to this resource. They are not taken from any product's question bank, and no product endorses them.

## Contents

- [How to use this resource](#how-to-use-this-resource)
- [Concepts covered](#concepts-covered)
- [JavaScript questions (1–9)](#javascript-questions)
- [React questions (10–12)](#react-questions)
- [Summary table](#summary-table)
- [Scoring yourself](#scoring-yourself)
- [More practice](#more-practice)
- [Official documentation](#official-documentation)

## How to use this resource

1. Read the question and the code. Don't run it yet.
2. Write down your answer **and** how confident you are (sure / fairly sure / guessing).
3. Open the collapsed answer and read the explanation.
4. Run the snippet yourself in Node.js 18+ or a browser console to confirm it.

A correct guess and a correct explanation are different results. If you picked the right option but couldn't have explained why, count it as a gap.

**Assumptions:** JavaScript snippets run as ES modules (strict mode) in Node.js 18+ or a modern browser. React snippets assume React 18 or 19 with `createRoot`.

## Concepts covered

| Area | Concepts |
|---|---|
| Types and coercion | `typeof`, the `+` operator with arrays and strings, `-` forcing numeric conversion |
| Scope and hoisting | `var` hoisting vs the temporal dead zone of `let` |
| Asynchronous JavaScript | Where an `async` function pauses, `Promise.allSettled` |
| Arrays and objects | `map` callback arguments, `reduce` without an initial value, shallow copies with spread |
| `this` | Plain function callbacks in strict mode |
| React | State identity and `Object.is`, `&&` in JSX, `useLayoutEffect` vs `useEffect` |

---

## JavaScript questions

### 1. What does `typeof` report?

**Concept:** primitive types and `typeof`

```js
console.log(typeof null, typeof NaN, typeof []);
```

- A. `null NaN array`
- B. `object number object`
- C. `object NaN object`
- D. `null number array`

<details>
<summary>Answer</summary>

**B. `object number object`**

`typeof null` returning `"object"` is a long-standing quirk kept for backwards compatibility. `NaN` is a value of the Number type, so `typeof NaN` is `"number"`. Arrays are objects; use `Array.isArray()` to detect them.

[MDN: typeof](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof)
</details>

### 2. `var` vs `let` before declaration

**Concept:** hoisting and the temporal dead zone

```js
console.log(a);
var a = 1;

console.log(b);
let b = 2;
```

- A. Logs `undefined`, then `undefined`
- B. Logs `undefined`, then throws `ReferenceError`
- C. Throws `ReferenceError` on the first line
- D. Logs `1`, then `2`

<details>
<summary>Answer</summary>

**B. Logs `undefined`, then throws `ReferenceError`**

Both declarations are hoisted, but differently. A `var` binding is created and initialised to `undefined` at the start of its scope, so reading it early gives `undefined`. A `let` binding exists from the start of its block but stays uninitialised until the declaration line runs. Reading it in that window (the temporal dead zone) throws a `ReferenceError`.

[MDN: let — temporal dead zone](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz)
</details>

### 3. Where does an `async` function pause?

**Concept:** `async`/`await` and the microtask queue

```js
async function run() {
  console.log(1);
  await null;
  console.log(2);
}

run();
console.log(3);
```

- A. `1 2 3`
- B. `3 1 2`
- C. `1 3 2`
- D. `1 2` then `3` after a timer tick

<details>
<summary>Answer</summary>

**C. `1 3 2`**

An `async` function runs synchronously until its first `await`. At `await null`, the rest of the function is scheduled as a microtask and control returns to the caller, which logs `3`. When the current script finishes, the microtask queue runs and logs `2`. Awaiting a non-promise value still yields.

[MDN: await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
</details>

### 4. `map` with `parseInt`

**Concept:** callback arguments

```js
console.log(['1', '2', '3'].map(parseInt));
```

- A. `[1, 2, 3]`
- B. `[1, NaN, NaN]`
- C. `[NaN, NaN, NaN]`
- D. `['1', '2', '3']`

<details>
<summary>Answer</summary>

**B. `[1, NaN, NaN]`**

`map` calls its callback with `(element, index, array)`, and `parseInt` accepts `(string, radix)`. The calls are therefore `parseInt('1', 0)`, `parseInt('2', 1)` and `parseInt('3', 2)`. A radix of `0` is treated as "not specified", so the first gives `1`. Radix `1` is invalid, so the second gives `NaN`. In base 2 the digit `3` doesn't exist, so the third gives `NaN`. Use `.map(Number)` or `.map((s) => parseInt(s, 10))` instead.

[MDN: parseInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt)
</details>

### 5. `reduce` on an empty array

**Concept:** `reduce` without an initial value

```js
const total = [].reduce((sum, n) => sum + n);
console.log(total);
```

- A. `0`
- B. `undefined`
- C. `NaN`
- D. It throws `TypeError`

<details>
<summary>Answer</summary>

**D. It throws `TypeError`**

Without an initial value, `reduce` uses the first element as the starting accumulator. An empty array has no first element, so it throws `TypeError: Reduce of empty array with no initial value`. Passing an initial value, as in `[].reduce((sum, n) => sum + n, 0)`, returns `0` safely. This bug often appears in code that works in testing and fails when a filtered list comes back empty.

[MDN: Array.prototype.reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
</details>

### 6. `+` and `-` with mixed types

**Concept:** type coercion

```js
console.log([1, 2] + [3], 1 + '2' - 1);
```

- A. `[1, 2, 3]` and `2`
- B. `1,23` and `11`
- C. `1,23` and `'111'`
- D. `6` and `2`

<details>
<summary>Answer</summary>

**B. `1,23` and `11`**

`+` converts objects to primitives first. Arrays become strings (`"1,2"` and `"3"`), and once either side is a string, `+` concatenates: `"1,23"`. In the second expression, `1 + '2'` is string concatenation and gives `'12'`. `-` has no string meaning, so `'12' - 1` converts to a number and gives `11`.

[MDN: Addition (+)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Addition)
</details>

### 7. Spread copies one level

**Concept:** shallow vs deep copies

```js
const original = { tags: ['js'] };
const copy = { ...original };

copy.tags.push('react');
console.log(original.tags.length);
```

- A. `1`
- B. `2`
- C. `undefined`
- D. It throws `TypeError`

<details>
<summary>Answer</summary>

**B. `2`**

Object spread creates a new outer object, but it copies property values as they are. `copy.tags` and `original.tags` point to the same array, so pushing through one is visible through the other. For a deep copy of plain data, use `structuredClone(original)`.

[MDN: structuredClone](https://developer.mozilla.org/en-US/docs/Web/API/Window/structuredClone)
</details>

### 8. `this` inside a plain function callback

**Concept:** `this` binding in strict mode

```js
// Runs as an ES module, so strict mode applies.
const counter = {
  count: 0,
  start() {
    [1, 2, 3].forEach(function () {
      this.count++;
    });
    return this.count;
  },
};

console.log(counter.start());
```

- A. `3`
- B. `0`
- C. `NaN`
- D. It throws `TypeError`

<details>
<summary>Answer</summary>

**D. It throws `TypeError`**

A regular `function` gets its own `this`, decided by how it is called. `forEach` calls the callback without a receiver, and in strict mode that leaves `this` as `undefined`, so `this.count++` throws. An arrow function has no `this` of its own and inherits it from `start()`. With `forEach(() => { this.count++; })` the result is `3`.

(In sloppy-mode scripts, `this` would be the global object instead. No error is thrown and `counter.count` stays `0`, which is why the assumption is stated.)

[MDN: this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
</details>

### 9. `Promise.allSettled` statuses

**Concept:** handling partial failure

```js
Promise.allSettled([
  Promise.resolve(1),
  Promise.reject(new Error('network')),
  3,
]).then((results) => {
  console.log(results.map((r) => r.status));
});
```

- A. It rejects with `Error: network`
- B. `['fulfilled', 'rejected', 'fulfilled']`
- C. `['fulfilled', 'rejected']`
- D. `['fulfilled', 'rejected', 'pending']`

<details>
<summary>Answer</summary>

**B. `['fulfilled', 'rejected', 'fulfilled']`**

`Promise.allSettled` waits for every input and never rejects because one input failed. Each result is `{ status: 'fulfilled', value }` or `{ status: 'rejected', reason }`. A plain value such as `3` counts as an already-fulfilled promise. Compare `Promise.all`, which rejects as soon as any input rejects.

[MDN: Promise.allSettled](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)
</details>

---

## React questions

### 10. Mutating state, then setting it

**Concept:** state identity and `Object.is`

```jsx
function Profile() {
  const [user, setUser] = useState({ name: 'Ada' });

  function rename() {
    user.name = 'Grace';
    setUser(user);
  }

  return <button onClick={rename}>{user.name}</button>;
}
```

After one click, what does the button show?

- A. `Grace`
- B. `Ada`
- C. Nothing; React throws an error
- D. An empty button

<details>
<summary>Answer</summary>

**B. `Ada`**

React compares the new state with the old one using `Object.is`. Here they are the same object reference, so React skips the re-render and the screen keeps showing `Ada`, even though the object now holds `'Grace'`. Create a new object instead: `setUser({ ...user, name: 'Grace' })`.

[React: Updating objects in state](https://react.dev/learn/updating-objects-in-state)
</details>

### 11. `&&` with a number in JSX

**Concept:** conditional rendering

```jsx
function Inbox() {
  const count = 0;
  return <div>{count && <p>You have {count} messages</p>}</div>;
}
```

What renders inside the `<div>`?

- A. Nothing
- B. `0`
- C. `You have 0 messages`
- D. `false`

<details>
<summary>Answer</summary>

**B. `0`**

`count && <p>…</p>` returns the left side when it is falsy, so the expression evaluates to `0`. React skips `false`, `null` and `undefined`, but it does render numbers, so a stray `0` appears. Make the condition a boolean, as in `count > 0 && …`, or use a ternary.

[React: Conditional rendering](https://react.dev/learn/conditional-rendering)
</details>

### 12. Measuring before paint

**Concept:** `useLayoutEffect` vs `useEffect`

You need to read a tooltip's rendered height and reposition it before the user can see it in the wrong place. Which hook fits?

- A. `useEffect`
- B. `useLayoutEffect`
- C. `useMemo`
- D. `useCallback`

<details>
<summary>Answer</summary>

**B. `useLayoutEffect`**

`useLayoutEffect` runs after React has updated the DOM but before the browser repaints, so a measure-then-adjust step happens without a visible flicker. `useEffect` usually runs after paint, which can briefly show the wrong position. `useMemo` and `useCallback` cache values and functions; they don't run side effects. Prefer `useEffect` by default, since `useLayoutEffect` blocks painting.

[React: useLayoutEffect](https://react.dev/reference/react/useLayoutEffect)
</details>

---

## Summary table

| # | Topic | Skill being tested | Why it matters |
|---|---|---|---|
| 1 | `typeof` | Knowing which values report as which type | Type checks such as `typeof x === 'object'` let `null` through |
| 2 | Hoisting and TDZ | Telling `var` initialisation apart from `let` | Explains "Cannot access before initialization" errors |
| 3 | `async`/`await` | Knowing where an async function yields | Predicting log order and race conditions in async code |
| 4 | `map` + `parseInt` | Reading callback signatures | Passing a function reference can pass extra arguments |
| 5 | `reduce` | Handling the empty-input case | Prevents crashes when a filtered list is empty |
| 6 | Coercion | Applying `+` vs `-` conversion rules | Debugging string-number bugs from form inputs and query params |
| 7 | Spread | Shallow vs deep copying | Avoiding accidental shared state, especially in React |
| 8 | `this` | Function vs arrow callback binding | A common source of `undefined` errors in methods |
| 9 | `Promise.allSettled` | Choosing a combinator for partial failure | Running independent requests where some may fail |
| 10 | React state identity | Immutable updates | Explains "I set state but nothing re-rendered" |
| 11 | JSX `&&` | Which falsy values React renders | Stops a stray `0` appearing in the UI |
| 12 | `useLayoutEffect` | Effect timing relative to paint | Avoids layout flicker in tooltips, popovers and measurements |

## Scoring yourself

Group your misses by concept, not by question number:

- **Missed 1, 6:** review type conversion; the [MDN guide to equality and sameness](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness) is a good next read.
- **Missed 2, 8:** review scope and `this`; [MDN on closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) covers the related scope rules.
- **Missed 3, 9:** review the event loop and microtasks; see the [MDN microtask guide](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide).
- **Missed 4, 5, 7:** reread the array method and copying docs linked above, and test the edge cases yourself.
- **Missed 10–12:** read the React docs pages on [state as a snapshot](https://react.dev/learn/state-as-a-snapshot) and [effects](https://react.dev/reference/react/useEffect).

Questions you answered correctly but marked "guessing" go on the same list. Come back to them in a few days and see whether you can explain the answer without looking.

## More practice

The questions above are a small, hand-picked set. For a larger bank, [JourneyUncommon](https://journeyuncommon.com/) is a developer interview-practice site with multiple-choice questions organised by language and topic. Its [JavaScript practice questions](https://journeyuncommon.com/javascript-mcq) and [React practice questions](https://journeyuncommon.com/react-mcq) show the code on the page. A free account lets you answer, see your score and track accuracy by topic, and questions you get wrong come back later for review.

## Official documentation

- [MDN Web Docs: JavaScript reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference)
- [MDN: Using microtasks in JavaScript](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide)
- [React docs: useState](https://react.dev/reference/react/useState)
- [React docs: useEffect](https://react.dev/reference/react/useEffect)
- [React docs: useLayoutEffect](https://react.dev/reference/react/useLayoutEffect)
- [ECMAScript language specification](https://tc39.es/ecma262/)

## Contributing

Found an answer that's wrong under the stated assumptions, or an explanation that could be clearer? Open an issue with the snippet, your runtime (Node.js or browser version) and what you observed.

## License

The text and code in this repository are licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). You can share and adapt them for any purpose, including commercially, as long as you give appropriate credit and link back to this repository.
