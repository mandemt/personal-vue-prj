<template>
    <div class="allcharts">
        <section class="barchartsection">
            <ul>
                <li>  <h4> {{ details.name }}</h4></li>
                <li>  <h4> <a :href="details.set_url">Link</a></h4></li>
                <li>  <h4> {{ details.set_num }}</h4></li>
                <li>  <h4> {{ details.num_parts }} pieces</h4></li>
            </ul>
            <svg v-if="this.data" class="bargraph" :width="width + 100" :height="630">
                <g class="bars">
                    <rect v-for="d in data" v-on:mouseover="details = d" :x="xScale(d.set_num) + 50"
                        :y="yScale(d.num_parts) - 100" :width="xScale.bandwidth()"
                        :height="height - yScale(d.num_parts)" :value="d.num_parts" :class="d.num_parts" :name="d.name"
                        :url="d.set_url" :year="d.year" />
                </g>
                <g class="domain">
                    {{ barAxis }}
                </g>
            </svg>
        </section>

        <section class="linechartsection">
        <svg v-if="this.data" class="linegraph" :width="width" :height="height">
            <g cass="line">
                <path :d="valueline" />
                <circle v-for="j in calculated" v-on:mouseover="line = j" :cx="xLineScale(j.year)" :cy="yLineScale(j.mean) - 100" :r="7"
                    fill="yellow" :value="j.mean">
                </circle>
            </g>
            <g class="linedomain">
                {{ lineAxis }}
            </g>
        </svg>
        <ul>
                <li>  <h4> Year: {{ line.year}}</h4></li>
                <li>  <h4> Average amount of pieces in a set: {{ line.mean }} pieces</h4></li>
            </ul>
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
        <tr v-for="d in calculated">
            <td>{{ d.year }}</td>
            <td>{{ d.mean }}</td>
        </tr>
    </table>
</template>

<script>

import { scaleBand, scaleLinear, max, select, path, scalePoint, axisBottom, axisLeft, line, min, selectAll, curveStep } from 'd3'
import D3 from './D3.vue'
export default {
    data() {
        return {
            details: [],
            line: [],
            width: 800,
            height: 700
        }
    },
    props: ['data', 'calculated'],
    mounted() {
    },
    computed: {
        fixedData() {
            return this.data
        },
        xScale() {
            const xScale = scaleBand()
                .range([0, this.width]) //  the pixel range of the axis
                .domain(this.data.map((d) => d.set_num)) // the data range of the axis (amount of items)
                .padding(.1) // the space between the values
            return xScale;
        },
        yScale() {
            console.log(this.data)
            const yScale =
                scaleLinear().domain([0, max(this.data, result => { // the highest value defines the data range of this axis.
                    return result.num_parts + 250;
                })]).range([this.height, 0])
            return yScale;
        },
        barAxis() {
            selectAll(".axis")
                .remove()
            let graph = select('.bargraph')

            graph
                .append('g')
                .attr('class', 'axis')
                .attr('transform', 'translate(50, 600)')
                .call(axisBottom(this.xScale))

            graph
                .append('g')
                .attr('class', 'axis')
                .attr('transform', 'translate(50, -100)')
                .call(axisLeft(this.yScale).tickSize(-(this.width)).ticks(20))


        },
        lineAxis() {
            selectAll(".lineaxis")
                .remove()

            let linegraph = select('.linegraph')

            linegraph
                .append('g')
                .attr('class', 'lineaxis')
                .attr('transform', 'translate(0, 600)')
                .call(axisBottom(this.xLineScale))

            linegraph
                .append('g')
                .attr('class', 'lineaxis')
                .attr('transform', 'translate(50, -100)')
                .call(axisLeft(this.yLineScale).tickSize(-this.width).ticks(20))
        },
        calculatedData() {
            console.log(this.calculated)
            return this.calculated;
        },
        yLineScale() {
            let maximum = max(this.calculated, result => { // the highest value defines the data range of this axis.
                return result.mean + 10
            })
            let minimum = min(this.calculated, result => { // the highest value defines the data range of this axis.
                return result.mean 
            })
            console.log(minimum)
            return scaleLinear()
                .domain([minimum, maximum + 10])
                .range([this.height, 100])

        },
        xLineScale() {
            return scalePoint()
                .range([50, this.width]) //  the pixel range of the axis
                .domain(this.calculated.map((d) => d.year)) // the data range of the axis (amount of items)
                .padding(.9)

        },

        // this line is for the linechart, succeeded by accident
        valueline() {
            let thisline = line()
                .defined(function (d) { return d.mean !== null; })
                .curve(curveStep)
                .y((d) => this.yLineScale(d.mean) - 100)
                .x((d) => this.xLineScale(d.year))(this.calculated)

            return thisline;
        },



    },

}


</script>
<style scoped lang="scss">
.barchartsection {
    display: flex;
    flex-direction: column-reverse;
    align-items: center;
}
circle{
    z-index: 3;
}
.bargraph {
    background-color: #222f3e;
}

ul {
    width: 700px;
    list-style-type: none;
    background-color: white;
    border-radius: 10px;
    padding: 0 30px;
    top: 10px;
    right: calc(50% + 80px);
    display: flex;
    flex-wrap: wrap;

    li {
        width: 350px;
        padding: 0;
        color: black;
        text-align: center;
    }
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

.allcharts {
    background-color: #222f3e;
    color: white;
    font-size: 13px;
    padding-top: 100px;
    display: flex;
    justify-content: space-around;
}

.bars rect {
    transition: .2s ease;
}

rect {
    fill: #78e08f;
}

rect:hover {
    fill: #78e08f9e;

}

path {
    fill: none;
    stroke: white;
    stroke-width: 5px;
}
</style>