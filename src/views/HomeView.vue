<template>
  <div class="page">
    <header class="hero" v-if="!quizRunning">
      <h1 class="hero__eyebrow">Ultimate</h1>
      <h1 class="hero__title">Mental Math Trainer</h1>
      <h2 class="hero__subtitle">Build sharper number sense with adaptive drills.</h2>
    </header>

    <section v-if="!quizRunning" class="settings">
      <div class="settings__row">
        <label class="settings__label" for="difficulty">Difficulty</label>
        <select id="difficulty" v-model="selectedDifficulty" class="settings__select">
          <option v-for="option in difficultyOptions" :key="option.id" :value="option.id">
            {{ option.label }}
          </option>
        </select>
      </div>
      <p class="settings__description">{{ selectedDifficultyDescription }}</p>

      <div class="settings__row">
        <label class="settings__label" for="quiz-time">Quiz length (seconds)</label>
        <input
          id="quiz-time"
          type="number"
          min="30"
          step="15"
          v-model.number="quizTime"
          class="settings__input"
        />
      </div>

      <p class="settings__hint">Mix and match the skills you want to drill and choose number ranges below.</p>
    </section>

    <section v-if="!quizRunning" class="operations">
      <h3 class="operations__title">Practice focus</h3>
      <div class="operations__grid">
        <article
          v-for="([key, definition], index) in operationEntries"
          :key="key"
          class="operation-card"
        >
          <div class="operation-card__header">
            <label>
              <input type="checkbox" v-model="operations[key].active" />
              <span class="operation-card__label">{{ definition.label }}</span>
            </label>
            <span class="operation-card__symbol">{{ definition.symbol }}</span>
          </div>
          <p class="operation-card__description">{{ definition.description }}</p>

          <div
            v-for="control in definition.rangeControls"
            :key="`${key}-${control.key}`"
            class="operation-card__control"
          >
            <label :for="`${key}-${control.key}`">{{ control.label }}</label>
            <select
              class="operation-card__select"
              :id="`${key}-${control.key}`"
              v-model="operations[key][control.key]"
            >
              <option
                v-for="option in rangeOptionsByType(control.type)"
                :key="option.id"
                :value="option.id"
              >
                {{ option.label }}
              </option>
            </select>
          </div>

          <footer class="operation-card__footer">
            <span class="operation-card__preview-label">Example</span>
            <span class="operation-card__preview">{{ previewForOperation(key) }}</span>
          </footer>
        </article>
      </div>

      <p v-if="setupError" class="operations__error">{{ setupError }}</p>

      <div class="operations__actions">
        <button class="primary" @click="startQuiz">Start training</button>
      </div>
    </section>

    <section v-else class="quiz">
      <header class="quiz__header">
        <div class="quiz__timer">
          <span class="quiz__timer-label">Time left</span>
          <span class="quiz__timer-value">{{ formattedTimeLeft }}</span>
        </div>
        <div class="quiz__meta">
          <span>Current skill: {{ currentOperationLabel }}</span>
          <span>Question time: {{ individualTime }}s</span>
        </div>
        <button class="secondary" @click="stopQuiz">End session</button>
      </header>

      <div class="quiz__prompt">
        <span>{{ currentQuestion.prompt }}</span>
      </div>

      <form class="quiz__form" @submit.prevent="checkAnswer">
        <label class="quiz__answer-label" for="answer">Your answer</label>
        <input
          id="answer"
          ref="answerField"
          type="text"
          autocomplete="off"
          class="quiz__answer"
          v-model="answer"
          @input="clearFeedback"
        />
        <button type="submit" class="primary">Check</button>
      </form>
      <p v-if="feedback" class="quiz__feedback">{{ feedback }}</p>
    </section>

    <section v-if="logs.length" class="results">
      <h3 class="results__title">Latest session</h3>
      <table class="results__table">
        <thead>
          <tr>
            <th>#</th>
            <th>Prompt</th>
            <th>Your answer</th>
            <th>Correct answer</th>
            <th>Time (s)</th>
            <th>Result</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(log, index) in logs" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ log.prompt }}</td>
            <td>{{ log.userAnswer }}</td>
            <td>{{ log.correctAnswer }}</td>
            <td>{{ log.time }}</td>
            <td>
              <span :class="{ 'result--correct': log.correct, 'result--wrong': !log.correct }">
                {{ log.correct ? '✓' : '✕' }}
              </span>
            </td>
          </tr>
        </tbody>
      </table>
    </section>
  </div>
