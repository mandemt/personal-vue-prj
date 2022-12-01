
<script setup>
import Charts from './Charts.vue'
import { mean } from 'd3'
import Computed from './Computed.vue'
</script>

<template>

    <div class="graphcontainer">
        <!-- input for theme -->
        <section class="theme-chooser">
            <h2>Kies een thema: </h2>
            <select v-model="selected" class="newtheme" @change="retrieveData()">
                <option v-if="themes" v-for="theme in sortedThemes" :value="{ name: theme.name, id: theme.id }">{{
                        theme.name
                }}
                </option>
            </select>
        </section>
     
        <!-- filter the data on year  -->
        <select v-if="results" v-model="filtered">
            <option value="jaar">Alle jaren</option>
            <option v-if="sortedYears" v-for="year in sortedYears" :value="year">{{ year }}</option>
        </select>
        <h1>{{ this.selected.name }}</h1>
        <select v-model="filtering">
<option>lol</option>
<option value="fff">sort on piece count</option>
</select>
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
            years: null,
            filtered: [],
            jaren: [],
            allData: [],
            selected: [],
            theResults: [],
            year: null,
            theme: '',
        }
    },

    // general data for all charts
    methods: {
        // get all themes
        async getThemes() {
            let response = await fetch('https://rebrickable.com/api/v3/lego/themes/?page_size=500&key=' + API_KEY)
            let data = await response.json()
            this.themes = data.results
        },
        async retrieveData() {
            this.filtered = 'jaar'
            const response = await fetch('https://rebrickable.com/api/v3/lego/sets/?page_size=500&theme_id=' + this.selected.id + '&key=' + API_KEY)
            let data = await response.json()
            this.allData = await data.results
        },
        async filterDataOnYear() {
            const year = event.target.value // value from input
            this.year = year
            await this.retrieveData(this.theme.id)

            var newResults = (year === 'jaar') ? this.results : this.results.filter(result => result.year === Number(year));
            this.results = null // reset results
            setTimeout(() => {
                this.results = newResults // set filtered results
            }, 8);
        },
    },

    computed: {
        sortedThemes() {
            return this.themes.sort((a, b) => a.name.localeCompare(b.name))
        },
        calculatedResults() {
            let yearAndMean = []
            this.sortedYears.forEach(year => {
                let setsFromYear = this.allData.filter(d => d.year === year)
                let gem = mean(setsFromYear, set => set.num_parts)
                yearAndMean.push({ 'year': year, 'mean': Math.round(Number(gem)) })
            })
            return yearAndMean;
        },
        allData() {
            if(this.filtering == 'fff'){
                console.log(this.filtering)
                return this.allData.sort((a, b) => a.num_parts - b.num_parts)
            }
            if (typeof this.filtered == 'number') {
                return this.allData.filter(d => d.year === this.filtered)
            }
           
            else {
                return this.allData
            }
            console.log(this.filtering)
            return this.allData
        },

        sortedYears() {
            return new Set(this.allData.map(item => item.year).sort((a, b) => a - b));
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
    flex-direction: column;
}

.theme-chooser {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
}

h2 {
    color: white;
    width: 100%;
    margin: 0;

}

body {
    margin: 0;
    padding: 0;
}

.newtheme {
    margin: 1em 0;

}

select {
    color: white;
    padding: 0;
    font-size: 18px;
    margin: auto;
    border: none;
    width: 25%;
    background-color: #718093;
    text-align: center;
    height: 50px;

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
    transition: .2s ease;
    position: absolute;
    font-size: 15px;
    color: white;
}

.tick line {
    color: gray;
}
</style>