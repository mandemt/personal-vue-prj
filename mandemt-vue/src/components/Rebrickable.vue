
<script setup>
import Charts from './Charts.vue'

</script>

<template>
    <div class="graphcontainer">

        <!-- input for theme -->
        <section class="theme-chooser">
            <h2>Kies een thema: </h2>
            <select v-if="themes" v-model="selected" class="newtheme" @change="newTheme">
                <option v-for="theme in themes" :value="{ name: theme.name, id: theme.id }">{{ theme.name }}</option>
            </select>
        </section>

        <!-- filter the data on year  -->
        <section class="filter">
            <h1>{{ this.selected.name }}</h1>
            <select v-if="years" @change="filterDataOnYear()">
                <option value="jaar">Alle jaren</option>
                <option v-for="year in years" :value="year.year">{{ year.year }}</option>
            </select>

            <!-- the chart -->
            <Charts v-if="results" :data="results"></Charts>
        </section>
    </div>
</template>

<script>
let API_KEY = import.meta.env.VITE_API_KEY // hidden api key from rebrickable API

export default {

    data() {
        return { // define all reactive states
            themes: null,
            results: null,
            years: null,
            selected: [],
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
        newTheme() {
            this.results = null // reset results so the bar charts regenerates
            this.theme = this.selected
            this.retrieveData(this.selected.id) // retrieve data for the selected theme
        },

        // get sets and years of one theme
        async retrieveData(theme) {
            fetch('https://rebrickable.com/api/v3/lego/sets/?page_size=1000&theme_id=' + theme + '&key=' + API_KEY)
                .then((response) => response.json())
                .then((data) => this.results = data.results)
                .then((data) => this.years = data.reduce((o, a) => {
                    o[a.year] = a; return o
                }, {}));
        },

        // when a year is selected
        filterDataOnYear() {
            const year = event.target.value // value from input
            this.year = year
            this.retrieveData(this.theme.id)
            var newResults = (year === 'jaar') ? this.results : this.results.filter(result => result.year === Number(year));
            this.results = null // reset results
            setTimeout(() => {
                this.results = newResults // set filtered results
            }, 10);
        },
    },

    mounted() {
        this.getThemes()
    }
}




</script>

<style >
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
    width: 70%;
    background-color: #222f3e;
    padding: 15px;
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

.domain,
.yaxis {
    stroke: white;
    color: white;
    font-size: 12px;
    font-weight: 100;
}
</style>