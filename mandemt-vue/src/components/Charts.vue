<template>
    <h1 v-if="this.pieceCount">{{ this.pieceCount.name.value }} </h1>
    <h1 v-if="this.pieceCount">{{ this.pieceCount.value.value }} pieces </h1>
    <h1 v-if="this.pieceCount"> <a :href="this.pieceCount.url.value">link</a> </h1>
    <h1 v-if="this.pieceCount"> year: {{ this.pieceCount.year.value }} </h1>
    <div class="outer">
        <svg v-if="props.data" class="graph" :width="width" :height="height">
            <g class="bars">
                <rect @mouseover="changePiece()" v-for="d in data" :x="xScale(d.set_num) + 50"
                    :y="yScale(d.num_parts) - 100" :width="xScale.bandwidth()" :height="height - yScale(d.num_parts)"
                    :value="d.num_parts" :class="d.num_parts" :name="d.name" :url="d.set_url" :year="d.year" />
            </g>
        </svg>
    <section class="l-graph">
        <h4 v-if="this.details">{{ this.details }} </h4>
        <svg class="linegraph" :width="width" :height="height">

            <path v-if="yearline" class="line" :d="yearline"></path>
            
            
            <circle 
                @mouseover="showDetails()"
                v-for="j in jaren" 
                :cx="xLineScale(j.year)" 
                :cy="yLineScale(j.mean)"
                :r="5" fill="yellow" 
                :value="j.mean">
            </circle>
        </svg>

    </section>
    </div>

    <table>
        <tr>
            <th>set</th>
            <th>year</th>
            <th>pieces</th>
            <th>set number</th>
        </tr>
        <tr v-for="d in data">
            <td>
                <a :href="d.set_url">{{ d.name }}</a>
                <img :src="d.set_img_url">
            </td>
            <td>{{ d.year }}</td>
            <td>{{ d.num_parts }}</td>
            <td>{{ d.set_num }}</td>
        </tr>
    </table>

    <table>
        <tr>
            <th>year</th>
            <th>average pieces</th>
        </tr>
        <tr v-for="d in jaren">
            <td>{{ d.year }}</td>
            <td>{{ d.mean }}</td>
        </tr>
    </table>

</template>

<script setup>
// d3 functions
import { scaleBand, scaleLinear, max, select, path, scalePoint, axisBottom, axisLeft, line, min, selectAll, curveCardinal } from 'd3'
console.log('fasdjfasdf')
// define the data type
const props = defineProps({
    data: { type: Array, required: true },
    jaren: Array,
    naam: String
})

let data = props.data // all data from Rebrickable.vue
let jaren = props.jaren // the array containing the seperate years and mean amount of num_parts
console.log(data)

// settings for the horizontal and vertical axis
let height = 800;
let width = 800

// define the scales
const xScale =
    scaleBand()
        .range([0, width - 250]) //  the pixel range of the axis
        .domain(data.map((d) => d.set_num)) // the data range of the axis (amount of items)
        .padding(0.2) // the space between the values

const yScale =
    scaleLinear().domain([0, max(data, result => { // the highest value defines the data range of this axis.
        return result.num_parts + 300;
    })]).range([height, 0])

//create the path with labels
let xaxis = axisBottom(xScale)
console.log(xaxis)
let yaxis = axisLeft(yScale).tickSize(-width).ticks(20)

// scales for the line graph

const xLineScale = scalePoint()
    .range([100, width - 10]) //  the pixel range of the axis
    .domain(jaren.map((d) => d.year)) // the data range of the axis (amount of items)
    .padding(.9)

let maximum = max(jaren, result => { // the highest value defines the data range of this axis.
    return result.mean + 100
})

const yLineScale = scaleLinear()
    .domain([0, maximum])
    .range([height - 200, 0])

let xlineaxis = axisBottom(xLineScale).tickSize(-height).ticks(20)
let ylineaxis = axisLeft(yLineScale).tickSize(-width).ticks(20)

// the path for :d attribute in the path for the line graph
const yearline =
    line()
        .defined(function (d) { return d.mean !== null; })
        .y((d) => yLineScale(d.mean))
        .x((d) => xLineScale(d.year))(jaren)


// add the axes to the svg canvas
setTimeout(() => {
    let svg = select('.graph')
    svg.append("g")
        .attr("class", "domain")
        .attr("transform", "translate(50 , 700)")
        .call(xaxis)

    svg.append("g")
        .attr("class", "yaxis")
        .attr("transform", "translate(50, -100)")
        .call(yaxis)

        let linegraph = select('.linegraph',)
    linegraph.append("g")
        .attr("class", "domain")
        .attr("transform", "translate(0 , 600)")
        .call(xlineaxis)

    linegraph.append("g")
        .attr("class", "yaxis")
        .attr("transform", "translate(80 ,0)")
        .call(ylineaxis)
}, 200)
</script>
<script>

export default {
    
    data() {
        return {
            pieceCount: null, // shows when hovering over a bar
            details: null // shows when hovering over a line chart point
        }
    },

    methods: {
        changePiece() { // when hovering over a bar
            this.pieceCount = event.target.attributes
        },
        showDetails() { // when hovering over a line chart point
            this.details = event.target.attributes.value.value
        },
computed : {

},

    },
    mounted() {
        console.log('bar chart mounted')
    }
}





</script>
<style scoped lang="scss">
circle:hover {
    transition: .2s;
    r: 10;
}
.l-graph {
    position: relative;

}

h4 {
    text-align: center;
    color: white;
    position: absolute;
    top: 0;
    left: 100px;
}

table img {
    z-index: 8;
    visibility: hidden;
    position: absolute;
    top: -100px;
    padding: 3px;
    background-color: gray;
    right: 0;
    width: 250px;
    height: 250px;
    object-fit: cover;
}

td {
    position: relative;

    &:hover {
        background-color: lightpink;

        img {
            visibility: visible;
        }
    }
}

table a {
    text-decoration: NONE;
    font-weight: 800;
    color: black;
}

tr:nth-of-type(odd) {
    background-color: aqua;
}

tr:nth-of-type(even) {
    background-color: white;
}

.outer {
    display: flex;
    width: 800;
    height: 800;
    overflow: scroll;
}

path {
    fill: none;
    stroke-width: 2.5px;
    stroke: white;
}

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

#tooltip {
    top: 200px;
    color: white;
    left: 1050px;
    position: absolute;
}
</style>