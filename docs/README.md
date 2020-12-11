# What is Vuex?

::: tip NOTE
This is the docs for Vuex 3, which works with Vue 2. If you're looking for docs for Vuex 4, which works with Vue 3, [please check it out here](https://next.vuex.vuejs.org/).
:::

<VideoPreview />

<span class='definition'>Vuex is a **state management pattern + library** for Vue.js applications</span>. It serves as a <span class='important'>centralized store for all the components</span> in an application, with <span class='definition'>rules</span> ensuring that the state can only be mutated in a <span class='important'>predictable fashion</span>. It also integrates with Vue's official [devtools extension](https://github.com/vuejs/vue-devtools) to provide advanced features such as <span class='definition'>zero-config time-travel debugging</span> and <span class='definition'>state snapshot export / import</span>.

## What is a "State Management Pattern"?

Let's start with a simple Vue counter app:

```js
new Vue({
  // state
  data () {
    return {
      count: 0
    }
  },
  // view
  template: `
    <div>{{ count }}</div>
  `,
  // actions
  methods: {
    increment () {
      this.count++
    }
  }
})
```

It is a self-contained app with the following parts:

- The **state**, the source of truth that drives our app;
- The **view**, a declarative mapping of the **state**;
- The **actions**, the possible ways the state could change in reaction to user inputs from the **view**.

This is a simple representation of the concept of <span class='definition'>"one-way data flow"</span>:

<p style="text-align: center; margin: 2em">
  <img style="width:100%;max-width:450px;" src="/flow.png">
</p>

However, <span class='important'>the simplicity quickly breaks down when we have **multiple components that share a common state**:</span>

- Multiple views may depend on the same piece of state.
- Actions from different views may need to mutate the same piece of state.

For problem one, <span class='important'>passing props can be tedious for deeply nested components</span>, and <span class='important'>simply doesn't work for sibling components</span>. For problem two, we often find ourselves resorting to solutions such as reaching for <span class='definition'>direct parent/child instance references</span> or trying to mutate and synchronize multiple copies of the state via <span class='definition'>events</span>. Both of these patterns are <span class='important'>brittle and quickly lead to unmaintainable code</span>.

So why don't we <span class='definition'>extract the shared state out of the components, and manage it in a global singleton</span>? With this, our component tree becomes a big "view", and any component can access the state or trigger actions, no matter where they are in the tree!

By <span class='definition'>defining and separating</span> the concepts involved in state management and <span class='definition'>enforcing rules</span> that maintain independence between views and states, <span class='important'>we give our code more structure and maintainability</span>.

This is <span class='important'>the basic idea behind Vuex, inspired by</span> [Flux](https://facebook.github.io/flux/docs/overview), [Redux](http://redux.js.org/) and [The Elm Architecture](https://guide.elm-lang.org/architecture/). Unlike the other patterns, Vuex is also a library implementation <span class='important'>tailored specifically for Vue.js</span> to take advantage of its <span class='definition'>granular reactivity system</span> for efficient updates.

If you want to learn Vuex in an interactive way you can check out this [Vuex course on Scrimba](https://scrimba.com/g/gvuex), which gives you a mix of screencast and code playground that you can pause and play around with anytime.

![vuex](/vuex.png)

## When Should I Use It?

Vuex helps us deal with shared state management with <span class='important'>the cost of more concepts and boilerplate</span>. It's a <span class='important'>trade-off between short term and long term productivity</span>.

If you've never built a large-scale SPA and jump right into Vuex, it may feel verbose and daunting. That's perfectly normal - if your app is simple, you will most likely be fine without Vuex. A simple [store pattern](https://vuejs.org/v2/guide/state-management.html#Simple-State-Management-from-Scratch) may be all you need. But if you are building a medium-to-large-scale SPA, chances are you have run into situations that make you think about how to better handle state outside of your Vue components, and Vuex will be the natural next step for you. There's a good quote from Dan Abramov, the author of Redux:

> Flux libraries are like glasses: you’ll know when you need them.
