<!-- This example requires Tailwind CSS v2.0+ -->
<template>
  <div class="px-4 py-5">
    <div class="mx-auto w-full max-w-sm">
      <h1 class="font-bold">Quran Memorization Randomizer</h1>
      <p class="text-sm">Test your memorization by starting from a random verse in a Surah</p>
    </div>
    <div class="bg-white mx-auto mt-3 px-4 py-5 pb-6 border border-gray-300 sm:px-6 max-w-sm w-full rounded-lg shadow">
      <div class="flow-root" v-if="selectedSurahID">
        <div class="text-center font-bold flex items-center justify-center" v-if="selectedSurah">
          <span class="ml-2">{{ selectedSurah.transliteration }}</span>
          <span class="ml-2 arabic">{{ selectedSurah.name }}</span>
          <span class="ml-2">({{ selectedSurah.id }})</span>
        </div>
        <ul role="list">
          <li v-for="(verse, verseIdx) in timeline" :key="verse.id">
            <div class="relative pb-8">
              <span v-if="verseIdx !== timeline.length - 1" class="absolute top-4 right-4 -mr-px h-full w-0.5 bg-gray-200" aria-hidden="true"></span>
              <div class="relative flex space-x-3">
                <div class="min-w-0 flex-1 pt-1.5">
                  <div>
                    <p class="text-2xl text-gray-500 text-right arabic">
                      {{ verse.text }}
                    </p>
                  </div>
  
                  <div class="absolute -bottom-10 right-0">
                    <div class="text-right text-sm whitespace-nowrap text-gray-500" v-if="verseIdx == 0 && timeline.length == 1 && verse.id !== selectedSurah.total_verses">
                      <div class="flex items-center">
                        <span @click="getHint" class="h-8 w-8 bg-gray-100 border border-gray-300 text-gray-500 shadow rounded-full flex items-center justify-center cursor-pointer hover:bg-blue-500 hover:text-white">
                          <QuestionMarkCircleIcon class="text-sm h-5 w-5" />
                        </span>
                        <span @click="getNextVerse" class="h-8 w-8 ml-2 bg-gray-100 border border-gray-300 text-gray-500 shadow rounded-full flex items-center justify-center cursor-pointer hover:bg-blue-500 hover:text-white">
                          <ChevronDownIcon class="text-sm h-5 w-5" />
                        </span>
                      </div>
                    </div>
                    <div class="text-right text-sm whitespace-nowrap text-gray-500" v-else-if="verseIdx > 0 && verse.id === timeline[timeline.length -1].id">
                      <div class="flex items-center">
                        <span v-if="verse.id !== selectedSurah.total_verses" @click="getHint" class="h-8 w-8 bg-gray-100 border border-gray-300 text-gray-500 shadow rounded-full flex items-center justify-center cursor-pointer hover:bg-blue-500 hover:text-white">
                          <QuestionMarkCircleIcon class="text-sm h-5 w-5" />
                        </span>
                         <span @click="getNextVerse" class="h-8 w-8 ml-2 bg-gray-100 border border-gray-300 text-gray-500 shadow rounded-full flex items-center justify-center cursor-pointer hover:bg-blue-500 hover:text-white">
                          <ChevronDownIcon class="text-sm h-5 w-5" />
                        </span>
                        <span @click="missedIt(verse)" class="h-8 w-8 ml-2 bg-gray-100 border border-gray-300 text-gray-500 shadow rounded-full flex items-center justify-center cursor-pointer hover:bg-red-500 hover:text-white">
                          <LockOpenIcon class="text-sm h-5 w-5" />
                        </span>
                        <span @click="gotIt(verse)" class="h-8 w-8 ml-2 bg-gray-100 border border-gray-300 text-gray-500 shadow rounded-full flex items-center justify-center cursor-pointer hover:bg-green-500 hover:text-white">
                          <LockClosedIcon class="text-sm h-5 w-5" />
                        </span>
                      </div>
                    </div>
                  </div>
                </div>
                <div class="relative">
                  <ActionButtons :verse="verse"></ActionButtons>
                </div>
              </div>
            </div>
          </li>
        </ul>
      </div>
      
      <div v-if="!selectedSurahID">
        <div v-if="quran.length">
          <div v-for="surah in quran" class="py-1 hover:font-bold hover:cursor-pointer flex justify-between" @click="fetchSurah(surah.id)">
            <span>{{surah.id}}</span>
            <span>{{surah.transliteration}}</span>
            <span class="text-xl arabic">{{ surah.name }}</span>
          </div>
        </div>
      </div>
    </div>
  
    <div class="mt-2 mx-auto max-w-sm w-full text-gray-500 text-xs">
      <p>Legend</p>
      <div class="mt-1 flex items-center">
        <ChevronDownIcon class="h-4 w-4"></ChevronDownIcon>
        <p class="ml-2">Next verse</p>
      </div>
      <div class="mt-1 flex items-center">
        <QuestionMarkCircleIcon class="h-4 w-4"></QuestionMarkCircleIcon>
        <p class="ml-2">Get hint for next word</p>
      </div>
      <div class="mt-1 flex items-center">
        <LockClosedIcon class="h-4 w-4"></LockClosedIcon>
        <p class="ml-2">Memorization is strong</p>
      </div>
      <div class="mt-1 flex items-center">
        <LockOpenIcon class="h-4 w-4"></LockOpenIcon>
        <p class="ml-2">Memorization is weak</p>
      </div>
    </div>
  </div>

  <div class="absolute top-0 right-0 m-4 cursor-pointer" v-if="selectedSurahID" @click="resetApp">
    <button type="button" class="inline-flex items-center p-2 border border-transparent rounded-full shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
      <RefreshIcon class="h-4 w-4" aria-hidden="true" />
    </button>
  </div>

