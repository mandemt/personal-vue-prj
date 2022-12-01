<script>
import D3 from './D3.vue'

import Rebrickable from './Rebrickable.vue'

const routes = {
    '/' : Rebrickable,
}
export default {
  data() {
    return {
      currentPath: window.location.hash
    }
  },
  computed: {
    currentView() {
      return routes[this.currentPath.slice(1) || '/'] || NotFound
    }
  },
  mounted() {
    window.addEventListener('hashchange', () => {
		  this.currentPath = window.location.hash
		})
  }
}
</script>
<template>
  <header>
    <a href="#/">Home</a> |
    <a href="https://github.com/mandemt/personal-vue-prj">Github</a>
</header>

<Suspense>
<Transition name="fade">

    <component :is="currentView" />
    </Transition>
  </Suspense>
  </template>
  <style >
.fade-enter-from,
.fade-leave-to {
opacity: 0;
}
.fade-enter-active,
.fade-leave-active {
transition: opacity .2s ease;
}
a{
  color :white;
  text-decoration: none;
}</style>