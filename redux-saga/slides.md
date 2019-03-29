---
slideOptions:
  transition: fade
  controls: false
---

<!--
    Marcus: Intro, Controlling a Saga, Advantages, Fazit
    Moritz: Generators, Testing
    Leo: Example + Demo, More Saga Effects
-->

<style>
img {
    border: none !important;
    width: 30vh;
    object-fit: contain;
    background: none !important;
    box-shadow: none !important;
}

p code:not(.hljs),
li code:not(.hljs) {
    color: #dc8c8c;
}

.reveal blockquote {
    padding: 0 25px;
}

.profilepics > div {
    display: flex;
    justify-content: space-around;
}

.profilepics img {
    border-radius: 50%;
    height: 25vh;
    width: 25vh;
    object-fit: cover;
    max-width: 100%;
}

.profilepics h2 {
    font-size: 3vh;
    margin-bottom: 0;
}

.profilepics h3 {
    font-size: 2vh;
}

.hljs {
    font-size: 80%;
}
</style>

# Redux Saga

#### Transparent Asynchronous State Changes

*React Meetup Düsseldorf*

---

<!-- .slide: class="profilepics" -->

<div class="image">
    <img src="https://i.imgur.com/m7xBGhd.jpg" width="300" height="300">
    <h2>Leo Bernard</h2>
    <h3>github.com/leolabs</h3>
</div>

<div class="image">
    <img src="https://i.imgur.com/WIwJk9U.jpg" width="300" height="300">
    <h2>Marcus Weiner</h2>
    <h3>github.com/mraerino</h3>
</div>

<div class="image">
    <img src="https://i.imgur.com/vE8DKi4.jpg" width="300" height="300">
    <h2>Moritz Gunz</h2>
    <h3>github.com/NeoLegends</h3>
</div>

---

### Nice to have

- ES2015
- Redux knowledge
- Test Assertions

Note:
Marcus

---

# 😅
## Finding a way to do async things with Redux

Note:
- Reducers are synchronous
- IRL ist aber alles async

---

### Cases

- Data fetching
- Data subscriptions
- Retry, Cancel, Debounce
- Rollbacks / Optimistic updates
- Integrating async actions into the redux state machine

---

# 🎉
## Saga

---

> [...] a Saga is like a separate thread in your application that's solely responsible for side effects.

---

# ☸️
## What Can a Saga Do?

---

- Await events
- Interoperate with Redux
- Manage other sagas

---

# 🏭
## Generators

Note:
Moritz
- Wer kennt das?

---

```javascript
function* generator() { /* ... */ }

const iterator = generator()
```

```javascript
iterator.next() => { value: T | undefined, done: bool }
```
<!-- .element: class="fragment" -->

```javascript
for (const item of iterator) { // calls iterator.next()
  console.log(item)
}
```
<!-- .element: class="fragment" -->

Note:
- works with for...of
- function only runs on call to .next()

---

```javascript
function* generator() {
  const num = 2

  // Pass value to callee and pause execution
  yield 1

  // Callee can pass values into generator
  let a = yield 2

  // `num` maintains value across yield points
  yield a + num
}
```

```javascript







```
<!-- .element: class="fragment" -->

Note:
- Yield -> resumable function
- Try-catch / if-else, etc.
- Push values into generator

---

```javascript
function* generator() {
  const num = 2

  // Pass value to callee and pause execution
  yield 1

  // Callee can pass values into generator
  let a = yield 2

  // `num` maintains value across yield points
  yield a + num
}
```

```javascript
const iterator = generator()

// Generator progress controlled from outside
iterator.next().value   == 1
iterator.next().value   == 2
iterator.next(10).value == 12
iterator.next().done    == true
```
<!-- .element: class="fragment" -->

---

# ⚡️
## Effects

Note:
Leo
- "Glue" between generators and saga library
  - -> Way to control sagas

---

```javascript
function* saga() {
  yield put({
    type: "SET_JOKE",
    payload: "What do you call a fake noodle? An impasta!",
  })
}
```

```javascript
saga().next() == {
  PUT: {
    action: {
      type: "SET_JOKE",
      payload: "What do you call a fake noodle? An impasta!",
    }
  }
}
```
<!-- .element: class="fragment" -->

---

# 💻
## Example

---

```javascript
function* fetchJoke() {
  const { joke } = yield call(
    fetchJson,
    "https://icanhazdadjoke.com/",
  )
  yield put({ type: 'SET_JOKE', payload: joke })
}
```

```javascript
function* jokeThread() {
  while (true) {
    const action = yield take('GET_JOKE')
    yield fork(fetchJoke)
  }
}
```
<!-- .element: class="fragment" -->

Note:
- Why infinite loop?

---

## DEMO

https://codesandbox.io/s/1q5856pww4

---

# 😍
## Why is Saga so great?

Note:
Marcus

---

### Model the Whole Story of Your App

**App Lifecycle**
Login → Interaction → Logout → Login ...

```javascript
function* app() {
  yield* login()
  yield* displayData()
  yield* logout()
}
```
<!-- .element: class="fragment" -->

Note:
yield*

---

# ✅
## Testing our Saga

---

```javascript
it("should fetch a joke and store it in the state", () => {
  const jokeMock = {
    joke: "Chuck Norris knows the last digit of pi."
  }

  const gen = fetchJoke()
  gen.next() // returns { CALL: { ... } } effect object

  expect(gen.next(jokeMock).value).toEqual(put({
    type: "SET_JOKE",
    payload: jokeMock.joke,
  }))
  expect(gen.next())
    .toEqual({ done: true, value: undefined })
})
```

---

# ✨
## More Saga Effects

Note:
Leo

---

### `cancel(saga)`

```javascript
const saga = yield fork(handler)
yield delay(1000)
yield cancel(saga)
```

---

### `takeEvery(type, handler)`

fork `handler` for every action of `type`
(allow parallelism)

---

### `takeLatest(type, handler)`

like `takeEvery`, but run handler for last action only (cancel previous)

---

# 💡
## Advanced use cases

---

#### Debounce

```javascript
function* handleInput({ payload }) {
  yield delay(500)

  const res = yield call(
    fetch,
    "https://myAPI",
    { body: payload },
  )
}

function* watchInput() {
  yield takeLatest('INPUT_CHANGED', handleInput)
}
```

---

## Retries

---

### Worker State

- Redux store in every worker<!-- .element: class="fragment" -->
- Dispatch every action across workers (`postMessage`)<!-- .element: class="fragment" -->
- Guarantees consistent state across workers<!-- .element: class="fragment" -->

---

# 🤷‍
## Why Saga?

---

- Pull Model
- Ideally Pure
- Sagas Run Sequentially

---

![Festify Logo](https://d33wubrfki0l68.cloudfront.net/1967c0923ec2039f845c917dae55fa32f0b73e74/ca447/img/festify-logo.svg)
## [github.com/Festify/app](https://github.com/Festify/app)

---

# ❤️
## Thank you!
## [festify.rocks](https://festify.rocks)