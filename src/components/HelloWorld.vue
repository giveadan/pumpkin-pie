<script setup>
import { ref } from 'vue'

const props = defineProps({
  msg: String,
})

const displayMsg = ref(props.msg)
const buttonState = ref('initial') // 'initial', 'joy', 'coworker', 'hidden'

// Reaction game state
const gameActive = ref(false)
const gameButtonVisible = ref(false)
const gameButtonPosition = ref({ top: '50%', left: '50%' })
const isRotating = ref(false)
const buttonAppearTime = ref(0)
const showFrowny = ref(false)
const isCheater = ref(false)
const userReactionTime = ref(0)

// Just okay flow state
const showDogImage = ref(false)
const dogImageUrl = ref('')
const availableCoworkers = ref(['Jean', 'John', 'Mike', 'Dan', 'Jon'])
const snarkyMessage = ref('')

const getRandomPosition = () => {
  // Generate random position within game area (620px x 220px)
  // Leave some padding for the button size (assuming ~120px button width, ~45px height)
  const buttonWidth = 120
  const buttonHeight = 45
  const maxLeft = 620 - buttonWidth - 20 // 20px padding from right edge
  const maxTop = 220 - buttonHeight - 20 // 20px padding from bottom edge
  const left = Math.random() * maxLeft + 10 // 10px to maxLeft
  const top = Math.random() * maxTop + 10 // 10px to maxTop
  return { top: `${top}px`, left: `${left}px` }
}

const handleGreat = () => {
  displayMsg.value = "Lets play a game. Click the button when it appears"
  buttonState.value = 'hidden'
  
  // Start reaction game
  gameActive.value = true
  const delay = Math.random() * 3000 + 2000 // Random between 2-5 seconds
  
  setTimeout(() => {
    gameButtonPosition.value = getRandomPosition()
    buttonAppearTime.value = Date.now()
    gameButtonVisible.value = true
  }, delay)
}

const handleGameButtonClick = (event) => {
  // Check if activated via keyboard (Tab + Enter) - detail is 0 for keyboard
  if (event.detail === 0) {
    // Cheater detected!
    gameButtonVisible.value = false
    displayMsg.value = "Nice try, cheater."
    showFrowny.value = true
    isCheater.value = true
    
    // After animation completes, refresh the page
    setTimeout(() => {
      window.location.reload()
    }, 3000)
    return
  }
  
  const reactionTime = Date.now() - buttonAppearTime.value
  userReactionTime.value = (reactionTime / 1000).toFixed(3) // Convert to seconds with 3 decimal places
  
  if (reactionTime <= 520) {
    // Fast enough! Continue with rotation in place
    isRotating.value = true
    
    // After 5 seconds, hide button and show final message
    setTimeout(() => {
      gameButtonVisible.value = false
      isRotating.value = false
      gameActive.value = false
      displayMsg.value = `Good job. ${userReactionTime.value} seconds`
    }, 5000)
  } else {
    // Too slow! Show frowny face
    gameButtonVisible.value = false
    displayMsg.value = `You're too slow. ${userReactionTime.value} seconds`
    showFrowny.value = true
    isCheater.value = false
    
    // After animation completes, refresh the page
    setTimeout(() => {
      window.location.reload()
    }, 3000) // 3 seconds for the emoji to grow and fill the screen
  }
}

const handleJustOkay = () => {
  displayMsg.value = "How much joy do you want to experience?"
  buttonState.value = 'joy'
}

const fetchInstagramImage = async () => {
  // Array of Instagram post IDs from @gimletfluff
  const instagramPosts = [
    'DQwv4J5EaRA',
    // Add more post IDs here as needed
  ]
  
  const randomPost = instagramPosts[Math.floor(Math.random() * instagramPosts.length)]
  
  // Strategy 1: Try Instagram's direct media endpoint
  const strategies = [
    `https://www.instagram.com/p/${randomPost}/media/?size=l`,
    // Strategy 2: Try with different size parameter
    `https://www.instagram.com/p/${randomPost}/media/`,
    // Strategy 3: Use third-party service (instagramez.com or similar)
    // Note: These services may require CORS proxy
  ]
  
  // For now, try the direct Instagram media URL
  // Instagram may block this due to CORS, but it's worth trying
  const mediaUrl = strategies[0]
  
  // Return the URL - if it fails to load, the browser will handle it
  // and we can show an error or fallback in the img onerror handler
  return mediaUrl
}

const handleTheMost = async () => {
  displayMsg.value = "Here's some joy!"
  dogImageUrl.value = await fetchInstagramImage()
  showDogImage.value = true
  buttonState.value = 'hidden'
}

