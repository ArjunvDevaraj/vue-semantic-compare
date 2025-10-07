<template>
  <div class="min-h-screen bg-gray-50 dark:bg-gray-950 text-gray-800 dark:text-gray-100 py-12 px-4">
    <div class="max-w-4xl mx-auto">
      <!-- Project Intro -->
      <div class="text-center mb-12">
        <h1 class="text-3xl font-bold mb-4">Semantic Text Comparison (Gemini API)</h1>
        <p class="text-gray-600 dark:text-gray-400 max-w-2xl mx-auto">
          This tool uses <strong>Google Gemini embeddings</strong> to compare two texts based on their <em>semantic meaning</em>,
          not just matching words. It returns a similarity score and highlights the sentence differences.
        </p>
      </div>

      <!-- How It Works -->
      <div class="mb-10 bg-white dark:bg-gray-900 p-6 rounded-2xl shadow-sm border border-gray-100 dark:border-gray-800">
        <h2 class="text-xl font-semibold mb-3">How It Works</h2>
        <ul class="list-disc list-inside text-sm text-gray-700 dark:text-gray-300 space-y-2">
          <li>Each text is converted into a <strong>vector embedding</strong> using Google’s Gemini API.</li>
          <li>The tool calculates <strong>cosine similarity</strong> between these embeddings.</li>
          <li>A similarity score (0–100%) indicates how semantically close the two texts are.</li>
          <li>Sentence-level differences are categorized as
            <span class="text-green-600 font-medium">added</span>,
            <span class="text-red-600 font-medium">removed</span>, or
            <span class="text-gray-600 font-medium">same</span>.
          </li>
        </ul>
      </div>

      <!-- Main Component -->
      <CompareText />

      <!-- Footer -->
      <p class="mt-10 text-center text-sm text-gray-500 dark:text-gray-400">
        Built with ❤️ using <strong>Vue 3</strong> + <strong>Google Gemini API</strong>.
      </p>
    </div>
  </div>
</template>

<script setup>
import CompareText from './components/CompareText.vue'
</script>
