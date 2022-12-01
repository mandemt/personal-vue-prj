


<template>
    <svg width="400" height="300">
        <g>
            <rect v-for="(set, index) in results" :key="set" :x="index * 10" y="10" width="2" :height="set.num_parts"
                fill="steelblue"></rect>
        </g>
    </svg>
    <div class="graphcontainer">
        <!-- input for theme -->
        <section class="theme-chooser">
            <h2>Kies een thema: </h2>
            <select v-if="themes" v-model="selected" class="newtheme" @change="newTheme()">
                <option v-for="theme in themes" :value="{ name: theme.name, id: theme.id }">{{ theme.name }}</option>
            </select>
        </section>

        <!-- filter the data on year  -->
        <section class="filter">
            <h1>{{ this.selected.name }}</h1>
            <select v-if="years" v-model="filtered">
                <option value="jaar">Alle jaren</option>
                <option v-for="year in years" :value="year.year">{{ year.year }}</option>
            </select>
            <svg v-if="results" class="graph" :width="width" :height="height" viewBox="0 -10 800 700">
                <g class="bars">
                    <rect @mouseover="changePiece()" v-for="d in results" :x="xScale(d.set_num)" :y="yScale(d.num_parts)"
                        :width="xScale.bandwidth()" :height="height - yScale(d.num_parts)" :value="d.num_parts"
                        :class="d.num_parts" :name="d.name" :fill="blue" />
                </g>
            </svg>
            <!-- the chart -->
            <!-- <Charts v-if="results" :data="results" :jaren="jaren"></Charts> -->
        </section>
    </div>
</template>
<script setup>
  


import Charts from './Charts.vue'
import { scaleBand, scaleLinear, max, select, interpolateNumber, axisBottom, axisTop, axisLeft, line, min, selectAll } from 'd3'


let themes = await fetch('https://rebrickable.com/api/v3/lego/themes/?page_size=500&key=' + API_KEY)
    .then((response) => response.json())

themes = themes.results

</script>
<script>
let API_KEY = import.meta.env.VITE_API_KEY // hidden api key from rebrickable API
let width = 800;
let height = 800;
export default {
    data() {
        return {
            themes: [],
            selected: [],
            results: []
        }
    },
    methods: {
        newTheme() {
            this.theme = this.selected
            console.log(this.selected)
            // retrieveData(this.selected.id) // retrieve data for the selected theme
            this.retrieveData(this.selected.id)
        },
        async retrieveData(theme) {
            let results = await fetch('https://rebrickable.com/api/v3/lego/sets/?page_size=1000&theme_id=' + theme + '&key=' + API_KEY)
                .then((response) => response.json())

            this.results = await results.results
            console.log(this.results)
            results = results.results

            const xScale =
                scaleBand()
                    .range([0, width]) //  the pixel range of the axis
                    .domain(results.map((d) => d.set_num)) // the data range of the axis (amount of items)
                    .padding(0.2) // the space between the values

            const yScale =
                scaleLinear().domain([0, max(results, result => { // the highest value defines the data range of this axis.
                    return result.num_parts + 50;
                })]).range([height, 0])

            let xaxis = axisBottom(xScale)
            let yaxis = axisLeft(yScale).ticks(10)

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

        },
    }

}</script>

<!-- <script>
import {reactive} from 'vue'
let API_KEY = import.meta.env.VITE_API_KEY // hidden api key from rebrickable API

