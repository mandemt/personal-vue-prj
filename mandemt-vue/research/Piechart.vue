<script setup>
import * as d3 from "d3"
import { namespace } from "d3";
import { onMounted } from "vue";

</script>

<template>
    <svg class="piechart"></svg>
</template>

<script>
export default {
    name: "PieChart",
    props: {
        title: String,
        data: Array
    },
    methods: {
        fillPieChartData(data, {
            value,
            name,
            title,
            width = d3.select('svg').node().getBoundingClientRect().width, // get width from css
            height = d3.select('svg').node().getBoundingClientRect().height, // get height from css
            innerRadius = 100,
            outerRadius = Math.min(width, height) /2,
            labelRadius = (innerRadius * 0.2 + outerRadius * 0.8),
            names,
            colors = ["#1f77b4", "#ff7f0e", "#2ca02c", "#8c564b", "#e377c2", "#7f7f7f", "#bcbd22", "#17becf"], // array of colors for names
            padAngle = 0
        } = {}) {

            const N = d3.map(data, name);
            const V = d3.map(data, value);
            const I = d3.range(N.length).filter(i => !isNaN(V[i]));
            if (names === undefined) names = N;
            names = new d3.InternSet(names);

            if (colors === undefined) colors = d3.schemeSpectral[names.size];
            if (colors === undefined) colors = d3.quantize(t => d3.interpolateSpectral(t * 0.8 + 0.1), names.size);

            const color = d3.scaleOrdinal(names, colors);

            //title
            const O = d3.map(data, d => d);
            const T = title;
            title = i => T(O[i], i, data);

            // Construct arcs.
            const arcs = d3.pie().padAngle(padAngle).sort(null).value(i => V[i])(I);
            const arc = d3.arc().innerRadius(innerRadius).outerRadius(outerRadius);
            const arcLabel = d3.arc().innerRadius(labelRadius).outerRadius(labelRadius);


            const svg = d3.select(".piechart")
                .attr("width", width)
                .attr("height", height)
                .attr("viewBox", [-width / 2, -height / 2, width, height])
                .attr("style", "max-width: 100%; height: auto;")

            svg.append("g")
                .selectAll("path")
                .data(arcs)
                .join("path")
                .attr("fill", d => color(N[d.data]))
                .attr("d", arc)
                .append("title")
                .text(d => title(d.data));


            svg.append("g")
                .attr("font-family", "sans-serif")
                .attr("font-size", 12)
                .attr("text-anchor", "middle")
                .selectAll("text")
                .data(arcs)
                .join("text")
                .attr("transform", d => `translate(${arcLabel.centroid(d)})`)
                .selectAll("tspan")
                .data(d => {
                    const lines = `${title(d.data)}`.split(/\n/);
                    return (d.endAngle - d.startAngle) > 0.25 ? lines : lines.slice(0, 1);
                })
                .join("tspan")
                .attr("x", 0)
                .attr("y", (_, i) => `${i * 1.1}em`)
                .attr("font-weight", (_, i) => i ? null : "bold")
                .text(d => d);
            return Object.assign(svg.node(), { scales: { color } });
        }
    },

    mounted() {
        let data = this.data
        this.fillPieChartData(data, {
            name: d => d.name,
            value: d => d.num_parts,
            title: d => d.name,

        })
    },
}



</script>
<style scoped>
tspan:hover {
    visibility: visible;
    color: blue;
}
</style>