</template>

<script setup>
import { computed, onBeforeUnmount, reactive, ref, watch } from 'vue'

const numberRangeOptions = [
  { id: 'single', label: 'Single-digit (1-9)', min: 1, max: 9 },
  { id: 'double', label: 'Two-digit (10-99)', min: 10, max: 99 },
  { id: 'triple', label: 'Three-digit (100-999)', min: 100, max: 999 }
]

const percentRangeOptions = [
  { id: 'percent-basic', label: '5% - 50% (step 5)', min: 5, max: 50, step: 5 },
  { id: 'percent-advanced', label: '10% - 90% (step 5)', min: 10, max: 90, step: 5 }
]

const numberRangeMap = createRangeLookup(numberRangeOptions)
const percentRangeMap = createRangeLookup(percentRangeOptions)

const operationDefinitions = {
  addition: {
    label: 'Addition',
    symbol: '+',
    description: 'Combine numbers quickly and reinforce core number facts.',
    rangeControls: [
      { key: 'firstRange', label: 'First number', type: 'number' },
      { key: 'secondRange', label: 'Second number', type: 'number' }
    ],
    defaultState: { firstRange: 'double', secondRange: 'double' },
    generate: state => {
      const first = pickNumber(state.firstRange)
      const second = pickNumber(state.secondRange)
      return {
        prompt: `${first} + ${second}`,
        operator: '+',
        first,
        second,
        answer: first + second
      }
    }
  },
  subtraction: {
    label: 'Subtraction',
    symbol: '−',
    description: 'Strengthen subtraction fluency with controlled ranges.',
    rangeControls: [
      { key: 'firstRange', label: 'Starting number', type: 'number' },
      { key: 'secondRange', label: 'Number to subtract', type: 'number' }
    ],
    defaultState: { firstRange: 'double', secondRange: 'double' },
    generate: state => {
      let first = pickNumber(state.firstRange)
      let second = pickNumber(state.secondRange)
      if (second > first) {
        ;[first, second] = [second, first]
      }
      return {
        prompt: `${first} − ${second}`,
        operator: '−',
        first,
        second,
        answer: first - second
      }
    }
  },
  multiplication: {
    label: 'Multiplication',
    symbol: '×',
    description: 'Train rapid recall of products in your chosen range.',
    rangeControls: [
      { key: 'firstRange', label: 'First factor', type: 'number' },
      { key: 'secondRange', label: 'Second factor', type: 'number' }
    ],
    defaultState: { firstRange: 'single', secondRange: 'double' },
    generate: state => {
      const first = pickNumber(state.firstRange)
      const second = pickNumber(state.secondRange)
      return {
        prompt: `${first} × ${second}`,
        operator: '×',
        first,
        second,
        answer: first * second
      }
    }
  },
  division: {
    label: 'Division',
    symbol: '÷',
    description: 'Master quotients that divide cleanly for mental calculations.',
    rangeControls: [
      { key: 'firstRange', label: 'Quotient range', type: 'number' },
      { key: 'secondRange', label: 'Divisor range', type: 'number' }
    ],
    defaultState: { firstRange: 'single', secondRange: 'double' },
    generate: state => {
      const quotient = pickNumber(state.firstRange)
      let divisor = pickNumber(state.secondRange)
      if (divisor === 0) divisor = 1
      const dividend = quotient * divisor
      return {
        prompt: `${dividend} ÷ ${divisor}`,
        operator: '÷',
        first: dividend,
        second: divisor,
        answer: quotient
      }
    }
  },
  percentage: {
    label: 'Percentages',
    symbol: '%',
    description: 'Calculate percentages with tidy, real-world friendly numbers.',
    rangeControls: [
      { key: 'firstRange', label: 'Percent range', type: 'percent' },
      { key: 'secondRange', label: 'Base number range', type: 'number' }
    ],
    defaultState: { firstRange: 'percent-basic', secondRange: 'double' },
    generate: state => {
      const percent = pickPercent(state.firstRange)
      const base = pickNumber(state.secondRange)
      const raw = (percent * base) / 100
      const answer = Number.isInteger(raw) ? raw : Number(raw.toFixed(2))
      return {
        prompt: `${percent}% of ${base}`,
        operator: '%',
        first: percent,
        second: base,
        answer
      }
    }
  },
  square: {
    label: 'Squares',
    symbol: '²',
    description: 'Remember key squares for faster calculations.',
    rangeControls: [{ key: 'firstRange', label: 'Number range', type: 'number' }],
    defaultState: { firstRange: 'single' },
    generate: state => {
      const value = pickNumber(state.firstRange)
      return {
        prompt: `${value}²`,
        operator: '²',
        first: value,
        second: null,
        answer: value * value
      }
    }
  },
  squareRoot: {
    label: 'Square Roots',
    symbol: '√',
    description: 'Identify perfect squares and their roots instantly.',
    rangeControls: [{ key: 'firstRange', label: 'Root range', type: 'number' }],
    defaultState: { firstRange: 'single' },
    generate: state => {
      const root = pickNumber(state.firstRange)
      const radicand = root * root
      return {
        prompt: `√${radicand}`,
        operator: '√',
        first: radicand,
        second: null,
        answer: root
      }
    }
  }
}

