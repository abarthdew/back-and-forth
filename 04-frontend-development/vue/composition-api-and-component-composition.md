Example codes both of these two things:

## Composition API:

The Composition API is a new way of organizing and reusing code in Vue.js 3. It provides a set of functions that allow developers to create reusable logic that can be used across multiple components. The Composition API is designed to help developers build complex applications by providing a more organized way of managing component state and logic.

### Here's an example code using the Composition API:

```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <button @click="increment">Increment</button>
    <p>Count: {{ count }}</p>
  </div>
</template>

<script>
import { reactive, computed } from 'vue';

export default {
  setup() {
    const state = reactive({
      count: 0,
    });

    const title = computed(() => `Count is ${state.count}`);

    const increment = () => {
      state.count++;
    };

    return { state, title, increment };
  },
};
</script>
```

In this example, we use the reactive function to create a reactive object, which we can then use to manage component state. We also use the computed function to create a computed property, which we can use to derive data from our component's state. Finally, we use the setup function to return the state, computed properties, and methods that our component needs to function.

## Component Composition:

Component composition is a way of building components by composing smaller, reusable components together. In Vue.js 2, component composition is achieved using slots and props. By using slots and props, developers can create reusable components that can be combined in different ways to create more complex components.

### Here's an example code using component composition:

```vue
<template>
  <div>
    <slot :count="count" :increment="increment" />
  </div>
</template>

<script>
import { reactive } from 'vue';

export default {
  props: {
    count: {
      type: Number,
      required: true,
    },
  },
  setup(props) {
    const state = reactive({
      count: props.count,
    });

    const increment = () => {
      state.count++;
    };

    return { state, increment };
  },
};
</script>
```

In this example, we use a slot to allow the parent component to provide the content of the component. We also use props to pass data from the parent component to the child component. Finally, we use the setup function to return the state and methods that our component needs to function.

Overall, the Composition API provides a more organized way of managing component state and logic, while component composition provides a way to build components by composing smaller, reusable components together.
<br/>
<br/>

> What does `Composition` mean?


**Composition** is a programming paradigm that involves building complex structures or functionalities by combining simpler building blocks or components. In composition, the focus is on creating components that can be reused across different parts of the application rather than building large monolithic structures. Composition enables developers to create more flexible, maintainable, and scalable code by breaking down complex functionalities into smaller, reusable components.

In object-oriented programming, composition is achieved by combining objects to create more complex objects, rather than inheriting behavior from parent classes. In functional programming, composition is achieved by composing smaller functions to create more complex functions. In both cases, composition involves building complex functionalities by combining simpler building blocks.

Composition is a fundamental concept in many programming languages and frameworks, including Vue.js, React, and Angular. In these frameworks, composition enables developers to build complex UIs by combining smaller, reusable components together. The Composition API in Vue.js 3, for example, provides a way to build more flexible and maintainable components by allowing developers to organize component logic into reusable functions.
