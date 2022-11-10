<script setup>
import * as d3 from "d3"
import { svg } from "d3";

</script>

<template>
    <div class="barchartdiv">
        <select @change="update()">
            <!-- the year filter -->
            <option>jaar</option>
            <option v-for="year in years" :value="year.year">{{ year.year }}</option>
        </select>

        <svg class="barchart"></svg> <!-- the canvas for the bar chart -->
    </div>
</template>

<script>

export default {

    data() {
   
    },

    //the properties from the <barchart> element
    props: {
        title: String,
        data: Array
    },

    // functions used for the bar chart
    methods: {

        // to get all years with a set from the specific theme
        getYears(data) {
            this.years = Object.values(
                data.reduce((o, a) => { o[a.year] = a; return o }, {}));
        },

        // create the bar chart
        fillData(data) {

            let width = d3.select('svg').node().getBoundingClientRect().width // get width from css
            let height = d3.select('svg').node().getBoundingClientRect().height // get height from css

            // the <svg> sizing
            let svg = d3.select(".barchart")
                .attr("width", width)
                .attr("height", height)
                .attr("viewBox", [0, 0, width, height])
                .attr("style", "max-width: 100%; height: auto; height: intrinsic;");


            // the pixels to divide the bars in width
            let xScale = d3.scaleBand()
                .range([0, width - 100])
                .padding(0.1);

            let yScale = d3.scaleLinear()
                .range([height, 100]);

            let g = svg.append("g").attr("transform", "translate(" + 50 + "," + -50 + ")"); // the positioning of the chart within the canvas

            // the range in values 
            xScale.domain(data.map(result => {
                return result.set_num
            }))

            yScale.domain([0, d3.max(data, result => {
                return result.num_parts;
            })]);

            // the bars in the svg element
            g.selectAll("bar")
                .data(data) // put data to elements
                .enter() // create element for every data point
                .append("rect")
                .attr("class", "bar") // give the <rect> a class
                .attr("x", function (d) { return xScale(d.set_num); }) // the position comes from every data point
                .attr("y", function (d) { return yScale(d.num_parts); })
                .attr("width", xScale.bandwidth()) // the width is divided over the range with the padding
                .attr("height", function (d) { return height - yScale(d.num_parts); }) // the difference between the y axis and the data point

            //   the y axis
            g.append("g")
                .call(d3.axisLeft(yScale).tickFormat(function (d) { // the amount of labels on the y axis
                    return d;
                }).ticks(20))
                .append("text")
            // the label



            // x axis
            g.append("g")
                .attr("transform", "translate(0," + height + ")")
                .call(d3.axisBottom(xScale))
                .selectAll("text")
                .attr("transform", "rotate(90)")
                .attr("y", "0")
                .attr("x", "10")
                .style("text-anchor", "start");



        },

        // when the year filter is changed
        update() {
            this.year = event.target.value
            this.fillData(data)
        },
    },
    // when the page is loaded or changed
    mounted() {
        let data = this.data
        data.sort((a, b) => a.set_num - b.set_num)
        this.getYears(data)
        this.fillData(data)
    }

}
</script>

  
<style>
select {
    width: 5em;
    height: 2em;
}

.bar {
    transition: r 0.2s ease-in-out;
}

.bar-positive:hover {
    fill: brown;
}

rect {
    fill: #e67e22;
}

rect:hover {
    fill: #27ae60;
}

.barchartdiv {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}
</style>
  