const operationEntries = Object.entries(operationDefinitions)

const difficultyPresets = {
  easy: {
    id: 'easy',
    label: 'Easy',
    description: 'Single-digit addition and subtraction in a 60 second burst.',
    time: 60,
    operations: {
      addition: { active: true, firstRange: 'single', secondRange: 'single' },
      subtraction: { active: true, firstRange: 'single', secondRange: 'single' }
    }
  },
  medium: {
    id: 'medium',
    label: 'Medium',
    description: 'Two-digit addition/subtraction with single-digit multiplication (90s).',
    time: 90,
    operations: {
      addition: { active: true, firstRange: 'double', secondRange: 'double' },
      subtraction: { active: true, firstRange: 'double', secondRange: 'double' },
      multiplication: { active: true, firstRange: 'single', secondRange: 'double' },
      square: { active: true, firstRange: 'single' }
    }
  },
  hard: {
    id: 'hard',
    label: 'Hard',
    description: 'Mix of multiplication, clean division, squares and percentages (120s).',
    time: 120,
    operations: {
      addition: { active: true, firstRange: 'double', secondRange: 'double' },
      subtraction: { active: true, firstRange: 'double', secondRange: 'double' },
      multiplication: { active: true, firstRange: 'double', secondRange: 'double' },
      division: { active: true, firstRange: 'single', secondRange: 'double' },
      percentage: { active: true, firstRange: 'percent-basic', secondRange: 'double' },
      square: { active: true, firstRange: 'double' }
    }
  },
  expert: {
    id: 'expert',
    label: 'Expert',
    description: 'Broad challenge with advanced ranges, roots and percentages (150s).',
    time: 150,
    operations: {
      addition: { active: true, firstRange: 'triple', secondRange: 'triple' },
      subtraction: { active: true, firstRange: 'triple', secondRange: 'triple' },
      multiplication: { active: true, firstRange: 'double', secondRange: 'double' },
      division: { active: true, firstRange: 'double', secondRange: 'double' },
      percentage: { active: true, firstRange: 'percent-advanced', secondRange: 'triple' },
      square: { active: true, firstRange: 'double' },
      squareRoot: { active: true, firstRange: 'double' }
    }
  }
}

const difficultyOptions = [
  ...Object.values(difficultyPresets),
  { id: 'custom', label: 'Custom', description: 'Set your own mix of skills and pacing.' }
]

const operations = reactive(createInitialOperations())
const selectedDifficulty = ref('easy')
const quizTime = ref(difficultyPresets.easy.time)
const currentQuizTime = ref(quizTime.value)
const quizRunning = ref(false)
const answer = ref('')
const feedback = ref('')
const setupError = ref('')
const logs = ref([])
const individualTime = ref(0)
const quizTimer = ref(null)
const individualTimer = ref(null)
const isApplyingPreset = ref(false)
const answerField = ref(null)

const currentQuestion = reactive({
  key: '',
  prompt: '',
  operator: '',
  first: null,
  second: null,
  answer: null
})

