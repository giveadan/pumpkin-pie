<script setup>
import { ref } from 'vue'

const props = defineProps({
  msg: String,
})

const displayMsg = ref(props.msg)
const buttonState = ref('initial') // 'initial', 'joy', 'coworker', 'ranking', 'hidden'

// Reaction game state
const gameActive = ref(false)
const gameButtonVisible = ref(false)
const gameButtonPosition = ref({ top: '50%', left: '50%' })
const isRotating = ref(false)
const buttonAppearTime = ref(0)
const showFrowny = ref(false)
const isCheater = ref(false)
const userReactionTime = ref(0)
const showTryAgain = ref(false)

// Just okay flow state
const showDogImage = ref(false)
const dogImageUrl = ref('')
const availableCoworkers = ref(['Jean', 'John', 'Mike', 'Dan', 'Jon'])
const snarkyMessage = ref('')
const showCountdown = ref(false)
const countdownValue = ref(5)

// Ranking system state
const selectedChoice = ref(null) // Single choice slot
const availableChoices = ref(['Containerization', 'Mega Monitor', 'Office Snacks', 'Dan'])
const draggedItem = ref(null)
const rankingError = ref('')

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

const startReactionGame = () => {
  // Reset game state
  gameButtonVisible.value = false
  isRotating.value = false
  showFrowny.value = false
  showTryAgain.value = false
  gameActive.value = true
  
  displayMsg.value = "Fast typer, but slow... clicker?"
  
  const delay = Math.random() * 3000 + 2000 // Random between 2-5 seconds
  
  setTimeout(() => {
    gameButtonPosition.value = getRandomPosition()
    buttonAppearTime.value = Date.now()
    gameButtonVisible.value = true
  }, delay)
}

const handleGreat = () => {
  displayMsg.value = "Fast typer, but slow... clicker?"
  buttonState.value = 'hidden'
  startReactionGame()
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
    // Too slow! Show frowny face and try again option
    gameButtonVisible.value = false
    displayMsg.value = `You're too slow. ${userReactionTime.value} seconds`
    showFrowny.value = true
    isCheater.value = false
    
    // After animation completes, show try again button
    setTimeout(() => {
      showFrowny.value = false
      showTryAgain.value = true
    }, 3000) // 3 seconds for the emoji to grow
  }
}

const handleTryAgain = () => {
  startReactionGame()
}