export default {

    data() {
        return { // define all reactive states
            themes: null,
            results: null,
            years: null,
            jaren: null,
            selected: [],
            year: null
        }
    },

    // general data for all charts
    methods: {
        // get all themes
        async getThemes() {
            let response = await fetch('https://rebrickable.com/api/v3/lego/themes/?page_size=500&key=' + API_KEY)
            let data = await response.json()
            data.results.sort(function (a, b) {
                return a.name.localeCompare(b.name)
            })
            this.themes = data.results
        },

        // when a new theme is selected
        async newTheme() {
            this.results = null // reset results so the bar charts regenerates
            this.theme = this.selected
            this.retrieveData(this.selected.id) // retrieve data for the selected theme
           
        },

        // get sets and years of one theme
     async  retrieveData(theme) {

          await fetch('https://rebrickable.com/api/v3/lego/sets/?page_size=1000&theme_id=' + theme + '&key=' + API_KEY)
                .then((response) => response.json())
                .then((data) => this.results = data.results)
                .then((data) => this.years = data.reduce((o, a) => {
                    o[a.year] = a; return o
                }, {}));
            await this.calculate(this.results)
   
        },
calculate(e){
            const distinct = (value, index, self) => {
                return self.indexOf(value) === index;
            }
            let lineYears = this.results.map(d =>
                d.year)
                
            lineYears = lineYears.filter(distinct)
            let yearAndMean = []
            lineYears.forEach(year => {
                let lol = this.results.filter(d => d.year === year)
                let gem = mean(lol, d => d.num_parts)
                console.log(gem)
                yearAndMean.push({ 'year': year, 'mean': Number(gem) })

            })
            setTimeout(() => {

                this.jaren = yearAndMean.sort().sort((a, b) =>
                    a.year - b.year)
                // set filtered results
            }, 10);


},
        // when a year is selected
        async filterDataOnYear() {
            const year = event.target.value // value from input
            this.year = year
            await this.retrieveData(this.theme.id)
            var newResults = (year === 'jaar') ? this.results : this.results.filter(result => result.year === Number(year));
            console.log(newResults)
            this.results = null // reset results
            this.jaren = null
            setTimeout(() => {
                this.results = newResults // set filtered results
            }, 8);
        },
    },

    mounted() {
      console.log('hallo')
    }
}




</script> 

<style >
.expand-enter-active,
.expand-leave-active {
    transition: height 1s ease-in-out;
    overflow: hidden;
}

.expand-enter,
.expand-leave-to {
    height: 0;
}

rect {
    /* height: 0; */
    opacity: .5;
    transition: 1s ease;
    /* -moz-transition: height .5s;
  -ms-transition: height .5s;
  -o-transition: height .5s;
  -webkit-transition: height .5s;
  transition: height .5s;
  overflow: hidden;
  outline: 1px solid red; */
}

.hoi1 {
    background-color: red;
    width: 80%;
}

.hoi2 {
    background-color: blue;
    width: 50%;
}

.hoi3 {
    background-color: green;
    width: 30%;
}

.hoi4 {
    background-color: yellow;
    width: 10%;
}

.hoi5 {
    background-color: orange;
    width: 56%;
}

.hoi6 {
    background-color: purple;
    width: 90%;
}

.hoi7 {
    background-color: pink;
    width: 20%;
}

.graphcontainer {
    display: flex;
}

.theme-chooser {
    display: flex;
    flex-direction: column;
    background-color: #38ada9;
    width: 50%;
    align-items: center;
    text-align: center;
}

h2 {
    color: white;
    background: #82ccdd;
    width: 100%;
    margin: 0;

}

body {
    margin: 0;
    padding: 0;
}

.newtheme {
    width: 50%;
    margin: 5em 0;

}

.filter {
    display: flex;
    flex-direction: column;
    justify-content: center;
    background-color: #222f3e;
    padding: 15px;
}

div {
    transition: 1s;
}

select {
    padding: 0;
    margin: auto;
    border: none;
    background-color: #0abde3;
    width: 25%;
    text-align: center;
    height: 50px;
    border-radius: 20px;
    color: black;

}

h1 {
    margin: 0;
    padding: 0;
    color: white;
    text-align: center;
}

.sets {
    flex-direction: column;
}

.barchart,
.linechart,
.piechart {
    border-radius: 10%;
    padding: 50px;
    box-shadow: 10px 5px 5px grey;
    background: lightgray;
    width: 500px;
    height: 300px;
}

.domain {
    stroke: white;
    color: white;
    font-size: 8px;
    font-weight: 100;
}

.domain text:hover,
.yaxis {
    transition: .2s;
    position: absolute;
    color: ORANGE;
    font-size: 15px;
}

rect {
    transition: 1s;
}
</style>