<template>
    <div class="allcharts">
        <section class="barchartsection">
            <ul>
                <li>  <h4> {{ details.name }}</h4></li>
                <li>  <h4> <a :href="details.set_url">Link</a></h4></li>
                <li>  <h4> {{ details.set_num }}</h4></li>
                <li>  <h4> {{ details.num_parts }} pieces</h4></li>
            </ul>
            <svg v-if="this.data" class="bargraph" :width="width + 100" :height="730">
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
            <svg v-if="this.data" class="linegraph" :width="width" :height="630">

                <g class="line">
                    <path :d="valueline" />
                    <circle v-for="dot in calculated" 
                        v-on:mouseover="line = dot" 
                        :cx="xLineScale(dot.year)" 
                        :cy="yLineScale(dot.mean) - 100" 
                        :r="7"
                        fill="yellow" 
                        :value="dot.mean">
                    </circle>
                </g>
                <g class="linedomain">
                    {{ lineAxis }}
                </g>
            </svg>

            <ul>
                <li><h4> 
                        Year: {{ line.year}}
                    </h4>
                </li>

                <li>
                    <h4>
                        Average amount of pieces in a set: {{ line.mean }} pieces
                    </h4>
                </li>

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
export default {
    data() {
        // information for the hover event and canvas sizes
        return {
            details: [],
            line: [],
            width: 800,
            height: 700
        }
    },
    props: ['data', 'calculated'],

    computed: {
        // the data for the charts
        fixedData() {
            return this.data
        },

        // set up the range and domain for both axes for the bar chart
        xScale() {
            const xScale = scaleBand()
                .range([0, this.width]) //  the pixel range of the axis
                .domain(this.data.map((d) => d.set_num)) // the data range of the axis (amount of items)
                .padding(.1) // the space between the values
            return xScale;
        },
        yScale() {
            const yScale =
                scaleLinear().domain([0, max(this.data, result => { // the highest value defines the data range of this axis.
                    return result.num_parts + 250;
                })]).range([this.height, 0])
            return yScale;
        },

        yLineScale() {
            let maximum = max(this.calculated, result => { // the highest value defines the data range of this axis.
                return result.mean + 10
            })
            let minimum = min(this.calculated, result => { // the highest value defines the data range of this axis.
                return result.mean 
            })
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

        barAxis() {
            selectAll(".axis")
                .remove() // remove old axis when data changes

            let graph = select('.bargraph') // the html svg element

            graph // add all axis
                .append('g')
                .attr('class', 'axis')
                .attr('transform', 'translate(50, -100)')
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

        // this line is for the linechart
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
