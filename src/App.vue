<script setup lang="ts">
import GameHeader from './components/GameHeader.vue'
import GameFigure from './components/GameFigure.vue'
import GameWrongLetters from './components/GameWrongLetters.vue'
import GameWord from './components/GameWord.vue'
import GamePopup from './components/GamePopup.vue'
import GameNotification from './components/GameNotification.vue'
import { computed, ref, watch } from 'vue'
import axios from 'axios'

const word = ref('')
const letters = ref<string[]>([])
const correctLetters = computed(() => letters.value.filter((x) => word.value.includes(x)))
const wrongLetters = computed(() => letters.value.filter((x) => !word.value.includes(x)))
const filteredWrongLetters = computed(() =>
  wrongLetters.value.filter((item, index) => {
    return wrongLetters.value.indexOf(item) === index
  })
)
const isLose = computed(() => filteredWrongLetters.value.length === 6)
const isWin = computed(() => [...word.value].every((x) => correctLetters.value.includes(x)))
const notification = ref<InstanceType<typeof GameNotification> | null>(null)
const popup = ref<InstanceType<typeof GamePopup> | null>(null)

const getRandomName = async () => {
  try {
    const { data } = await axios<{ FirstName: string }>(
      'https://api.randomdatatools.ru/?unescaped=false&params=FirstName'
    )
    word.value = data.FirstName.toLowerCase()
  } catch (err) {
    console.log(err)
    word.value = ''
  }
}

getRandomName()

watch(filteredWrongLetters, () => {
  if (isLose.value) {
    popup.value?.open('lose')
  }
})

watch(correctLetters, () => {
  if (isWin.value) {
    popup.value?.open('win')
  }
})

window.addEventListener('keydown', ({ key }) => {
  if (isLose.value || isWin.value) {
    return
  }
  if (letters.value.includes(key)) {
    notification.value?.open()
    setTimeout(() => {
      notification.value?.close()
    }, 2000)
  }

  if (/[а-яА-ЯёË]/.test(key)) {
    letters.value.push(key.toLowerCase())
  }
})

const restart = async () => {
  await getRandomName()
  letters.value = []
  popup.value?.close()
}
</script>

<template>
  <GameHeader />
  <div class="game-container">
    <GameFigure :wrongLettersCount="filteredWrongLetters.length" />
    <GameWrongLetters :wrongLetters="filteredWrongLetters" />
    <GameWord :word="word" :correctLetters="correctLetters" />
  </div>
  <GamePopup ref="popup" :word="word" @restart="restart" />
  <GameNotification ref="notification" />
</template>
