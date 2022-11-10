<!-- MENU COMPONENT -->
<script setup>
</script>

<template>
    <div class="naam">
        <input v-model="name" placeholder="Vul hier je naam in" />
        <button v-on:click="getUser(name)">Get user</button>
    </div>
    <ul>
        <li>Home</li>
        <li>D3</li>
        <li>API info</li>
        <li>probeer zelf</li>
    </ul>

</template>
<script>
export default {
    data() {
        return {
            name: ""
        }
    },
    props: {
        getUser: Function
    }
}

</script>