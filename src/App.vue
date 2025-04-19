<script setup>
import HelloWorld from './components/HelloWorld.vue'
import {ref, onMounted} from 'vue'
import PocketBase from 'pocketbase'
const pb = new PocketBase('http://127.0.0.1:8090')
const hi = "Aura Mankimi"
const solatApiLink = "https://www.e-solat.gov.my/index.php?r=esolatApi/takwimsolat&period=today&zone=SGR01"
const pokeLink = "https://pokeapi.co/api/v2/pokemon?limit=151"
let number = ref(0);
let name = ref("");
let added = ref(false);
let empty = ref(false);
let win = ref(false)
let lose = ref(false)
let gameStart = ref(false)
let capitalCities = ref(["Kimi Shibuya", "Man Pinang", "Man Town", "Man City", "Kimi Lumpur", "Kimi Hawaii"])
let interval;
let color = ref("White")
let prayerTime = ref({})
let pokeList = ref([])
let inputPokemon = ref("")
let searchResult= ref("")
let found = ref(false)
let list = ref([])
let todoInput = ref("")

function increment(){
  number.value++;
}
function stop(){
  clearInterval(interval)
  if(number.value==10){
    win.value = true
  }else{
    lose.value = true
  }
  console.log(number.value)
  console.log(win.value)
}
function startInterval(){
  number.value = 0
  lose.value = false
  interval = setInterval(increment,100)
  color = ref("Red");
  gameStart.value = true
}
function addCity(){
  if(name.value != 0){
    capitalCities.value.push(name.value)
    empty.value = false
    added.value = true
    name.value = ""
  } else{
    empty.value = true
  }
}
function getPrayerTime(){
    fetch(solatApiLink)
    .then(result => result.json())
    .then(data => {
      prayerTime.value = data.prayerTime[0]
      console.log(prayerTime.value)
  }
  )
}
function getPokeList(){
  fetch(pokeLink)
  .then(res => res.json())
  .then(data => {
    pokeList.value = data.results
    console.log(pokeList.value)
  })
}
function searchPokemon(){
  for(let pokemon in pokeList.value){
    if(pokeList.value[pokemon].name == inputPokemon.value){
      searchResult.value = pokeList.value[pokemon].name
      found.value = true
    }
  }
}
async function getTodolist(){
  list.value = await pb.collection('todos').getFullList();
  console.log(list.value);
}
async function createTodo(){
  const newTodo = {
    item: todoInput.value,
    isDone: false
  }
  const record = await pb.collection('todos').create(newTodo);
}
onMounted(()=>{
  getPrayerTime()
  getPokeList()
  getTodolist()

})
</script>

