<template>

  <div>

    <ScoreBoard :winCount="this.winCount" :loseCount="this.loseCount" />

    <template v-if="this.question">

      <h1 v-html="this.question">
      </h1>

      <template v-for="(answer, index) in this.answers" :key="index">
      <input
        :disabled="this.answerSubmitted"
        type="radio" 
        name="options" 
        :value="answer"
        v-model="this.chosenAnswer"
      >
      
      <label v-html="answer"></label><br>
      </template>

      <button 
      v-if="!this.answerSubmitted"
      @click="this.submitAnswer()" 
      class="send" 
      type="button">Send
      </button>

      <section v-if="this.answerSubmitted" class="result">

        <h4 v-if="this.chosenAnswer == this.correctAnswer"
        v-html="'&#9989; Congratulations, the answer ' + this.correctAnswer + ' is correct.'">
        
        </h4>

        <h4 v-else
        v-html="'&#10060; Im sorry, you picked the wrong answer. The correct is ' + this.correctAnswer + '.'">
        
        </h4>

        <button @click="this.getNewQuestion()" class="send" type="button">Next Question</button>

      </section>

    </template>

  </div>
</template>

<script>
  
import ScoreBoard from '@/components/ScoreBoard.vue'

export default {
  name: 'App',
  components: {
    ScoreBoard
  },
  data() {
    return {
      questions: [], //Questions Array
      currentQuestionIndex: 0,
      chosenAnswer: undefined,
      answerSubmitted: false,
      winCount: 0,
      loseCount: 0
    }
  },
  computed: {
    question() {
      return this.questions[this.currentQuestionIndex]?.question;
    },
    incorrectAnswers() {
      return this.questions[this.currentQuestionIndex]?.incorrect_answers;
    },
    correctAnswer() {
      return this.questions[this.currentQuestionIndex]?.correct_answer;
    },
    answers() {
      const answers = this.incorrectAnswers ? [...this.incorrectAnswers] : [];
      answers.splice(Math.round(Math.random() * answers.length), 0, this.correctAnswer);
      return answers;
    }
  },
  methods: {
    submitAnswer() {
      if (!this.chosenAnswer) {
        alert('Pick one of the options.');
      } else {
        this.answerSubmitted = true;
        if (this.chosenAnswer == this.correctAnswer) {
          this.winCount++;
        } else {
          this.loseCount++;
        }
      }
    },
    getNewQuestion() {
      // Completed all the questions, do the request again.
      if (this.currentQuestionIndex >= this.questions.length - 1) {
        this.axios.get('https://opentdb.com/api.php?amount=10')
          .then((response) => {
            this.questions = response.data.results;
            this.currentQuestionIndex = 0;
            this.resetQuestionState();
          });
      } else {
        this.currentQuestionIndex++;
        this.resetQuestionState();
      }
    },
    resetQuestionState() {
      this.answerSubmitted = false;
      this.chosenAnswer = undefined;
    }
  },
  created() {
    this.getNewQuestion();
  }
}
</script>

<style lang="scss">

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 60px auto;
  max-width: 960px;
  
input[type=radio]{
  margin: 12px 4px;
}

button.send{
  margin-top: 12px;
  height: 40px;
  min-width: 120px;
  padding: 0 16px;
  color: #fff;
  background-color: #1867c0;
  border: 1px solid #06448a;
  cursor: pointer;
}

}
</style>
