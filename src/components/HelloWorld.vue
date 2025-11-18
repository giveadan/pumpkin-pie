<script setup>
import { ref } from 'vue'

const props = defineProps({
  msg: String,
})

const displayMsg = ref(props.msg)
const buttonState = ref('initial') // 'initial', 'yesno', 'hidden'

const handleGreat = () => {
  displayMsg.value = "That's wonderful."
  buttonState.value = 'hidden'
}

const handleJustOkay = () => {
  displayMsg.value = "Would you like to talk about it?"
  buttonState.value = 'yesno'
}

const handleYes = () => {
  displayMsg.value = "I'm not your therapist."
  buttonState.value = 'hidden'
}

const handleNo = () => {
  displayMsg.value = "Good. It's healthy to keep it bottled in."
  buttonState.value = 'hidden'
}
</script>

<template>
  <h1>{{ displayMsg }}</h1>

  <div class="card" v-if="buttonState !== 'hidden'">
    <template v-if="buttonState === 'initial'">
      <button type="button" @click="handleGreat">Great</button>
      <button type="button" @click="handleJustOkay">Just okay</button>
    </template>
    <template v-else-if="buttonState === 'yesno'">
      <button type="button" @click="handleYes">Yes</button>
      <button type="button" @click="handleNo">No</button>
    </template>
  </div>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
button {
  margin: 0 10px;
}
</style>
