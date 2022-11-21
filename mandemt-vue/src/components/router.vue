<script>
import Home from './Home.vue'
import D3 from './D3.vue'

import Rebrickable from './Rebrickable.vue'

const routes = {
    '/' : Home,
    '/d3' : D3,
    '/rebrickable' : Rebrickable,
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
    <a href="#/">Home</a> |
    <a href="#/d3">d3</a> |
    <a href="#/rebrickable">Rebrickable API</a> |
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
}</style>