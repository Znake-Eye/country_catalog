<script setup>
import axios from 'axios';
import {ref, onMounted, computed, watch} from "vue"

const rowPerPage = 25
const currentPage = ref(1)
const searchName = ref('')
var countries = ref([])
const sortType = ref('')

onMounted(() => {
    fetchApi()
})
const fetchApi = async () => {
    await axios.get('https://restcountries.com/v3.1/all')
                .then(response => {
                    countries.value = response.data
                })
}

//search function
const searchCountry = computed(() => {
    let search = searchName.value.trim().toLowerCase()
    if(!search) return countries.value

    return searchObjectsByKey(countries.value, search)

})

//help function search
function searchObjectsByKey(array, searchTerm) {
    const results = [];
    for (const obj of array) {
        const result = searchObject(obj.name, searchTerm);
        if (result)  results.push(obj);
    }
    return results;
}
function searchObject(obj, searchTerm) {
    for (const key in obj) {
        if (typeof obj[key] === 'object') {
            const result = searchObject(obj[key], searchTerm);
            if (result) return result;
            
        } else if (typeof obj[key] === 'string' && obj[key].toLowerCase().includes(searchTerm.toLowerCase())) {
            return true;
        }
    }
    return false;
}

//watch for changing sortType that user select
watch(sortType, (newType, oldType) => {
    console.log('watch work')
    if(newType == 'asc'){
        countries.value = countries.value.sort(sortByNameAsc)
    }
    else if(newType == 'desc'){
        countries.value = countries.value.sort(sortByNameDesc)
    }
    console.log('old type',oldType)
})

//sort function
function sortByNameAsc(a, b) {
    return a.name.official.localeCompare(b.name.official);
}
function sortByNameDesc(a, b) {
    return b.name.official.localeCompare(a.name.official);
}

const totalPages = computed(() => {
      return Math.ceil(searchCountry.value.length / rowPerPage);
});

const paginatedData = computed(() => {
    const startIndex = (currentPage.value - 1) * rowPerPage;
    const endIndex = startIndex + rowPerPage;
    return searchCountry.value.slice(startIndex, endIndex);
})

const prePage = () => {
    if(currentPage.value > 1){
        currentPage.value --
    }
}

const nextPage = () => {
    if (currentPage.value < totalPages.value) {
        currentPage.value++;
      }
}

</script>
<template>
    <input type="text" class="search-input" v-model="searchName" placeholder="Search country..."/>
    <div>
        Sort Name By :
        <select v-model="sortType">
            <option value="asc">ASC</option>
            <option value="desc">DESC</option>
        </select>
    </div>
    <div>
        <button @click="prePage()">pre</button>
        <span> {{ currentPage }}/{{ totalPages }} </span>
        <button @click="nextPage()">Next</button>
    </div>
    <table>
        <thead>
            <tr>
                <th>No</th>
                <th>Flag</th>
                <th>Country Name</th>
                <th>CCA2</th>
                <th>CCA3</th>
                <th>Native Name</th>
                <th>Alternative Name</th>
                <th>IDD</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(country , index) in paginatedData" :key="index">
                <td>{{ index + 1 }}</td>
                <td>
                    <div class="display-img">
                        <img :src="country.flags.png" />
                    </div>
                </td>
                <td>{{ country.name.official }}</td>
                <td>{{ country.cca2 }}</td>
                <td>{{ country.cca3 }}</td>
                <td>
                    <ul>
                        <li v-for="(outerValue, outerKey) in country.name.nativeName" :key="outerKey">
                            <strong>{{ outerKey }}</strong>:
                            <ul>
                            <li v-for="(innerValue, innerKey) in outerValue" :key="innerKey">
                                {{ innerKey }}: {{ innerValue }}
                            </li>
                            </ul>
                        </li>
                    </ul>
                </td>
                <td>
                    <ul>
                        <li v-for="(item,id) in country.altSpellings" :key="id">{{ item }}</li>
                    </ul>
                </td>
                <td>
                    {{ country.idd.root }}
                    <ul>
                        <li v-for="(suffix,id) in country.idd.suffixes" :key="id">{{ suffix  }}</li>
                    </ul>
                </td>
            </tr>
        </tbody>
    </table>
</template>
<style scoped>

table{
    border: 1px solid gray;
    border-collapse: collapse;
    width: 100%;
}
th, td{
    border:1px solid gray;
    padding: 5px;
}
td{
    text-align: left;
}
.display-img{
    width: 60px;
}
.display-img img{
    width: 100%;
}
.search-input{
    outline: none;
    padding: 0.5rem;
    font-size: 18px;
}


</style>