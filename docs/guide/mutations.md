# Mutations

<div class="scrimba"><a href="https://scrimba.com/p/pnyzgAP/ckMZp4HN" target="_blank" rel="noopener noreferrer">Try this lesson on Scrimba</a></div>

<span class='important'>The only way to actually change state in a Vuex store is by committing a mutation</span>. Vuex mutations are very similar to events: each mutation has a string **type** and a **handler**. The <span class='definition'>handler function</span> is where we perform actual state modifications, and it will receive the state as the first argument:

```js
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // mutate state
      state.count++
    }
  }
})
```

<span class='important'>You cannot directly call a mutation handler</span>. Think of it more like event registration: "When a mutation with type `increment` is triggered, call this handler." To invoke a mutation handler, you need to call `store.commit` with its type:

```js
store.commit('increment')
```

## Commit with Payload

You can pass an additional argument to `store.commit`, which is called the <span class='definition'>**payload** for the mutation</span>:

```js
// ...
mutations: {
  increment (state, n) {
    state.count += n
  }
}
```

```js
store.commit('increment', 10)
```

In most cases, <span class='important'>the payload should be an object</span> so that it can contain multiple fields, and the recorded mutation will also be more descriptive:

```js
// ...
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

```js
store.commit('increment', {
  amount: 10
})
```

## Object-Style Commit

An alternative way to commit a mutation is by directly using an object that has a `type` property:

```js
store.commit({
  type: 'increment',
  amount: 10
})
```

When using <span class='definition'>object-style commit</span>, the entire object will be passed as the payload to mutation handlers, so the handler remains the same:

```js
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

## Mutations Follow Vue's Reactivity Rules

Since a <span class='important'>Vuex store's state is made reactive by Vue</span>, when we mutate the state, Vue components observing the state will update automatically. This also means <span class='definition'>Vuex mutations are subject to the same reactivity caveats</span> when working with plain Vue:

1. Prefer <span class='important'>initializing your store's initial state with all desired fields upfront</span>.

2. <span class='definition'>When adding new properties to an Object</span>, you should either:

  - Use <span class='definition'>`Vue.set(obj, 'newProp', 123)`</span>, or

  - <span class='definition'>Replace that Object</span> with a fresh one. For example, using the [object spread syntax](https://github.com/tc39/proposal-object-rest-spread) we can write it like this:

    ```js
    state.obj = { ...state.obj, newProp: 123 }
    ```

## Using Constants for Mutation Types

It is a commonly seen pattern to use constants for mutation types in various Flux implementations. This allows the code to take advantage of tooling like linters, and putting all constants in a single file allows your collaborators to get an at-a-glance view of what mutations are possible in the entire application:

```js
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'
```

```js
// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.Store({
  state: { ... },
  mutations: {
    // we can use the ES2015 computed property name feature
    // to use a constant as the function name
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```

Whether to use constants is largely a preference - it can be helpful in large projects with many developers, but it's totally optional if you don't like them.

## Mutations Must Be Synchronous

One important rule to remember is that <span class='important'>**mutation handler functions must be synchronous**</span>. Why? Consider the following example:

```js
mutations: {
  someMutation (state) {
    api.callAsyncMethod(() => {
      state.count++
    })
  }
}
```

Now imagine we are debugging the app and looking at the devtool's mutation logs. For every mutation logged, <span class='important'>the devtool will need to capture a "before" and "after" snapshots of the state</span>. However, the asynchronous callback inside the example mutation above makes that impossible: the callback is not called yet when the mutation is committed, and there's no way for the devtool to know when the callback will actually be called - <span class='important'>any state mutation performed in the callback is essentially un-trackable</span>!

## Committing Mutations in Components

You can commit mutations in components with `this.$store.commit('xxx')`, or use the `mapMutations` helper which maps component methods to `store.commit` calls (requires root `store` injection):

```js
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // map `this.increment()` to `this.$store.commit('increment')`

      // `mapMutations` also supports payloads:
      'incrementBy' // map `this.incrementBy(amount)` to `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // map `this.add()` to `this.$store.commit('increment')`
    })
  }
}
```

## On to Actions

Asynchronicity combined with state mutation can make your program very hard to reason about. For example, when you call two methods both with async callbacks that mutate the state, how do you know when they are called and which callback was called first? This is exactly why we want to separate the two concepts. <span class='definition'>In Vuex, **mutations are synchronous transactions**</span>:

```js
store.commit('increment')
// any state change that the "increment" mutation may cause
// should be done at this moment.
```

To handle asynchronous operations, let's introduce [Actions](actions.md).
