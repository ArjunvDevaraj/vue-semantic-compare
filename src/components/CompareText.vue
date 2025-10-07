<script setup>
import { ref, computed } from 'vue'
import { GoogleGenerativeAI } from '@google/generative-ai'

const textA = ref('')
const textB = ref('')
const score = ref(null)
const diffResults = ref([])

// Initialize Gemini client
const genAI = new GoogleGenerativeAI(import.meta.env.VITE_GEMINI_API_KEY)
const model = genAI.getGenerativeModel({ model: 'text-embedding-004' })

// Compute cosine similarity
const cosineSimilarity = (vecA, vecB) => {
  const dot = vecA.reduce((sum, a, i) => sum + a * vecB[i], 0)
  const magA = Math.sqrt(vecA.reduce((sum, a) => sum + a * a, 0))
  const magB = Math.sqrt(vecB.reduce((sum, b) => sum + b * b, 0))
  return dot / (magA * magB)
}

const compareTexts = async () => {
  if (!textA.value || !textB.value) return

  try {
    const [resA, resB] = await Promise.all([
      model.embedContent(textA.value),
      model.embedContent(textB.value)
    ])

    const embeddingA = resA.embedding.values
    const embeddingB = resB.embedding.values
    score.value = cosineSimilarity(embeddingA, embeddingB)
    generateDiff()
  } catch (error) {
    console.error('Error fetching Gemini embeddings:', error)
  }
}

const generateDiff = () => {
  const aSentences = textA.value.split(/[\.\n]+/).map(s => s.trim()).filter(Boolean)
  const bSentences = textB.value.split(/[\.\n]+/).map(s => s.trim()).filter(Boolean)
  diffResults.value = []

  const aSet = new Set(aSentences)
  const bSet = new Set(bSentences)

  aSentences.forEach(sentence => {
    if (!bSet.has(sentence)) diffResults.value.push({ text: sentence, type: 'removed' })
  })

  bSentences.forEach(sentence => {
    if (!aSet.has(sentence)) diffResults.value.push({ text: sentence, type: 'added' })
  })

  const common = aSentences.filter(s => bSet.has(s))
  common.forEach(sentence => diffResults.value.push({ text: sentence, type: 'same' }))
}

const scoreDisplay = computed(() => score.value ? (score.value * 100).toFixed(2) + '%' : '--')
const scoreColor = computed(() => {
  if (score.value > 0.8) return 'text-green-600 dark:text-green-400'
  if (score.value > 0.5) return 'text-yellow-600 dark:text-yellow-400'
  return 'text-red-600 dark:text-red-400'
})
</script>

<template>
  <div class="p-6 rounded-2xl bg-white shadow-sm border border-gray-100 dark:bg-gray-900 dark:border-gray-800">
    <h2 class="text-lg font-semibold mb-6 text-center">Compare Two Texts (Gemini)</h2>

    <!-- Inputs -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <div>
        <label class="block text-sm font-medium mb-1">Text A</label>
        <textarea
          v-model="textA"
          class="w-full h-40 p-3 border rounded-lg focus:ring-2 focus:ring-blue-500 bg-gray-50 dark:bg-gray-800 dark:border-gray-700 resize-none"
          placeholder="Enter or paste the first text..."
        ></textarea>
      </div>
      <div>
        <label class="block text-sm font-medium mb-1">Text B</label>
        <textarea
          v-model="textB"
          class="w-full h-40 p-3 border rounded-lg focus:ring-2 focus:ring-blue-500 bg-gray-50 dark:bg-gray-800 dark:border-gray-700 resize-none"
          placeholder="Enter or paste the second text..."
        ></textarea>
      </div>
    </div>

    <!-- Compare Button -->
    <div class="mt-8 flex flex-col sm:flex-row items-center justify-center gap-4">
      <button
        @click="compareTexts"
        class="px-8 py-2.5 rounded-lg bg-blue-600 text-white font-medium hover:bg-blue-700 transition-all shadow-md"
      >
        🔍 Compare Texts
      </button>

      <span v-if="score !== null" class="text-gray-700 dark:text-gray-300">
        Similarity Score:
        <strong :class="scoreColor">{{ scoreDisplay }}</strong>
      </span>
    </div>

    <!-- Score Bar -->
    <div v-if="score !== null" class="mt-6">
      <div class="h-2 w-full bg-gray-200 dark:bg-gray-700 rounded-full overflow-hidden">
        <div
          class="h-2 bg-blue-500 transition-all duration-500 ease-out"
          :style="{ width: `${score * 100}%` }"
        ></div>
      </div>
    </div>

    <!-- Diff Results -->
    <div v-if="diffResults.length" class="mt-8">
      <h3 class="text-md font-semibold mb-4 text-center">Sentence Comparison</h3>
      <div class="space-y-2">
        <div
          v-for="(item, index) in diffResults"
          :key="index"
          class="p-2 rounded-md text-sm"
          :class="{
            'bg-green-100 text-green-800 dark:bg-green-900/40 dark:text-green-300': item.type === 'added',
            'bg-red-100 text-red-800 dark:bg-red-900/40 dark:text-red-300': item.type === 'removed',
            'bg-gray-100 text-gray-700 dark:bg-gray-800 dark:text-gray-200': item.type === 'same'
          }"
        >
          <span class="font-medium capitalize">{{ item.type }}:</span> {{ item.text }}
        </div>
      </div>
    </div>
  </div>
</template>
