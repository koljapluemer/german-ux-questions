<template>
  <div class="p-6 max-w-xl mx-auto bg-base-200 rounded shadow">

    <!-- Show the question with the first word replaced by ＿ -->
    <p class="mb-2 text-3xl m-2">
      <span class="font-semibold">Question:</span>
      <span v-if="!isAnswered">{{ questionWithBlank }}</span>
      <span v-else> {{ question }}</span>
    </p>


    <!-- Answer choices -->
    <div class="space-x-2 ">
      <button v-for="(answer, idx) in possibleAnswers" :key="idx" class="btn btn-lg" :class="getButtonClass(answer)"
        @click="selectAnswer(answer)">
        {{ answer }}
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, computed } from 'vue';
import questionsData from '@/assets/questions.json';

import { pickRandomQuestion } from '@/composables/pickQuestion';
import { scoreQuestion } from '@/composables/fsrsManager';

export default defineComponent({
  name: 'PracticeQuestion',
  data() {
    return {
      question: '',
      firstWord: '',
      randomFirstWord: '',
      possibleAnswers: [] as string[], // Holds the two answers in random order
      isAnswered: false,              // Whether user has selected an answer
      userAnswer: '',                 // Track the user's chosen answer
    };
  },
  computed: {
    // Return the question text with the first word replaced by ＿
    questionWithBlank(): string {
      if (!this.question) return '';
      const words = this.question.split(' ');
      if (words.length === 0) return '';
      words[0] = '＿＿＿';
      return words.join(' ');
    },
  },
  created() {
    this.getNewQuestion();
  },
  methods: {
    getNewQuestion() {
      this.isAnswered = false;
      this.userAnswer = '';

      // Pick a random question from your question set
      const { question, firstWord, randomFirstWord } = pickRandomQuestion(questionsData);

      this.question = question;
      this.firstWord = firstWord;
      this.randomFirstWord = randomFirstWord;

      // Randomize the order of the two possible answers
      const answers = [this.firstWord, this.randomFirstWord];
      this.possibleAnswers = answers.sort(() => Math.random() - 0.5);
    },
    selectAnswer(answer: string) {
      // If we've already answered, do nothing
      if (this.isAnswered) return;

      this.isAnswered = true;
      this.userAnswer = answer;

      // Determine correctness
      const isCorrect = (answer === this.firstWord);
      // Score the question in spaced repetition
      scoreQuestion(this.question, isCorrect);

      // Show the full question for 500ms, then load a new question
      setTimeout(() => {
        this.getNewQuestion();
      }, 800);
    },
    /**
     * Return a dynamic daisyUI class for each button
     */
    getButtonClass(answer: string): string {
      // If not answered yet, keep a neutral style
      if (!this.isAnswered) {
        return 'btn-primary';
      }
      // If answered, highlight correct or wrong
      const isCorrect = (answer === this.firstWord);
      const isChosen = (answer === this.userAnswer);

      // If this is the chosen answer and it's correct => green
      if (isChosen && isCorrect) return 'btn-success';
      // If this is the chosen answer and it's wrong => red
      if (isChosen && !isCorrect) return 'btn-error';
      // If not chosen, and correct => subtle indicator or neutral
      // Some prefer highlighting the correct answer in green anyway:
      // if (isCorrect) return 'btn-success';
      return 'btn-ghost';
    },
  },
});
</script>

<style scoped>
/* Additional overrides or custom classes, if needed */
</style>
