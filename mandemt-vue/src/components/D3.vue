<template>
  <h2>welkom</h2>
    <section>
        {{naam}}
   <button @click="changeValues()"></button>
      <div id="dia">
        <svg width="400" height="300">
          <g v-for="(value, index) in tom" :key="value">
            <rect v-if="tom" 
              :x="index * 50"
              y="10"
              width="25"
              :height="value "
              fill="steelblue"
            ></rect>
          </g>
        </svg>
      </div>
   
    </section>
  </template>
<script >
export default {
    props: ['naam'],
  name: 'VisualComponent',
  data: function() {
    return {
      tom: []
    }
  },
  methods: {
    changeValues() {
this.tom = [10,20,30,40]
    }
    }
}
let width = 400;
let color = 'orange'


</script>
<style scoped>

</style> 