const handleImageError = async () => {
  // Fallback to cat image if Instagram image fails to load
  try {
    const response = await fetch('https://api.thecatapi.com/v1/images/search')
    const data = await response.json()
    if (data && data[0] && data[0].url) {
      dogImageUrl.value = data[0].url
    } else {
      // Final fallback
      dogImageUrl.value = 'https://placekitten.com/800/600'
    }
  } catch (error) {
    // If API fails, use placekitten as final fallback
    dogImageUrl.value = 'https://placekitten.com/800/600'
  }
}

const handleOnlyALittle = () => {
  displayMsg.value = "You should talk to your favorite coworker..."
  buttonState.value = 'coworker'
  snarkyMessage.value = ''
}

const handleCoworkerSelect = (name) => {
  if (name === 'Dan') {
    displayMsg.value = "Yeah, talk to Dan."
    buttonState.value = 'hidden'
    snarkyMessage.value = ''
  } else {
    // Wrong answer - remove this option and show snarky message
    availableCoworkers.value = availableCoworkers.value.filter(c => c !== name)
    
    const snarkyResponses = {
      'Jean': "Jean is too cool to talk to you.",
      'John': "John is a great guy, but he's not your favorite.",
      'Mike': "lol good one.",
      'Jon': "Jon doesn't even know what time it is for you."
    }
    snarkyMessage.value = snarkyResponses[name] || "Wrong! Try again."
  }
}
</script>

<template>
  <h1>{{ displayMsg }}</h1>

  <div class="card" v-if="buttonState !== 'hidden'">
    <template v-if="buttonState === 'initial'">
      <button type="button" @click="handleGreat">Great</button>
      <button type="button" @click="handleJustOkay">Just okay</button>
    </template>
    <template v-else-if="buttonState === 'joy'">
      <button type="button" @click="handleTheMost">The most</button>
      <button type="button" @click="handleOnlyALittle">Only a little</button>
    </template>
    <template v-else-if="buttonState === 'coworker'">
      <div class="coworker-selection">
        <p v-if="snarkyMessage" class="snarky-message">{{ snarkyMessage }}</p>
        <div class="coworker-buttons">
          <button 
            v-for="name in availableCoworkers" 
            :key="name"
            type="button" 
            @click="handleCoworkerSelect(name)"
          >
            {{ name }}
          </button>
        </div>
      </div>
    </template>
  </div>

  <!-- Dog image display -->
  <div v-if="showDogImage" class="dog-image-container">
    <img :src="dogImageUrl" alt="A cute dog" class="dog-image" @error="handleImageError" />
  </div>

  <!-- Game area -->
  <div v-if="gameActive" class="game-area">
    <!-- Reaction game button -->
    <button 
      v-if="gameButtonVisible"
      type="button" 
      class="game-button"
      :class="{ rotating: isRotating }"
      :style="{ top: gameButtonPosition.top, left: gameButtonPosition.left }"
      @click="handleGameButtonClick($event)"
    >
      click me
    </button>
  </div>

  <!-- Emoji overlay for slow reaction or cheater -->
  <div v-if="showFrowny" class="frowny-overlay">
    <div class="frowny-emoji">{{ isCheater ? '😠' : '☹️' }}</div>
  </div>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
button {
  margin: 0 10px;
}

.coworker-selection {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.snarky-message {
  color: #ff4444;
  font-weight: bold;
  font-size: 18px;
  margin: 0;
  animation: shake 0.5s;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-10px); }
  75% { transform: translateX(10px); }
}

.coworker-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
}

.dog-image-container {
  margin: 40px auto;
  max-width: 800px;
  text-align: center;
}

.dog-image {
  max-width: 100%;
  height: auto;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.game-area {
  position: relative;
  width: 620px;
  height: 220px;
  background-color: rgba(0, 0, 0, 0.05);
  border-radius: 8px;
  margin: 40px auto;
}

.game-button {
  position: absolute;
  padding: 12px 24px;
  font-size: 16px;
  cursor: pointer;
  z-index: 1000;
}

.game-button.rotating {
  animation: accelerateRotation 5s linear;
}

@keyframes accelerateRotation {
  0% {
    transform: rotate(0deg);
    animation-timing-function: linear;
  }
  20% {
    transform: rotate(72deg);
  }
  40% {
    transform: rotate(288deg);
  }
  60% {
    transform: rotate(864deg);
  }
  80% {
    transform: rotate(2160deg);
  }
  100% {
    transform: rotate(7200deg);
  }
}

.frowny-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  pointer-events: none;
}

.frowny-emoji {
  font-size: 20px;
  animation: growEmoji 3s ease-in forwards;
}

@keyframes growEmoji {
  0% {
    font-size: 20px;
    opacity: 1;
  }
  100% {
    font-size: 2000px;
    opacity: 1;
  }
}
</style>
