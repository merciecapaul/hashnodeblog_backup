## What’s Coming To VueX

Quick summary:
> 
Vuex is a state management pattern + library for Vue. js applications. It serves as a centralised store for all the components in an application, with rules ensuring that the state can only be mutated in a predictable fashion.

The next version of Vuex 4, is making its way through the final steps before officially releasing. This release will bring full compatibility with Vue 3, but doesn’t add new features. While Vuex has always been a powerful solution, and the first choice for many developers for state management in Vue, some developers had hoped to see more workflow issues addressed.

With the advent of Vue 3 and it’s  [composition API](https://v3.vuejs.org/guide/composition-api-introduction.html#why-composition-api) , people have been looking into hand-built simple alternatives. For example,  [You Might Not Need Vuex](https://dev.to/blacksonic/you-might-not-need-vuex-with-vue-3-52e4)  demonstrates a relatively simple, yet flexible and robust pattern for using the composition API along with `provide/inject` to create shared state stores. This and other alternatives should only be used in smaller applications because they lack all those things that aren’t directly about the code: community support, documentation, conventions, good  [Nuxt](https://nuxtjs.org/)  integrations, and developer tools.

### Defining a Store
Before we can do anything with a Vuex store, we need to define one. In Vuex 4, a store definition will look like this:
```
import { createStore } from 'vuex'

export const counterStore = createStore({
  state: {
    count: 0
  },
  
  getters: {
    double (state) {
      return state.count * 2
    }
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

Each store has four parts: `state` stores the data, `getters` give you computed state, `mutations` are used to mutate the state, and `actions` are the methods that are called from outside the store to accomplish anything related to the store. Usually, actions don’t just commit a mutation as this example shows. Instead, they are used to do asynchronous tasks because mutations must be synchronous or they just implement more complicated or multi-step functionality.

```
import { defineStore } from 'vuex'

export const counterStore = defineStore({
  name: 'counter',
  
  state() {
    return { count: 0 }
  },
  
  getters: {
    double () {
      return this.count * 2
    }
  },
  
  actions: {
    increment () {
      this.count++
    }
  }
})
```

First, instead of `createStore`, we use `defineStore`. This difference is negligible, but it’s there for semantic reasons, which we’ll go over later. Next, we need to provide a `name` for the store, which we didn’t need before. In the past, modules got their own name, but they weren’t provided by the module itself; they were just the property name they were assigned to by the parent store that added them. Now, there are no modules. Instead, each module will be a separate store and have a name.

After that, we need to make `state` a function that returns the initial state instead of just setting it to the initial state. This is similar to the `data` option on components. We write `getters` very similar to the way we did in Vuex 4, but instead of using the `state` as a parameter for each getter, you can just use `this` to get to the state. In the same way, `actions` don’t need to worry about a `context` object being passed in: they can just use `this` to access everything. Finally, there are no `mutations`. Instead, mutations are combined with `actions`.

### Initiate the Store
In Vuex 4, things have changed from Vuex 3, but I’ll just look at v4 to keep things from getting out of hand. In v4, when you called `createStore`, you already instantiated it. You can then just use it in your app, either via `app.use` or directly:
```
import { createApp } from 'vue'
import App from './App.vue' // Your root component
import store from './store' // The store definition from earlier

const app = createApp(App)

app.use(store)
app.mount('#app')

// Now all your components can access it via `this.$store`
// Or you can use in composition components with `useStore()`

// -----------------------------------------------

// Or use directly... this is generally discouraged
import store from './store'

store.state.count // -> 0
store.commit('increment')
store.dispatch('increment')
store.getters.double // -> 4
```
This is one thing that Vuex 5 makes a bit more complicated than in v4. Each app now can get a separate instance of Vuex, which makes sure that each app can have separate instances of the same stores without sharing data between them. You can share an instance of Vuex if you want to share instances of stores between apps.
```
import { createApp } from 'vue'
import { createVuex } from 'vuex'
import App from './App.vue' // Your root component

const app = createApp(App)
const vuex = createVuex() // create instance of Vuex

app.use(vuex) // use the instance
app.mount('#app')
```
Now all of your components have access to the Vuex instance. Instead of giving your store(s) definition directly, you then import them into the components you want to use them in and use the Vuex instance to instantiate and register them:
```
import { defineComponent } from 'vue'
import store from './store'

export default defineComponent({
  name: 'App',

  computed: {
    counter () {
      return this.$vuex.store(store)
    }
  }
})
```
Calling `$vuex.store`, instantiates and registers the store in the Vuex instance. From that point on, any time you use `$vuex.store` on that store, it’ll give you back the already instantiated store instead of instantiating it again. You can call the `store` method straight on an instance of Vuex created by `createVuex()`.
Now your store is accessible on that component via `this.counter`. If you’re using the composition API for your component, you can use `useStore` instead of `this.$vuex.store`:
```
import { defineComponent } from 'vue'
import { useStore } from 'vuex' // import useStore
import store from './store'

export default defineComponent({
  setup () {
    const counter = useStore(store)

    return { counter }
  }
})
```

### Use the Store
Here’s what it looks like to use a store in Vuex 4.
```
store.state.count            // Access State
store.getters.double         // Access Getters
store.commit('increment')    // Mutate State
store.dispatch('increment')  // Run Actions
```
`State`, `getters`, `mutations`, and `actions` are all handled in different ways via different properties or methods. This has the advantage of explicitness, which I praised earlier, but this explicitness doesn’t really gain us anything.

Everything — the state, getters and actions — is available directly at the root of the store, making it simple to use with a lot less verbosity and practically removes all need for using `mapState`, `mapGetters`, `mapActions` and `mapMutations` for the options API.

### Composing Stores
The final aspect of Vuex 5 we’ll look at today is composability. Vuex 5 doesn’t have namespaced modules that are all accessible from the single store. Each of those modules would be split into a completely separate store. In v4, the namespacing convolutes the whole thing, so you need to use the namespace in your `commit` and `dispatch` calls, use `rootGetters` and `rootState` and then work your way up into the namespaces you want to access getters and state from. Here’s how it works in Vuex 5:
```
// store/greeter.js
import { defineStore } from 'vuex'

export default defineStore({
  name: 'greeter',
  state () {
    return { greeting: 'Hello' }
  }
})

// store/counter.js
import { defineStore } from 'vuex'
import greeterStore from './greeter' // Import the store you want to interact with

export default defineStore({
  name: 'counter',

  // Then `use` the store
  use () {
    return { greeter: greeterStore }
  },
  
  state () {
    return { count: 0 }
  },
  
  getters: {
    greetingCount () {
      return `${this.greeter.greeting} ${this.count}' // access it from this.greeter
    }
  }
})
```