applyDifficultyPreset(selectedDifficulty.value)

watch(selectedDifficulty, value => {
  if (value === 'custom') {
    return
  }
  applyDifficultyPreset(value)
})

watch(
  () => quizTime.value,
  () => {
    if (!isApplyingPreset.value && selectedDifficulty.value !== 'custom') {
      selectedDifficulty.value = 'custom'
    }
  }
)

watch(
  operations,
  () => {
    if (!isApplyingPreset.value && selectedDifficulty.value !== 'custom') {
      selectedDifficulty.value = 'custom'
    }
  },
  { deep: true }
)

watch(
  () => currentQuizTime.value,
  value => {
    if (quizRunning.value && value <= 0) {
      finishQuiz()
    }
  }
)

const activeOperationCount = computed(
  () => Object.values(operations).filter(operation => operation.active).length
)

const selectedDifficultyDescription = computed(() => {
  if (selectedDifficulty.value === 'custom') {
    return 'Tweak the operations and ranges below to design your own session.'
  }
  return difficultyPresets[selectedDifficulty.value]?.description ?? ''
})

const formattedTimeLeft = computed(() => formatTime(currentQuizTime.value))

const currentOperationLabel = computed(() => {
  if (!currentQuestion.key) {
    return '—'
  }
  return operationDefinitions[currentQuestion.key]?.label ?? 'Practice'
})

function createInitialOperations() {
  const initial = {}
  for (const [key, definition] of Object.entries(operationDefinitions)) {
    initial[key] = {
      active: false,
      ...definition.defaultState
    }
  }
  return initial
}

function resetOperationsToDefaults() {
  for (const [key, definition] of Object.entries(operationDefinitions)) {
    const state = operations[key]
    state.active = false
    Object.assign(state, definition.defaultState)
  }
}

function rangeOptionsByType(type) {
  return type === 'percent' ? percentRangeOptions : numberRangeOptions
}

function previewForOperation(key) {
  const definition = operationDefinitions[key]
  if (!definition) return ''
  try {
    const state = operations[key]
    const preview = definition.generate(state)
    return preview.prompt
  } catch (error) {
    return ''
  }
}

function applyDifficultyPreset(id) {
  const preset = difficultyPresets[id]
  if (!preset) {
    return
  }
  isApplyingPreset.value = true
  resetOperationsToDefaults()
  quizTime.value = preset.time
  currentQuizTime.value = preset.time
  Object.entries(preset.operations).forEach(([key, config]) => {
    if (!operations[key]) return
    operations[key].active = config.active ?? true
    if (config.firstRange) {
      operations[key].firstRange = config.firstRange
    }
    if (config.secondRange) {
      operations[key].secondRange = config.secondRange
    }
  })
  isApplyingPreset.value = false
}

function startQuiz() {
  if (!activeOperationCount.value) {
    setupError.value = 'Select at least one operation to begin your training.'
    return
  }
  if (!quizTime.value || quizTime.value < 15) {
    setupError.value = 'Set the quiz length to at least 15 seconds.'
    return
  }
  setupError.value = ''
  logs.value = []
  quizRunning.value = true
  feedback.value = ''
  answer.value = ''
  currentQuizTime.value = Math.round(quizTime.value)
  setNextQuestion()
  startQuizTimer()
  focusAnswerField()
}

function stopQuiz() {
  finishQuiz()
}

function finishQuiz() {
  quizRunning.value = false
  clearInterval(quizTimer.value)
  clearInterval(individualTimer.value)
  quizTimer.value = null
  individualTimer.value = null
}

function startQuizTimer() {
  clearInterval(quizTimer.value)
  quizTimer.value = window.setInterval(() => {
    currentQuizTime.value = Math.max(0, currentQuizTime.value - 1)
  }, 1000)
}

function startIndividualTimer() {
  clearInterval(individualTimer.value)
  individualTime.value = 0
  individualTimer.value = window.setInterval(() => {
    individualTime.value += 1
  }, 1000)
}

function setNextQuestion() {
  const question = generateQuestion()
  if (!question) {
    finishQuiz()
    return
  }
  currentQuestion.key = question.key
  currentQuestion.prompt = question.prompt
  currentQuestion.operator = question.operator ?? ''
  currentQuestion.first = question.first ?? null
  currentQuestion.second = question.second ?? null
  currentQuestion.answer = question.answer
  startIndividualTimer()
}

