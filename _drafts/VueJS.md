---
layout: post
title: Why we chose for Vue.js
categories: front-end
tags: [Vue,front-end,framework]
author: jeroenrinzema
---

At Sysco we have recently started using Vue.js. Vue is an amazing, powerfull and fun to work with front-end framework. In this blog post we will describe why we have chosen Vue and why it can also be a great choice for your project(s).

## What did we use before

Much of our previous work was made in Angular. And although, Angular is a great front-end framework, it also has its con's. One of our main issues with Angular is its slow learning curve.

It was very difficult for developers that had little to none front-end experience to make simple changes, corrections or building an additional modules.

Therefore we started looking for alternatives. We required a framework that had a short learning curve and could be used for a wide variety of projects.

Vue was for us a perfect fit. Its small size (~30kb gzipped) and how it handles code separation meant that any developer with an understanding of JavaScript and HTML could then work on our front-end applications.

## Learning curve

From a business perspective Vue is a great choice. Developers do not need to have extensive knowledge about building systems or a new syntax. Vue would feel natural to any developer that has an understanding of JavaScript and HTML.

It also typically takes developers less then a day of reading the guide to learn enough to start working on open tasks/issues.

This makes hiring new developer(s) easier. But also allows you to let other developers without any previous framework experience work on front-end tasks when needed.

**A Vue single file component**

```html
<template> <!-- The component template -->
  <div>
    <ul class="users"> <!-- Print all users -->
      <li v-for="user of users">{{user.name}}</li>
    </ul>
  </div>
</template>

<script> // The component logics
import {mapGetters} from 'vuex'

export default {
  computed: {
    ...mapGetters({
      users: 'users/active' // Get all active users
    })
  },
  async created () {
    await this.$store.dispatch('fetchActiveUsers') // preform a Vuex action
  }
}
</script>

<style scoped> /* The scoped component style */
.users > li {
  font-size: 18px;
}
</style>
```

## Scaling

While Vue scales up, it also scales down just as well as jQuery. You can start using Vue with a single script tag.

`<script src="https://cdn.jsdelivr.net/npm/vue"></script>`

Then you can start writing Vue code and even ship the minified version to production without feeling guilty or having to worry about performance problems.

A typical Vue project is also extremely small. A project with Vue and Vue Router included is around 30kb gzipped. Even though that Vue is small in size is it packed with features. This allows you to use Vue in all sorts of applications. From small widgets to big enterprise projects.

Projects generated with the new cli (v3) also support TypeScript. TypeScript has its benefits, static type checking can be very useful for large-scale applications, and can be a big productivity boost for developers with backgrounds in Java and C#.

## State management

Vue has like React it's own state manager called [Vuex](https://github.com/vuejs/vuex).

> Vuex is a state manager that deeply integrates into Vue. It serves as a centralised store for all components in a application, with rules ensuring that the state can only be mutated in a predictable fashion.

When working with a couple of stores or hundreds. Vuex has a well documented [application structure](https://vuex.vuejs.org/en/structure.html) that enforces a set of principals.

When a new developer joins on a project does it require a minimum amount of time for him/her to understand the projects states, getters, actions and mutations.

```
└── store                 # directory where all stores are located
    ├── index.js          # where we assemble modules and export the store
    ├── actions.js        # root actions
    ├── mutations.js      # root mutations
    └── modules
        ├── cart.js       # cart module
        └── products.js   # products module
```

## Upgradable & flexibility

The new Vue cli (v3) allowes you also to easily upgrade the build process of your Vue project. This is huge since previously you were not able to do this with any JavaScript framework once you modified the build process.

With the new cli are all build processes treated as modules. You can extend, build or modify them while still keeping the plausibility to upgrade your project once a new version comes out.

Vue is also less opinionated about the used building system. Offering official support for a variety of build systems.

## Conclusion

Vue is a framework that is all about simplicity and development speed. When working alone or in a big team Vue gives you a great framework with a set of rules that get's you started quickly.

We will continue posting our experiences with Vue and create a followup post after a while.

If you want to read a detailed comparison with Vue and other frameworks. Then is the [Vue comparison guide](https://vuejs.org/v2/guide/comparison.html) is a great place to get started.
