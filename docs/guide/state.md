# State

## Single State Tree

<div class="scrimba"><a href="https://scrimba.com/p/pnyzgAP/cWw3Zhb" target="_blank" rel="noopener noreferrer">Try this lesson on Scrimba</a></div>

Vuex uses a <span class='definition'>**single state tree** - that is, this single object contains all your application level state and serves as the "single source of truth."</span> This also means usually you will have only one store for each application. A single state tree makes it straightforward to locate a specific piece of state, and allows us to easily take snapshots of the current app state for debugging purposes.

<span class='important'>The single state tree does not conflict with modularity</span> - in later chapters we will discuss how to split your state and mutations into sub modules.

The data you store in Vuex follows the same rules as the `data` in a Vue instance, ie the state object must be plain. **See also:** [Vue#data](https://vuejs.org/v2/api/#data).

## Getting Vuex State into Vue Components

So how do we display state inside the store in our Vue components? Since Vuex stores are reactive, the simplest way t<span class='important'>o "retrieve" state from it is simply returning some store state from within a [computed property](https://vuejs.org/guide/computed.html)</span>:

```js
// let's create a Counter component
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
```

Whenever `store.state.count` changes, it will cause the computed property to re-evaluate, and trigger associated DOM updates.

However, <span class='important'>this pattern causes the component to rely on the <span class='definition'>global store singleton</span></span>. When using a module system, it requires importing the store in every component that uses store state, and also requires mocking when testing the component.

Vuex provides a mechanism to <span class='definition'>"inject" the store into all child components</span> from the root component with the `store` option (enabled by `Vue.use(Vuex)`):

```js
const app = new Vue({
  el: '#app',
  // provide the store using the "store" option.
  // this will inject the store instance to all child components.
  store,
  components: { Counter },
  template: `
    <div class="app">
      <counter></counter>
    </div>
  `
})
```

By providing the `store` option to the root instance, the store will be injected into all child components of the root and <span class='definition'>will be available on them as `this.$store`</span>. Let's update our `Counter` implementation:

```js
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```

## The `mapState` Helper

<div class="scrimba"><a href="https://scrimba.com/p/pnyzgAP/c8Pz7BSK" target="_blank" rel="noopener noreferrer">Try this lesson on Scrimba</a></div>

When a component needs to make use of multiple store state properties or getters, declaring all these computed properties can get repetitive and verbose. To deal with this we can make use of <span class='important'>the `mapState` helper which generates computed getter functions for us</span>, saving us some keystrokes:

```js
// in full builds helpers are exposed as Vuex.mapState
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // arrow functions can make the code very succinct!
    count: state => state.count,

    // passing the string value 'count' is same as `state => state.count`
    countAlias: 'count',

    // to access local state with `this`, a normal function must be used
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

We can also pass a string array to `mapState` when the name of a mapped computed property is the same as a state sub tree name.

```js
computed: mapState([
  // map this.count to store.state.count
  'count'
])
```

## Object Spread Operator

Note that `mapState` returns an object. How do we use it in combination with other local computed properties? Normally, we'd have to use a utility to merge multiple objects into one so that we can pass the final object to `computed`. However with the [object spread operator](https://github.com/tc39/proposal-object-rest-spread), we can greatly simplify the syntax:

```js
computed: {
  localComputed () { /* ... */ },
  // mix this into the outer object with the object spread operator
  ...mapState({
    // ...
  })
}
```

## Components Can Still Have Local State

<span class='important'>Using Vuex doesn't mean you should put **all** the state in Vuex</span>. Although putting more state into Vuex makes your state mutations more explicit and debuggable, sometimes it could also make the code more verbose and indirect. <span class='important'>If a piece of state strictly belongs to a single component, it could be just fine leaving it as local state</span>. You should weigh the trade-offs and make decisions that fit the development needs of your app.