function generateQuestion() {
  const available = Object.entries(operations)
    .filter(([, value]) => value.active)
    .map(([key]) => key)

  if (!available.length) {
    return null
  }

  const selectedKey = available[randomInt(0, available.length - 1)]
  const definition = operationDefinitions[selectedKey]
  const state = operations[selectedKey]
  if (!definition || !state) {
    return null
  }

  const generated = definition.generate(state)
  return {
    ...generated,
    key: selectedKey
  }
}

function checkAnswer() {
  if (!quizRunning.value) {
    return
  }
  const userInput = answer.value.trim()
  if (userInput === '') {
    feedback.value = 'Enter an answer to continue.'
    return
  }
  const numericInput = Number(userInput)
  const expected = currentQuestion.answer
  const correct = !Number.isNaN(numericInput) && isCorrectAnswer(numericInput, expected)

  logs.value.unshift({
    prompt: currentQuestion.prompt,
    userAnswer: userInput,
    correctAnswer: formatAnswer(expected),
    time: individualTime.value,
    correct
  })

  if (correct) {
    feedback.value = 'Correct!'
    answer.value = ''
    setNextQuestion()
  } else {
    feedback.value = 'Keep trying!'
  }
}

function clearFeedback() {
  feedback.value = ''
}

function focusAnswerField() {
  requestAnimationFrame(() => {
    answerField.value?.focus()
  })
}

function formatTime(totalSeconds) {
  const minutes = Math.floor(totalSeconds / 60)
  const seconds = totalSeconds % 60
  if (minutes <= 0) {
    return `${seconds}s`
  }
  return `${minutes}:${seconds.toString().padStart(2, '0')}`
}

function formatAnswer(value) {
  if (Number.isInteger(value)) {
    return value.toString()
  }
  return value.toFixed(2)
}

function isCorrectAnswer(userValue, expectedValue) {
  if (Number.isInteger(expectedValue)) {
    return Math.round(userValue) === expectedValue
  }
  return Math.abs(userValue - expectedValue) <= 0.01
}

function pickNumber(rangeId) {
  const range = numberRangeMap[rangeId] || numberRangeOptions[0]
  return randomInt(range.min, range.max)
}

function pickPercent(rangeId) {
  const range = percentRangeMap[rangeId] || percentRangeOptions[0]
  const step = range.step ?? 1
  const choices = Math.floor((range.max - range.min) / step)
  const offset = randomInt(0, choices)
  return range.min + offset * step
}

function randomInt(min, max) {
  const low = Math.ceil(min)
  const high = Math.floor(max)
  return Math.floor(Math.random() * (high - low + 1)) + low
}

function createRangeLookup(options) {
  return options.reduce((accumulator, option) => {
    accumulator[option.id] = option
    return accumulator
  }, {})
}

onBeforeUnmount(() => {
  clearInterval(quizTimer.value)
  clearInterval(individualTimer.value)
})
</script>

<style scoped>
.page {
  padding: 40px 5vw 80px;
  color: #f7f5f0;
}

.hero {
  text-align: center;
  margin-bottom: 32px;
}

.hero__eyebrow {
  text-transform: uppercase;
  letter-spacing: 6px;
  font-size: 14px;
  color: #a1a6c4;
  margin: 0 0 8px;
}

.hero__title {
  font-family: 'Cinzel', serif;
  font-size: 54px;
  color: #ffb347;
  margin: 0;
}

.hero__subtitle {
  font-weight: 400;
  margin: 16px 0 0;
  color: #dcd5f5;
}

.settings {
  background: rgba(16, 16, 28, 0.7);
  border-radius: 18px;
  padding: 24px;
  margin-bottom: 32px;
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.25);
}

.settings__row {
  display: flex;
  flex-direction: column;
  margin-bottom: 16px;
}

.settings__label {
  font-weight: 600;
  margin-bottom: 6px;
}

.settings__select,
.settings__input {
  padding: 10px 12px;
  border-radius: 10px;
  border: none;
  background: rgba(255, 255, 255, 0.1);
  color: inherit;
}

