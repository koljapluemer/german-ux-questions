<template>
  <div class="p-6 max-w-xl mx-auto bg-base-200 rounded shadow">
    <h2 class="text-2xl font-bold mb-4">Practice Question</h2>

    <!-- Show the question with the first word replaced by ＿ -->
    <p class="mb-2 text-3xl">
      <span class="font-semibold">Question:</span>
      <span v-if="!isAnswered">{{ questionWithBlank }}</span>

      <span v-else> {{ question }}</span>
    </p>

    <!-- Answer choices -->
    <div class="space-x-2 mb-4">
      <button v-for="(answer, idx) in possibleAnswers" :key="idx" class="btn btn-lg" :class="getButtonClass(answer)"
        @click="selectAnswer(answer)">
        {{ answer }}
      </button>
    </div>

    <!-- "Show Next" button if audio is playing -->
    <div v-if="audio" class="mt-2">
      <button class="btn btn-ghost" @click="showNext">Show Next</button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
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
      possibleAnswers: [] as string[], // The two answers, shuffled
      isAnswered: false,              // Has the user selected an answer?
      userAnswer: '',                 // The user’s chosen answer
      audio: null as HTMLAudioElement | null, // Audio object
    };
  },
  computed: {
    /**
     * Returns the question text with the first word replaced by ＿
     */
    questionWithBlank(): string {
      if (!this.question) return '';
      const words = this.question.split(' ');
      if (words.length > 0) {
        words[0] = '＿＿＿';
      }
      return words.join(' ');
    },
  },
  created() {
    this.getNewQuestion();
  },
  methods: {
    /**
     * Load a new question, reset state, shuffle the two possible answers.
     */
    getNewQuestion() {
      this.isAnswered = false;
      this.userAnswer = '';
      this.stopAudio(); // Stop any ongoing audio, remove event listeners

      const { question, firstWord, randomFirstWord } = pickRandomQuestion(questionsData);

      this.question = question;
      this.firstWord = firstWord;
      this.randomFirstWord = randomFirstWord;

      // Shuffle the two possible answers
      const answers = [this.firstWord, this.randomFirstWord];
      this.possibleAnswers = answers.sort(() => Math.random() - 0.5);
    },

    /**
     * Called when a user selects one of the answer buttons.
     */
    selectAnswer(answer: string) {
      // If we've already answered, do nothing
      if (this.isAnswered) return;

      this.isAnswered = true;
      this.userAnswer = answer;

      // Determine correctness and score
      const isCorrect = (answer === this.firstWord);
      scoreQuestion(this.question, isCorrect);

      // After revealing correctness, play audio
      this.playAudio();
    },

    /**
     * Construct the audio source from the question text.
     * You may need to adjust for special characters or spaces.
     */
    getAudioSrc(question: string): string {
      // Use encodeURIComponent if your filenames have special chars/spaces:
      return `/audio/${encodeURIComponent(question)}.mp3`;
    },

    /**
     * Play the audio for the current question.
     */
    playAudio() {
      // Stop any current audio
      this.stopAudio();

      this.audio = new Audio(this.getAudioSrc(this.question));
      this.audio.addEventListener('ended', this.handleAudioEnded);
      this.audio.play().catch((err) => {
        // Handle any errors from play() if needed
        console.error('Error playing audio:', err);
      });
    },

    /**
     * Stop and cleanup the current audio object, if any.
     */
    stopAudio() {
      if (this.audio) {
        this.audio.pause();
        this.audio.removeEventListener('ended', this.handleAudioEnded);
        this.audio = null;
      }
    },

    /**
     * Handler for audio's 'ended' event.
     * Proceed automatically to the next question.
     */
    handleAudioEnded() {
      this.stopAudio();
      this.getNewQuestion();
    },

    /**
     * "Show Next" button action: stop audio early, move to next question
     */
    showNext() {
      this.stopAudio();
      this.getNewQuestion();
    },

    /**
     * Dynamically style the answer buttons using daisyUI classes.
     */
    getButtonClass(answer: string): string {
      // If user hasn't answered yet, keep a neutral style
      if (!this.isAnswered) {
        return 'btn-primary';
      }
      // If answered, highlight correct or wrong
      const isCorrect = answer === this.firstWord;
      const isChosen = answer === this.userAnswer;

      if (isChosen && isCorrect) return 'btn-success'; // correct chosen
      if (isChosen && !isCorrect) return 'btn-error';   // wrong chosen
      return 'btn-ghost'; // Not chosen or unselected
    },
  },

  /**
   * Ensure we stop audio if the component unmounts
   * (cleanup to avoid memory leaks)
   */
  beforeUnmount() {
    this.stopAudio();
  },
});
</script>

<style scoped>
/* Tailwind or other overrides here if needed. */
</style>
