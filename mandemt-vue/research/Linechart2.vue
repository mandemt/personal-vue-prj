<script setup>
import * as d3 from 'd3'

</script>
<template>
    <svg class="linechart"></svg> <!-- the canvas for the line chart -->
</template>

<script>
export default {

    
    props: {
        title: String,
        xKey: String,
        yKey: String,
        data: Array
    },
    methods: {
        insertLineChartData(data, {
            xKey,
            yKey,
            defined,
            curve = d3.curveLinear, // method of interpolation between points
            marginTop = 20, // top margin, in pixels
            marginRight = 30, // right margin, in pixels
            marginBottom = 30, // bottom margin, in pixels
            marginLeft = 40, // left margin, in pixels
            width, // outer width, in pixels
            height,  // outer height, in pixels
            xType = d3.scaleLinear, // the x-scale type
            xDomain, // [xmin, xmax]
            xRange = [30, 400], // [left, right]
            yType = d3.scaleLinear, // the y-scale type
            yDomain, // [ymin, ymax]
            yRange = [400, 30], // [bottom, top]
            yFormat, // a format specifier string for the y-axis
            yLabel,
            xLabel,
            color = "CurrentColor", // stroke color of line
            strokeLinecap = "round", // stroke line cap of the line
            strokeLinejoin = "round", // stroke line join of the line
            strokeWidth = 3, // stroke width of line, in pixels
            strokeOpacity = 1, // stroke opacity of line
        } = {}) {

            const X = d3.map(data, xKey);
            const Y = d3.map(data, yKey);
            const I = d3.range(X.length)
            if (defined === undefined) defined = (d, i) => !isNaN(X[i]) && !isNaN(Y[i]);
            const D = d3.map(data, defined)
            if (xDomain === undefined) xDomain = [d3.min(X), d3.max(X)];
            if (yDomain === undefined) yDomain = [0, d3.max(Y) * 1.1];
            const xScale = xType(xDomain, xRange);
            const yScale = yType(yDomain, yRange);
            const xAxis = d3.axisBottom(xScale);
            const yAxis = d3.axisLeft(yScale).ticks(height / 40, yFormat);
            const line = d3.line()
                .defined(i => D[i])
                .curve(curve)
                .x(i => xScale(X[i]))
                .y(i => yScale(Y[i]));

            const svg = d3.select(".linechart")
                .attr("width", width)
                .attr("height", height)
                .attr("viewBox", [0, 0, width, height])
                .attr("style", "max-width: 100%; height: auto; height: intrinsic;");

            svg.append("g")
                .attr("transform", `translate(0,${height - marginBottom})`)
                .call(xAxis)
                .call(g => g.append("text")
                    .attr("y", -marginLeft)
                    .attr("y", 10)
                    .attr("fill", "blue")
                    .attr("text-anchor", "start")
                    .text(xLabel));


            svg.append("g")
                .attr("transform", `translate(${marginLeft},0)`)
                .call(yAxis)
                .call(g => g.select(".domain").remove())
                .call(g => g.selectAll(".tick line").clone()
                    .attr("x2", width - marginLeft - marginRight)
                    .attr("stroke-opacity", 0.1))
                .call(g => g.append("text")
                    .attr("x", -marginLeft)
                    .attr("y", 10)
                    .attr("fill", "blue")
                    .attr("text-anchor", "start")
                    .text(yLabel));

            svg.append("path")
                .attr("fill", "none")
                .attr("stroke", color)
                .attr("stroke-width", strokeWidth)
                .attr("stroke-linecap", strokeLinecap)
                .attr("stroke-linejoin", strokeLinejoin)
                .attr("stroke-opacity", strokeOpacity)
                .attr("d", line(I));
            return svg.node()
        }


    },
    mounted() {
        let yearAndMean = []
        let data = this.data.sort(
            (a, b) => a.num_parts - b.num_parts
        );
        const distinct = (value, index, self) => {
            return self.indexOf(value) === index;
        }
        let years = data.map(d =>
            d.set_num)
        yearAndMean.sort((a, b) =>
            a.set_num - b.set_num)
        this.insertLineChartData(yearAndMean, {
            xKey: d => d.year,
            yKey: d => d.num_parts,
            yLabel: 'Piece count',
            xLabel: 'Year',
            width: 500,
            height: 500,
            color: "orange"
        })

    }
}


</script>