.settings__select:focus,
.settings__input:focus,
.operation-card__select:focus,
.quiz__answer:focus {
  outline: 2px solid #ffb347;
}

.settings__description {
  margin: 0 0 16px;
  color: #d3d8f5;
}

.settings__hint {
  margin: 0;
  font-size: 14px;
  color: #c3c9e6;
}

.operations {
  background: rgba(16, 16, 28, 0.7);
  border-radius: 18px;
  padding: 24px;
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.25);
}

.operations__title {
  margin-top: 0;
  margin-bottom: 20px;
}

.operations__grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 16px;
}

.operation-card {
  background: rgba(255, 255, 255, 0.06);
  border-radius: 16px;
  padding: 16px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  min-height: 220px;
}

.operation-card__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
  font-weight: 600;
}

.operation-card__label {
  margin-left: 6px;
}

.operation-card__symbol {
  font-size: 24px;
  opacity: 0.8;
}

.operation-card__description {
  font-size: 14px;
  color: #c4c8e4;
  margin: 0 0 10px;
}

.operation-card__control {
  display: flex;
  flex-direction: column;
  margin-bottom: 8px;
}

.operation-card__select {
  margin-top: 4px;
  padding: 8px;
  border-radius: 10px;
  border: none;
  background: rgba(255, 255, 255, 0.1);
  color: inherit;
}

.operation-card__footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 13px;
  margin-top: auto;
  padding-top: 12px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.operation-card__preview-label {
  text-transform: uppercase;
  letter-spacing: 2px;
  color: #a3a7c6;
}

.operation-card__preview {
  font-weight: 600;
}

.operations__actions {
  text-align: center;
  margin-top: 24px;
}

.operations__error {
  margin-top: 16px;
  color: #ff8a80;
  text-align: center;
}

.primary,
.secondary {
  cursor: pointer;
  border: none;
  border-radius: 999px;
  padding: 12px 28px;
  font-size: 16px;
  font-weight: 600;
}

.primary {
  background: linear-gradient(135deg, #ffb347, #ffcc33);
  color: #221a2d;
}

.secondary {
  background: rgba(255, 255, 255, 0.16);
  color: inherit;
}

.quiz {
  background: rgba(16, 16, 28, 0.7);
  border-radius: 18px;
  padding: 32px;
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.25);
}

.quiz__header {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  margin-bottom: 24px;
}

.quiz__timer {
  display: flex;
  flex-direction: column;
  font-size: 18px;
}

.quiz__timer-label {
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 2px;
  color: #a3a7c6;
}

.quiz__timer-value {
  font-size: 28px;
  font-weight: 700;
}

.quiz__meta {
  display: flex;
  flex-direction: column;
  gap: 4px;
  font-size: 14px;
  color: #c4c8e4;
}

.quiz__prompt {
  text-align: center;
  font-size: 42px;
  font-weight: 700;
  margin-bottom: 24px;
}

.quiz__form {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.quiz__answer-label {
  font-size: 14px;
  text-transform: uppercase;
  letter-spacing: 2px;
  color: #a3a7c6;
}

.quiz__answer {
  width: 200px;
  font-size: 28px;
  text-align: center;
  padding: 8px 12px;
  border-radius: 12px;
  border: none;
  background: rgba(255, 255, 255, 0.1);
  color: inherit;
}

.quiz__feedback {
  margin-top: 12px;
  text-align: center;
  font-weight: 600;
}

.results {
  margin-top: 40px;
}

.results__title {
  margin-bottom: 16px;
}

.results__table {
  width: 100%;
  border-collapse: collapse;
  background: rgba(16, 16, 28, 0.7);
  border-radius: 18px;
  overflow: hidden;
}

.results__table th,
.results__table td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}

.results__table thead {
  background: rgba(255, 255, 255, 0.08);
}

.result--correct {
  color: #7cff8b;
}

.result--wrong {
  color: #ff8a80;
}

@media (max-width: 720px) {
  .page {
    padding: 24px 16px 48px;
  }

  .operations__grid {
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  }

  .quiz__prompt {
    font-size: 32px;
  }

  .quiz__form {
    width: 100%;
  }

  .quiz__answer {
    width: 100%;
  }
}
</style>
