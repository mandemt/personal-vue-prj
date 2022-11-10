<template>
    <svg class="graph" :width="width" :height="height" viewBox="0 -10 800 700">
        <g>
            <rect v-for="d in data" 
                :x="xScale(d.set_num)" 
                :y="yScale(d.num_parts)"
                :width="xScale.bandwidth()" 
                :height="height - yScale(d.num_parts)" 
                class="coffee-bar" 
                fill="blue"
            >
            </rect>
        </g>
    </svg>
    <div v-for="d in data" :top="xScale(d.set_num)">{{ d.name + d.num_parts }}</div>
</template>

<script setup>
// d3 functions
import { scaleBand, scaleLinear, max, select, axisBottom, axisLeft } from 'd3'

// define the data type
const props = defineProps({
    data: Array,
})

let data = props.data

//sizing of the svg canvas
const width = 800
const height = 600;

// settings for the horizontal and vertical axis
const xScale = scaleBand()
    .range([0, width]) //  the pixel range of the axis
    .domain(data.map((d) => d.set_num)) // the data range of the axis (amount of items)
    .padding(0.2) // the space between the values

const yScale = scaleLinear().domain([0, max(data, result => { // the highest value defines the data range of this axis.
    return result.num_parts + 50;
})]).range([height, 0])

//create the path with labels
let xaxis = axisBottom(xScale)
let yaxis = axisLeft(yScale).ticks(20, ",f")

// add the axes to the svg canvas
setTimeout(() => {
    let svg = select('.graph')
    svg.append("g")
        .attr("class", "domain")
        .attr("transform", "translate(0 , 600)")
        .call(xaxis)

    svg.append("g")
        .attr("class", "yaxis")
        .attr("transform", "translate(0 , 0)")
        .call(yaxis)
}, 200)

</script>

<style scoped>
svg {
    margin: 0 auto;
    padding: 50px;
}

rect {
    fill: #78e08f;
}

rect:hover {
    fill: #079992;
}
</style>