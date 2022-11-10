<script>
import Home from './Home.vue'
// import D3 from '../../research/D3.vue'

import Rebrickable from './Rebrickable.vue'

const routes = {
    '/' : Home,
    // '/d3' : D3,
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

    <component :is="currentView" />
  </template>