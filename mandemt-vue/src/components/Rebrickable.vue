
<script setup>
import { mean } from 'd3'
import Computed from './Computed.vue'
</script>

<template>

    <div class="graphcontainer">
        <!-- input for theme -->
        <section class="theme-chooser">
            <h2>Kies een thema: </h2>
            <select v-model="selected" class="newtheme" @change="retrieveData()">
                <option v-if="themes" v-for="theme in sortedThemes" :value="{ name: theme.name, id: theme.id }">
                    {{ theme.name }}
                </option>
            </select>
        </section>

        <!-- filter the data on year  -->
        <select v-if="results" v-model="filtered">
            <option value="jaar">Alle jaren</option>
            <option v-if="sortedYears" v-for="year in sortedYears" :value="year">{{ year }}</option>
        </select>

        <h1>{{ this.selected.name }}</h1>
        <!-- filter on ascending year or amount of pieces -->
        <select v-if="sortedYears" v-model="filtering">
            <option v-if="filtered == 'jaar'" value="sortyear">sort on year</option>
            <option value="sortpieces">sort on piece count</option>
        </select>
        <!-- the bar and line chart -->
        <Computed :data="allData" :calculated="calculatedResults"></Computed>
    </div>

</template>
<script>
let API_KEY = import.meta.env.VITE_API_KEY // hidden api key from rebrickable API

export default {
    data() {
        return { // define all reactive states
            themes: [],
            results: [],
            filtered: [],
            filtering: 'sortyear',
            allData: [],
            selected: [],
            year: null,
            theme: '',
        }
    },

    methods: {
        // retrieve all themes from the API
        async getThemes() {
            let response = await fetch('https://rebrickable.com/api/v3/lego/themes/?page_size=500&key=' + API_KEY)
            let data = await response.json()
            this.themes = data.results
        },

        // retrieve all sets from the API
        async retrieveData() {
            this.filtered = 'jaar'
            const response = await fetch('https://rebrickable.com/api/v3/lego/sets/?page_size=500&theme_id=' + this.selected.id + '&key=' + API_KEY)
            let data = await response.json()
            this.allData = await data.results
        },
    },

    computed: {
        // sort the themes on name alphabetically
        sortedThemes() {
            return this.themes.sort((a, b) => a.name.localeCompare(b.name))
        },

        // get the average amount of pieces for every year
        calculatedResults() {
            let yearAndMean = []
            this.sortedYears.forEach(year => {
                let setsFromYear = this.allData.filter(d => d.year === year)
                let gem = mean(setsFromYear, set => set.num_parts)
                yearAndMean.push({ 'year': year, 'mean': Math.round(Number(gem)) })
            })
            return yearAndMean;
        },

        // the data for the bar and line chart
        allData() {
            // when a filter is applied
            if (typeof this.filtered == 'number') {
                let nouveau = this.allData.filter(d => d.year === this.filtered) // filter on specific year
                if (this.filtering === 'sortyear') {
                    return nouveau.sort((a, b) => a.year - b.year)
                }
                if (this.filtering === 'sortpieces') {
                    return nouveau.sort((a, b) => a.num_parts - b.num_parts)
                }
            }

            else {
                if (this.filtering === 'sortyear') {
                    return this.allData.sort((a, b) => a.year - b.year)
                }
                if (this.filtering === 'sortpieces') {
                    return this.allData.sort((a, b) => a.num_parts - b.num_parts)
                }
            }
        },

        // sort the years on ascending order for the line chart
        sortedYears() {
            return new Set(this.allData.map(item => item.year).sort((a, b) => a - b));
        },
    },

    mounted() {
        this.getThemes()
    }
}
</script>

>