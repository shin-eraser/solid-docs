[![Edit in Eraser](https://firebasestorage.googleapis.com/v0/b/second-petal-295822.appspot.com/o/images%2Fgithub%2FOpen%20in%20Eraser.svg?alt=media&token=968381c8-a7e7-472a-8ed6-4a6626da5501)](https://app.eraser.io/workspace/n09HU5pMkFr7MXTg5tvx)
# Reactivity
Solid's data management is built off a set of flexible reactive primitives which are responsible for all the updates. It takes a very similar approach to MobX or Vue except it never trades its granularity for a VDOM. Dependencies are automatically tracked when you access your reactive values in your Effects and JSX View code.

Solid's primitives come in the form of `create` calls that often return tuples, where generally the first element is a readable primitive and the second is a setter. It is common to refer to only the readable part by the primitive name.

Here is a basic auto incrementing counter that is updating based on setting the `count` signal.

```
import { createSignal, onCleanup } from "solid-js";
import { render } from "solid-js/web";

const App = () => {
  const [count, setCount] = createSignal(0),
    timer = setInterval(() => setCount(count() + 1), 1000);
  onCleanup(() => clearInterval(timer));

  return <div>{count()}</div>;
};

render(() => <App />, document.getElementById("app"));
```
## Introducing Primitives
![3 primitives](https://firebasestorage.googleapis.com/v0/b/second-petal-295822.appspot.com/o/images%2Fworkspaces%2Fn09HU5pMkFr7MXTg5tvx%2FreS6fUv66LcKWYn8yV2OvCPvwSm2%2F---figure---k8yH6rL0P5OhbY_NBF3k1---figure---M6hl7Qq2PTo03VLyFKBTfQ.svg?alt=media&token=69f14c1d-46f5-4105-9f26-d6528cc15440 "3 primitives")

Solid is made up of 3 primary primitives: Signal, Memo, and Effect. At their core is the Observer pattern where Signals (and Memos) are tracked by wrapping Memos and Effects.

Signals are the most core primitive. They contain value, and get and set functions so we can intercept when they are read and written to.

```
const [count, setCount] = createSignal(0);
```
Effects are functions that wrap reads of our signal and re-execute whenever a dependent Signal's value changes. This is useful for creating side effects, like rendering.

```
createEffect(() => console.log("The latest count is", count()));
```
Finally, Memos are cached derived values. They share the properties of both Signals and Effects. They track their own dependent Signals, re-executing only when those change, and are trackable Signals themselves.

```
const fullName = createMemo(() => `${firstName()} ${lastName()}`);
```
## How it Works
Signals are event emitters that hold a list of subscriptions. They notify their subscribers whenever their value changes.

Where things get more interesting is how these subscriptions happen. Solid uses automatic dependency tracking. Updates happen automatically as the data changes.

The trick is a global stack at runtime. Before an Effect or Memo executes (or re-executes) its developer-provided function, it pushes itself on to that stack. Then any Signal that is read checks if there is a current listener on the stack and if so adds the listener to its subscriptions.

You can think of it like this:

```
function createSignal(value) {
  const subscribers = new Set();

  const read = () => {
    const listener = getCurrentListener();
    if (listener) subscribers.add(listener);
    return value;
  };

  const write = (nextValue) => {
    value = nextValue;
    for (const sub of subscribers) sub.run();
  };

  return [read, write];
}
```
Now whenever we update the Signal we know which Effects to re-run. Simple yet effective. The actual implementation is much more complicated but that is the guts of what is going on.

For more detailed understanding of how Reactivity works these are useful articles:

[﻿A Hands-on Introduction to Fine-Grained Reactivity](https://dev.to/ryansolid/a-hands-on-introduction-to-fine-grained-reactivity-3ndf) 

[﻿Building a Reactive Library from Scratch](https://dev.to/ryansolid/building-a-reactive-library-from-scratch-1i0p) 

[﻿SolidJS: Reactivity to Rendering](https://indepth.dev/posts/1289/solidjs-reactivity-to-rendering) 

<!--- Eraser file: https://app.eraser.io/workspace/n09HU5pMkFr7MXTg5tvx --->