<template>
  <div class="min-h-screen bg-zinc-900 text-gray-200 p-6 font-mono">
    <div class="max-w-4xl mx-auto">
      <!-- PocketBase -->
      <div>
        <h1>To Dos</h1>
        <input 
          type="text" 
          v-model="todoInput">
        <button @click="createTodo">Add</button> 
      </div>
      <!-- Aura Controls Section -->
      <div class="bg-zinc-800 p-6 rounded-lg shadow-lg mb-8">
        <div class="border-b border-zinc-700 pb-4 mb-4">
          <h1 class="text-3xl text-center font-bold text-white">{{ hi }}</h1>
          <h1 class="text-4xl font-bold text-center" :class="`text-[${color}]`">{{ number }}</h1>
          
          <div class="mt-4">
            <p v-if="win" class="text-lime-400 text-xl font-medium">You win! Mankimi Aura is at a good spot</p>
            <p v-if="gameStart && lose" class="text-red-500 text-xl font-medium">You lose! mankimi aura is not precise :(</p>
          </div>
        </div>
        
        <h2 class="text-xl font-bold mb-4 text-gray-300 border-b border-zinc-700 pb-2 text-center">Aura Controls</h2>
        <div class="flex flex-col md:flex-row md:space-x-4 space-y-4 md:space-y-0">
          <button @click="startInterval" class="flex-1 bg-zinc-700 hover:bg-zinc-600 text-white py-3 px-6 rounded transition-all duration-200 border-l-4 border-lime-500 shadow-md">
            <span class="flex items-center justify-center">
              <span class="mr-2">+</span>Aura
            </span>
          </button>
          <button @click="stop" class="flex-1 bg-zinc-700 hover:bg-zinc-600 text-white py-3 px-6 rounded transition-all duration-200 border-l-4 border-red-500 shadow-md">
            <span class="flex items-center justify-center">
              X Stop Aura
            </span>
          </button>
        </div>
      </div>
      
      <!-- Add City Section -->
      <div class="bg-zinc-800 p-6 rounded-lg shadow-lg mb-8">
        <h2 class="text-xl font-bold mb-4 text-gray-300 border-b border-zinc-700 pb-2">Add City</h2>
        <div class="space-y-4">
          <div class="flex flex-col space-y-2">
            <input 
              type="text" 
              v-model="name" 
              placeholder="Enter city name"
              class="bg-zinc-700 border border-zinc-600 text-gray-200 py-2 px-4 rounded focus:outline-none focus:ring-2 focus:ring-gray-400 focus:border-transparent"
            >
            <div class="h-6">
              <p v-if="added" class="text-lime-400 text-sm">City Added Successfully!</p>
              <p v-if="empty" class="text-red-500 text-sm">Please Type Anything?</p>
            </div>
          </div>
          <button @click="addCity" class="w-full bg-zinc-700 hover:bg-zinc-600 text-white py-2 px-4 rounded transition-all duration-200 border-l-4 border-blue-500 shadow-md">
            Add City
          </button>
        </div>
      </div>
      
      <!-- Cities Section -->
      <div class="bg-zinc-800 p-6 rounded-lg shadow-lg mb-8">
        <h2 class="text-xl font-bold mb-4 text-gray-300 border-b border-zinc-700 pb-2">Capital Cities (Man Version)</h2>
        <ul class="list-disc pl-6 space-y-1">
          <li v-for="city in capitalCities" class="text-gray-300">{{ city }}</li>
        </ul>
      </div>
      
      <!-- Prayer Times Section -->
      <div class="bg-zinc-800 p-6 rounded-lg shadow-lg mb-8">
        <div class="flex justify-between items-center mb-6">
          <h2 class="text-xl font-bold text-gray-300">Prayer Times</h2>
          <button @click="getPrayerTime" class="bg-zinc-700 hover:bg-zinc-600 text-white py-2 px-4 rounded transition-all duration-200 shadow-md text-sm">
            Refresh Times
          </button>
        </div>
        
        <div class="mb-6 text-center">
          <h3 class="text-2xl font-bold text-gray-300">Maghrib</h3>
          <p class="text-3xl font-bold text-lime-400">{{ prayerTime.maghrib }}</p>
        </div>
        
        <div class="overflow-x-auto rounded-lg border border-zinc-700">
          <table class="w-full">
            <thead>
              <tr class="bg-zinc-700">
                <th class="py-3 px-4 text-left text-sm font-medium text-gray-300">Fajr</th>
                <th class="py-3 px-4 text-left text-sm font-medium text-gray-300">Dhuha</th>
                <th class="py-3 px-4 text-left text-sm font-medium text-gray-300">Dhuhr</th>
                <th class="py-3 px-4 text-left text-sm font-medium text-gray-300">Asr</th>
                <th class="py-3 px-4 text-left text-sm font-medium text-gray-300">Maghrib</th>
                <th class="py-3 px-4 text-left text-sm font-medium text-gray-300">Isha</th>
              </tr>
            </thead>
            <tbody>
              <tr class="border-t border-zinc-700">
                <td class="py-3 px-4 text-gray-300">{{ prayerTime.fajr }}</td>
                <td class="py-3 px-4 text-gray-300">{{ prayerTime.dhuha }}</td>
                <td class="py-3 px-4 text-gray-300">{{ prayerTime.dhuhr }}</td>
                <td class="py-3 px-4 text-gray-300">{{ prayerTime.asr }}</td>
                <td class="py-3 px-4 text-gray-300">{{ prayerTime.maghrib }}</td>
                <td class="py-3 px-4 text-gray-300">{{ prayerTime.isha }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      
      <!-- Pokemon Search Section -->
      <div class="bg-zinc-800 p-6 rounded-lg shadow-lg">
        <h2 class="text-xl font-bold mb-4 text-gray-300 border-b border-zinc-700 pb-2">Pokemon Search</h2>
        <div class="space-y-4">
          <div class="flex flex-col md:flex-row space-y-2 md:space-y-0 md:space-x-2">
            <input 
              type="text" 
              v-model="inputPokemon" 
              placeholder="Enter pokemon name"
              class="flex-grow bg-zinc-700 border border-zinc-600 text-gray-200 py-2 px-4 rounded focus:outline-none focus:ring-2 focus:ring-gray-400 focus:border-transparent"
            >
            <button @click="searchPokemon" class="bg-zinc-700 hover:bg-zinc-600 text-white py-2 px-6 rounded transition-all duration-200 border-l-4 border-yellow-500 shadow-md">
              Search
            </button>
          </div>
          
          <p v-if="found" class="text-lime-400">{{ searchResult }} is found!</p>
          
          <div class="mt-4">
            <h3 class="text-lg font-medium text-gray-300 mb-2">Pokemon List</h3>
            <ol class="list-decimal pl-6 space-y-1">
              <li v-for="pokemon in pokeList" class="text-gray-300">{{ pokemon.name }}</li>
            </ol>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
