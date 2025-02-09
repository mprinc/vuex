# Getters

<div class="scrimba"><a href="https://scrimba.com/p/pnyzgAP/c2Be7TB" target="_blank" rel="noopener noreferrer">Try this lesson on Scrimba</a></div>

Sometimes we may need to <span class='definition'>compute derived state based on store state</span>, for example filtering through a list of items and counting them:

``` js
computed: {
  doneTodosCount () {
    return this.$store.state.todos.filter(todo => todo.done).length
  }
}
```

If more than one component needs to make use of this, we have to either duplicate the function, or extract it into a shared helper and import it in multiple places - both are less than ideal.

Vuex allows us to define <span class='definition'>"getters"</span> in the store. You can think of them as <span class='definition'>computed properties for stores</span>. Like computed properties, a getter's result is <span class='definition'>cached</span> based on its dependencies, and <span class='important'>will only re-evaluate when some of its dependencies have changed</span>.

Getters will receive the state as their 1st argument:

``` js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

## Property-Style Access

The getters will be exposed on the `store.getters` object, and you access values as properties:

``` js
store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]
```

Getters will also receive other getters as the 2nd argument:

``` js
getters: {
  // ...
  doneTodosCount: (state, getters) => {
    return getters.doneTodos.length
  }
}
```

``` js
store.getters.doneTodosCount // -> 1
```

We can now easily make use of it inside any component:

``` js
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}
```

Note that getters accessed as properties are <span class='definition'>cached as part of Vue's reactivity system</span>.

## Method-Style Access

You can also <span class='definition'>pass arguments to getters</span> by returning a function. This is particularly useful when you want to query an array in the store:

```js
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}
```

``` js
store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
```

Note that getters accessed via methods will run each time you call them, and <span class='important'>the result is not cached</span>.

## The `mapGetters` Helper

The `mapGetters` helper simply maps store getters to local computed properties:

``` js
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
    // mix the getters into computed with object spread operator
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}
```

If you want to <span class='definition'>map a getter to a different name</span>, use an object:

``` js
...mapGetters({
  // map `this.doneCount` to `this.$store.getters.doneTodosCount`
  doneCount: 'doneTodosCount'
})
```
