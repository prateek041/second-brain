# Introduction to Vue

## Single File Component (SFC)

An SFC as the name suggests encapsulates the component's logic (JavaScript), template (HTML) and Styles (CSS).

## API Styles

There are two ways to author Vue Components

- Options API
- Composition API

### Options API

The component logic is defined using options in an object, for example

```vue
<script>
export default {
  // properties returned from data() become reactive state and will be exposed on `this`
  data() {
    return {
      count: 0,
    };
  },

  // Methods are functions that mutate the state and trigger updates.
  // They can be bound as even handlers in templates.
  methods: {
    increment() {
      this.count++;
    },
  },

  // Lifecycle hooks are called at different stages of a component's lifecycle.
  // This function will be called when the component is mounted.
  mounted() {
    console.log(`The initial count is ${this.count}.`);
  },
};
</script>

<template>
  <button @click="increment">Count is : {{ count }}</button>
</template>
```

### Composition API

here we use imported API functions. top level variables and functions can be directly used in the template.

here is the same componenet built using Composition API

```vue
<script setup>
import {ref, onMounted} from 'vue'

// reactive count
const count = ref(0)

// function that mutates the state and triggers updates
function increment(){
  count.value++
}

// lifecycle hooks
onMounted(()=>{
  console.log(`The initial count is ${{count.value}}`)
})
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

> [!TODO] Read about how to use TypeScript and Component API.