const handleJustOkay = () => {
  displayMsg.value = "Just wanna have fun."
  buttonState.value = 'hidden'
  showCountdown.value = true
  countdownValue.value = 5
  
  const countdownInterval = setInterval(() => {
    countdownValue.value--
    
    if (countdownValue.value <= 0) {
      clearInterval(countdownInterval)
      // Redirect to Girls Just Want to Have Fun music video
      window.location.href = 'https://www.youtube.com/watch?v=PIb6AZdTr-A'
    }
  }, 1000)
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

// Ranking system handlers
const handleNotAChallenge = () => {
  displayMsg.value = "What's most important to you?"
  buttonState.value = 'ranking'
  // Reset ranking state
  selectedChoice.value = null
  availableChoices.value = ['Containerization', 'Mega Monitor', 'Office Snacks', 'Dan']
  rankingError.value = ''
}

const handleDragStart = (event, choice, source) => {
  draggedItem.value = { choice, source }
  event.dataTransfer.effectAllowed = 'move'
}

const handleDragOver = (event) => {
  event.preventDefault()
  event.dataTransfer.dropEffect = 'move'
}

const handleDrop = (event) => {
  event.preventDefault()
  if (!draggedItem.value) return
  
  const { choice, source } = draggedItem.value
  
  // If slot already has a choice, swap it back to available
  if (selectedChoice.value) {
    availableChoices.value.push(selectedChoice.value)
  }
  
  // Place the dragged choice in the slot
  selectedChoice.value = choice
  
  // Remove from available
  if (source === 'available') {
    availableChoices.value = availableChoices.value.filter(c => c !== choice)
  }
  
  draggedItem.value = null
}

const handleDropToAvailable = (event) => {
  event.preventDefault()
  if (!draggedItem.value) return
  
  const { choice, source } = draggedItem.value
  
  // Only allow moving back from slot to available
  if (source === 'slot') {
    selectedChoice.value = null
    availableChoices.value.push(choice)
  }
  
  draggedItem.value = null
}

const handleSubmitRanking = () => {
  // Check if Dan is selected
  if (selectedChoice.value === 'Dan') {
    displayMsg.value = "❤️"
    buttonState.value = 'hidden'
    rankingError.value = ''
  } else {
    // Clear error momentarily to retrigger the shake animation
    rankingError.value = ''
    setTimeout(() => {
      rankingError.value = "Wrong"
    }, 10)
  }
}
</script>

<template>
  <h1>{{ displayMsg }}</h1>

  <!-- Countdown display -->
  <div v-if="showCountdown" class="countdown-display">
    <h2 class="countdown-number">{{ countdownValue }}</h2>
  </div>

  <div class="card" v-if="buttonState !== 'hidden'">
    <template v-if="buttonState === 'initial'">
      <button type="button" @click="handleGreat">Challenge</button>
      <button type="button" @click="handleNotAChallenge">Challenge 2</button>
      <button type="button" @click="handleJustOkay">Why</button>
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
    <template v-else-if="buttonState === 'ranking'">
      <div class="ranking-container">
        <p v-if="rankingError" class="ranking-error">{{ rankingError }}</p>
        
        <!-- Single choice slot -->
        <div class="single-choice-slot"
             @dragover="handleDragOver"
             @drop="handleDrop"
        >
          <div 
            v-if="selectedChoice"
            class="ranking-card filled"
            draggable="true"
            @dragstart="handleDragStart($event, selectedChoice, 'slot')"
          >
            {{ selectedChoice }}
          </div>
          <div v-else class="ranking-card empty large">
            Drop your choice here
          </div>
        </div>

        <!-- Available choices -->
        <div class="available-names">
          <h3>Available Options:</h3>
          <div 
            class="available-names-container"
            @dragover="handleDragOver"
            @drop="handleDropToAvailable"
          >
            <div 
              v-for="choice in availableChoices" 
              :key="choice"
              class="ranking-card draggable"
              draggable="true"
              @dragstart="handleDragStart($event, choice, 'available')"
            >
              {{ choice }}
            </div>
          </div>
        </div>

        <!-- Submit button -->
        <button type="button" class="submit-ranking" @click="handleSubmitRanking">
          Submit
        </button>
      </div>
    </template>
  </div>

  <!-- Try again button after failing reaction game -->
  <div v-if="showTryAgain" class="card">
    <button type="button" @click="handleTryAgain">Try again</button>
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

.countdown-display {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 40px 0;
}

.countdown-number {
  font-size: 120px;
  font-weight: bold;
  color: #646cff;
  margin: 0;
  animation: pulse 1s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.1);
    opacity: 0.8;
  }
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

/* Ranking system styles */
.ranking-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 30px;
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
}

.ranking-error {
  color: #ff4444;
  font-weight: bold;
  font-size: 18px;
  margin: 0;
  animation: shake 0.5s;
}

.single-choice-slot {
  width: 100%;
  display: flex;
  justify-content: center;
  padding: 20px;
}

.ranking-card {
  padding: 12px 20px;
  border-radius: 8px;
  font-size: 16px;
  text-align: center;
  min-width: 120px;
  transition: all 0.2s;
}

.ranking-card.filled {
  background-color: #42b983;
  color: white;
  cursor: move;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.ranking-card.filled:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.ranking-card.empty {
  background-color: rgba(0, 0, 0, 0.05);
  border: 2px dashed #ccc;
  color: #999;
  cursor: default;
}

.ranking-card.large {
  min-width: 250px;
  padding: 24px 32px;
  font-size: 18px;
}

.ranking-card.draggable {
  background-color: #646cff;
  color: white;
  cursor: move;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.ranking-card.draggable:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.available-names {
  width: 100%;
}

.available-names h3 {
  font-size: 16px;
  margin: 0 0 12px 0;
  text-align: center;
}

.available-names-container {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
  padding: 20px;
  border: 2px dashed #ccc;
  border-radius: 8px;
  background-color: rgba(0, 0, 0, 0.02);
  min-height: 60px;
}

.submit-ranking {
  padding: 12px 40px;
  font-size: 18px;
  font-weight: bold;
  margin-top: 10px;
}
</style>
