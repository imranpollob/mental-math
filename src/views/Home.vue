<template>
  <div class="home">
    <form @submit.prevent="check">
      <span class="first-arg">{{first}}</span>
      <span class="operator">{{operator}}</span>
      <span class="second-arg">{{second}}</span>
      =
      <input type="number" class="answer" v-model="answer" @keydown="typing"/>
      <div class="feedback">{{feedback}}</div>
    </form>
  </div>
</template>

<script>
export default {
  name: "home",
  data() {
    return {
      first: 1,
      second: 2,
      operator: "+",
      answer: null,
      feedback: ""
    };
  },
  methods: {
    check() {
      this.feedback =
        this.first + this.second == this.answer ? "Correct" : "Opps";

      if (this.feedback == "Correct") {
        // setTimeout(() => {
          this.first += 1;
          this.second += 1;
          this.answer = null;
        // }, 1000);
      }
    },
    typing() {
      this.feedback = ''
    }
  }
};
</script>

<style scoped>
.first-arg,
.second-arg,
.oparator,
.answer,
.feedback {
  font-size: 24px;
}

.feedback{
  transition: opacity 1s linear;;
}
</style>
