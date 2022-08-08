<script setup>
import { LockClosedIcon, LockOpenIcon } from '@heroicons/vue/solid'
import { ref } from 'vue'

const statusMap = ref({
  1: 'bg-green-500', // memorized
  2: 'bg-red-500', // needs work
  3: 'bg-gray-500', // default
  4: 'bg-orange-500', // needed a hint
})

const isActive = ref(false)

function doSomething(param) {
  console.log(param)
}
  
defineProps({
  verse: {
    type: Object,
    required: true
  }
})
</script>

<template>
  <div>
    <button type="button" @click="isActive = !isActive" :class="[statusMap[verse.status], 'inline-flex items-center p-1 border border-transparent rounded-full shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500']">
      <span class="text-white text-xs flex items-center justify-center h-6 w-6">v{{verse.id}}</span>
    </button>

    <div v-if="isActive" class="absolute top-0 right-11 flex items-center">
      <button type="button" class="inline-flex items-center p-2 border border-transparent rounded-full shadow-sm text-white bg-red-500 hover:bg-red-600 focus:outline-none focus:ring-3 focus:ring-offset-2 focus:ring-red-500 shadow-lg" @click="verse.status = 2; isActive = false">
        <LockOpenIcon class="h-4 w-4" />
      </button>
      <button type="button" class="inline-flex items-center ml-2 p-2 border border-transparent rounded-full shadow-sm text-white bg-green-500 hover:bg-green-600 focus:outline-none focus:ring-3 focus:ring-offset-2 focus:ring-green-500 shadow-lg" @click="verse.status = 1; isActive = false">
        <LockClosedIcon class="h-4 w-4" />
      </button>
    </div>
  </div>
</template>

<style scoped>

</style>