<!--   <pre>randomVerse: {{ randomVerse }}</pre>
  <pre>timeline: {{ timeline }}</pre>
  <pre>selectedSurah: {{ selectedSurah }}</pre> -->
</template>

<script>
import { ChevronDownIcon, QuestionMarkCircleIcon, RefreshIcon, LockClosedIcon, LockOpenIcon } from '@heroicons/vue/solid'
import { ref, onMounted } from 'vue'
import ActionButtons from './components/ActionButtons.vue'

  function randomIntFromRange(min, max) { // min and max included 
    return Math.floor(Math.random() * (max - min + 1) + min)
  }

  export default {
    components: {
      ActionButtons,
      ChevronDownIcon,
      QuestionMarkCircleIcon,
      RefreshIcon,
      LockClosedIcon,
      LockOpenIcon
    },
    data() {
      return {
        quran: [],
        timeline: [],
        selectedSurahID: null,
        selectedSurah: null,
        randomVerse: null
      }
    },
    async mounted() {
      const response = await fetch(`https://cdn.jsdelivr.net/npm/quran-json@3.1.2/dist/chapters/index.json`).then((r) => r.json())

      this.quran = ref(response)
    },
    methods: {
      async fetchSurah(id) {
        this.selectedSurahID = id

        this.selectedSurah = await fetch(`https://cdn.jsdelivr.net/npm/quran-json@3.1.2/dist/chapters/${id}.json`).then((r) => r.json())
        this.randomVerse = randomIntFromRange(1, this.selectedSurah.total_verses)
        this.timeline.push({
          status: 3,
          ...this.selectedSurah.verses[this.randomVerse-1]
        })
      },
      getNextVerse() {
        // check if we're already on the last verse
        if (this.timeline[this.timeline.length - 1].id === this.selectedSurah.total_verses) {
          console.log('last verse here')
          return;
        }

        // check if we're currently showing a hint
        if (this.timeline[this.timeline.length - 1].isHint) {
          const currentVerseIdx = this.timeline[this.timeline.length - 1].id - 1;
          this.timeline[this.timeline.length - 1].text = this.selectedSurah.verses[currentVerseIdx].text;
          this.timeline[this.timeline.length - 1].isHint = false;
        }
        
        const currentVerseCount = this.timeline.length;
        const nextVerseIdx = this.randomVerse + currentVerseCount - 1;
        // console.log({
        //   currentVerseCount,
        //   nextVerseIdx
        // })
        this.timeline.push({
          status: 3,
          ...this.selectedSurah.verses[nextVerseIdx]
        })
      },
      gotIt(verse) {
        verse.status = 1
        this.getNextVerse()
      },
      missedIt(verse) {
        verse.status = 2
        this.getNextVerse()
      },
      resetApp() {
        this.selectedSurahID = null
        this.selectedSurah = null
        this.randomVerse = null
        this.timeline = []
      },
      getHint() {
        // check if we're already on the last verse
        if (this.timeline[this.timeline.length - 1].id === this.selectedSurah.total_verses) {
          console.log('last verse here')
          return;
        }

        // check if we're already viewing a hint
        if (this.timeline[this.timeline.length - 1].isHint) {
          console.log('we here')
          const verseId = this.timeline[this.timeline.length - 1].id
          const currentWordCount = this.timeline[this.timeline.length - 1].text.split(" ").length;
          const newWordCount = currentWordCount + 1;
          this.timeline[this.timeline.length - 1].text = this.selectedSurah.verses[verseId - 1].text.split(' ').slice(0, currentWordCount + 1).join(' ')

          if (this.timeline[this.timeline.length - 1].text.split(' ').length == this.selectedSurah.verses[verseId - 1].text.split(' ').length) {
            this.timeline[this.timeline.length - 1].isHint = false
          }
        } else {
          const currentVerseCount = this.timeline.length;
          const nextVerseIdx = this.randomVerse + currentVerseCount - 1;
          const nextVerse = JSON.parse(JSON.stringify(this.selectedSurah.verses[nextVerseIdx]))
          nextVerse.text = nextVerse.text.split(' ')[0]
          
          this.timeline.push({
            status: 4,
            isHint: true,
            ...nextVerse
          })
        }
      }
    }
  }

</script>

<style>
body {
  background-color: rgb(243 244 246);
}
.arabic {
  font-family: 'Noto Naskh Arabic', serif;
}
</style>