# Actions

<div class="scrimba"><a href="https://scrimba.com/p/pnyzgAP/c6ggR3cG" target="_blank" rel="noopener noreferrer">Try this lesson on Scrimba</a></div>

Actions are similar to mutations, the differences being that:

- Instead of mutating the state, <span class='definition'>actions commit mutations</span>.
- <span class='important'>Actions can contain arbitrary asynchronous operations</span>.

Let's register a simple action:

``` js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```

Action handlers receive a <span class='definition'>context object</span> which exposes the same set of methods/properties on the store instance, so you can call `context.commit` to commit a mutation, or access the state and getters via `context.state` and `context.getters`. <span class='important'>We can even call other actions with `context.dispatch`</span>. We will see why this context object is not the store instance itself when we introduce [Modules](modules.md) later.

In practice, we often use <span class='definition'>ES2015 [argument destructuring](https://github.com/lukehoban/es6features#destructuring)</span> to simplify the code a bit (especially when we need to call `commit` multiple times):

``` js
actions: {
  increment ({ commit }) {
    commit('increment')
  }
}
```

## Dispatching Actions

<span class='definition'>Actions are triggered with the `store.dispatch` method</span>:

``` js
store.dispatch('increment')
```

This may look silly at first sight: if we want to increment the count, why don't we just call `store.commit('increment')` directly? Remember that **mutations have to be synchronous**. Actions don't. We can perform **asynchronous** operations inside an action:

``` js
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```

Actions support the same <span class='definition'>payload format</span> and <span class='definition'>object-style dispatch</span>:

``` js
// dispatch with a payload
store.dispatch('incrementAsync', {
  amount: 10
})

// dispatch with an object
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```

A more practical example of real-world actions would be an action to checkout a shopping cart, which involves **calling an async API** and **committing multiple mutations**:

``` js
actions: {
  checkout ({ commit, state }, products) {
    // save the items currently in the cart
    const savedCartItems = [...state.cart.added]
    // send out checkout request, and optimistically
    // clear the cart
    commit(types.CHECKOUT_REQUEST)
    // the shop API accepts a success callback and a failure callback
    shop.buyProducts(
      products,
      // handle success
      () => commit(types.CHECKOUT_SUCCESS),
      // handle failure
      () => commit(types.CHECKOUT_FAILURE, savedCartItems)
    )
  }
}
```

Note we are performing <span class='definition'>a flow of asynchronous operations</span>, and <span class='definition'>recording the side effects (state mutations)</span> of the action by committing them.

## Dispatching Actions in Components

You can dispatch actions in components with `this.$store.dispatch('xxx')`, or use the `mapActions` helper which maps component methods to `store.dispatch` calls (requires root `store` injection):

``` js
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // map `this.increment()` to `this.$store.dispatch('increment')`

      // `mapActions` also supports payloads:
      'incrementBy' // map `this.incrementBy(amount)` to `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // map `this.add()` to `this.$store.dispatch('increment')`
    })
  }
}
```

## Composing Actions

Actions are often asynchronous, so how do we know when an <span class='important'>action is done</span>? And more importantly, how can we <span class='important'>compose multiple actions together</span> to handle more complex async flows?

The first thing to know is that `store.dispatch` can handle <span class='definition'>Promise returned by the triggered action handler</span> and it also returns Promise:

``` js
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}
```

Now you can do:

``` js
store.dispatch('actionA').then(() => {
  // ...
})
```

And also in another action:

``` js
actions: {
  // ...
  actionB ({ dispatch, commit }) {
    return dispatch('actionA').then(() => {
      commit('someOtherMutation')
    })
  }
}
```

Finally, if we make use of [async / await](https://tc39.github.io/ecmascript-asyncawait/), we can compose our actions like this:

``` js
// assuming `getData()` and `getOtherData()` return Promises

actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // wait for `actionA` to finish
    commit('gotOtherData', await getOtherData())
  }
}
```

> It's possible for a `store.dispatch` to <span class='important'>trigger multiple action handlers in different modules</span>. In such a case the returned value will be a Promise that resolves when all triggered handlers have been resolved.
