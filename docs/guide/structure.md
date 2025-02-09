# Application Structure

Vuex doesn't really restrict how you structure your code. Rather, it enforces a set of high-level principles:

1. <span class='definition'>Application-level state</span> is centralized in the store.

2. <span class='definition'>The only way to mutate the state</span> is by committing **mutations**, which are synchronous transactions.

3. <span class='definition'>Asynchronous logic</span> should be encapsulated in, and can be composed with **actions**.

As long as you follow these rules, it's up to you how to structure your project. <span class='important'>If your store file gets too big, simply start splitting the actions, mutations and getters into separate files</span>.

For any non-trivial app, we will likely need to leverage <span class='definition'>modules</span>. Here's an example project structure:

```bash
├── index.html
├── main.js
├── api
│   └── ... # abstractions for making API requests
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # where we assemble modules and export the store
    ├── actions.js        # root actions
    ├── mutations.js      # root mutations
    └── modules
        ├── cart.js       # cart module
        └── products.js   # products module
```

As a reference, check out the [Shopping Cart Example](https://github.com/vuejs/vuex/tree/dev/examples/shopping-cart).
