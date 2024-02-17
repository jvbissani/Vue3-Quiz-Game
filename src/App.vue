<template>
  <div>

    <template v-if="question">

    <ScoreBoard :winCount="winCount" :loseCount="loseCount" />

    <!-- Category Selector -->
    <select class="select" v-model="selectedCategory" @change="getNewQuestion">
      <option v-for="category in categories" :key="category.id" :value="category.id">
        {{ category.name }}
      </option>
    </select>
    
      <h1 v-html="question"></h1>

      <template v-for="(answer, index) in answers" :key="index">
        <input :disabled="answerSubmitted" type="radio" name="options" :value="answer" v-model="chosenAnswer">
        <label v-html="answer"></label><br>
      </template>

      <button v-if="!answerSubmitted" @click="submitAnswer" class="send" type="button">Send</button>

      <section v-if="answerSubmitted" class="result">
        <h4 v-if="chosenAnswer == correctAnswer" v-html="'&#9989; Congratulations, the answer ' + correctAnswer + ' is correct.'"></h4>
        <h4 v-else v-html="'&#10060; Sorry, you picked the wrong answer. The correct one is ' + correctAnswer + '.'"></h4>
        <button @click="getNewQuestion" class="send" type="button">Next Question</button>
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
      questions: [],
      currentQuestionIndex: 0,
      chosenAnswer: undefined,
      answerSubmitted: false,
      winCount: 0,
      loseCount: 0,
      categories: [], // Armazena as categorias
      selectedCategory: null // Categoria selecionada
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
      let url = 'https://opentdb.com/api.php?amount=10';
      if (this.selectedCategory) {
        url += `&category=${this.selectedCategory}`;
      }
      console.log("URL do Request:", url); // Debugging: Ver a URL final
      this.axios.get(url)
        .then((response) => {
          this.questions = response.data.results;
          this.currentQuestionIndex = 0;
          this.resetQuestionState();
        })
        .catch(error => console.error("Erro ao buscar questões:", error));
    },
    resetQuestionState() {
      this.answerSubmitted = false;
      this.chosenAnswer = undefined;
    },
    fetchCategories() {
      this.axios.get('https://opentdb.com/api_category.php')
        .then((response) => {
          this.categories = response.data.trivia_categories;
          // Inicializa selectedCategory com o primeiro id de categoria para evitar problemas de seleção
          if (this.categories.length > 0) {
            this.selectedCategory = this.categories[0].id;
          }
        });
    }
  },
  created() {
    this.fetchCategories();
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

  input[type=radio] {
    margin: 12px 4px;
  }

  button.send {
    margin-top: 12px;
    height: 40px;
    min-width: 120px;
    padding: 0 16px;
    color: #fff;
    background-color: #1867c0;
    border: 1px solid #06448a;
    cursor: pointer;
  }
  
  .select{
    margin-top: 30px;
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
