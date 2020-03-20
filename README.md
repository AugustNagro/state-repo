## state-repo.js
Here's the big idea. State Management shouldn't need such complex libraries.
State Management should be simple, easy, and fun.

Published on NPM at [state-repo](https://www.npmjs.com/package/state-repo). Check out the examples:

* Hello World [source](https://github.com/AugustNagro/state-repo/tree/master/examples/hello-world), and [live](https://august.nagro.us/examples/hello-world/)
* Counters [source](https://github.com/AugustNagro/state-repo/tree/master/examples/counter), and [live](https://august.nagro.us/examples/counter/)

```javascript
// imports the global state repository, but you can create your own.
import { repo } from './state-repo.js';

// create a StateWriter with an initial state.
// The initial state is an optional parameter.
const writer = repo.writer('USER_STATE',
  { firstName: 'Rocco', lastName: 'Nagro' });

// create a StateReader with a changeListener
const reader = repo.reader('USER_STATE', newVal => updateDom(newVal));

// this will fire updateDom with the new state
writer.set(userState => {
  userState.firstName = 'John';
  return userState;
});

// or, don't pass a function and directly set
writer.setValue(newUserState);

// elsewhere in your app
function updateSomethingElse(newVal, reader) {
  if (newVal.firstName === 'Leon') reader.unregister();
  someOtherCode();
}
repo.reader('USER_STATE', this.updateSomethingElse);
```

So there you go. A StateRepo is a repository of State, and if the global repository
`repo` won't cut it then you can

```javascript
import { StateRepo } from 'state-repo.js';
const newRepo = new StateRepo();
```

and make your own. The State object is an implementation detail, but know that it has four
properties: a unique identifier, a value, a set of StateReaders that hold change-listeners,
and at most one StateWriter that executes the listeners when `set(...)` is called.
StateWriters and StateReaders are returned from a StateRepository's `writer(id, initialValue)`
and `reader(id, changeListener)` methods, respectively. A StateReader may be `unregister()`ed
and then re-`register()`ed at any time, but when a writer unregisters it is permanent.
The StateRepo throws an exception if a StateWriter is already registered for a given State.

The [source code](https://github.com/AugustNagro/state-repo/blob/master/state-repo.js) is tiny and well documented.

### Comparison with "Props Down, Events Up"
Some already say that state management is simple, if you just use the browser's
tools and practice "props down, events up." But this paradigm quickly gets painful
when one node nested in the Document Tree needs to read the same state as
one which is not an immediate ancestor. And you can still run into one of
computer science's classic banes, the multiple writers problem. Even when using a single
thread (and avoiding async/await), directly updating a shared state by disparate parts
of code is a recipie for disaster or at the very least confusion.

### Comparison with Redux
Redux solves the multiple writers problem if you can get past its tedious boilerplate
and the performance impact of cloning an entire state object every time you
need to update one of it's properties.

Redux is actually a good framework, but is overkill for the vast majority of
applications. And the biggest benefit it provides (for all the boiler-plate
one needs to write) is the ability to track and even replay state changes.
But state-repo.js avoids the multiple writers problem too, by only allowing the
creation of one writer for a given state. And these writers are meant to be
encapsulated, not shared across components. If updating the same state
from disparate components is necessary, a pattern similar to Redux's action
definitions can be used:

file: `states/user-state.js`

```javascript
import { repo } from 'state-repo.js';

export const USER_STATE = 'USER_STATE';
// note that writer is not exported
const writer = repo.writer(USER_STATE, {});

export const userChangeName = (first, last) => writer.set(userState => {
  userState.first = first;
  userState.last = last;
  return userState;
});

export const userMakeBirthdayToday = () => writer.set(userState => {
  userState.birthday = new Date();
  return userState;
});
```

See the [Counter Demo](https://github.com/AugustNagro/state-repo/tree/master/examples/counter) source for a complete example.

Sure, state-repo.js isn't purely functional like Redux, but good encapsulations is sufficient;
JavaScript isn't a pure language to begin with. And while the common case is not multiple-writers,
it's easy to refactor to the 'action' pattern.

So just like those seeking to resolve a standardization problem end up adding
yet another competing standard, I too present a new library to solve
a solved problem. But at least this solution is small enough to copy/paste.
