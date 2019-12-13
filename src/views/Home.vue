<template>
  <div class="home">
    <div>
      Limit of first argument
      <input type="text" />
    </div>
    <div>
      Limit of second argument
      <input type="text" />
    </div>

    Operations:

    <input type="checkbox" id="addition" value="addition" v-model="operation" />
    <label for="addition">Addition</label>

    <input
      type="checkbox"
      id="subtraction"
      value="subtraction"
      v-model="operation"
    />
    <label for="subtraction">Subtraction</label>

    <input
      type="checkbox"
      id="multiplication"
      value="multiplication"
      v-model="operation"
    />
    <label for="multiplication">Multiplication</label>

    <input type="checkbox" id="division" value="division" v-model="operation" />
    <label for="division">Division</label>
    <br />
    <span>Checked names: {{ operation }}</span>

    <form @submit.prevent="check">
      <span class="first-arg">{{ first }}</span>
      <span class="operator">{{ operator }}</span>
      <span class="second-arg">{{ second }}</span>
      =
      <input type="number" class="answer" v-model="answer" @keydown="typing" />
      <div class="feedback">{{ feedback }}</div>
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
      feedback: "",
      firstLimit: 0,
      secondLimit: 0,
      operation: ["addition"]
    };
  },
  methods: {
    check() {
      switch (this.operator) {
        case "addition":
          this.feedback =
            this.first + this.second == this.answer ? "Correct" : "Opps";
          break;

        case "subtraction":
          this.feedback =
            this.first - this.second == this.answer ? "Correct" : "Opps";
          break;

        case "multiplication":
          this.feedback =
            this.first * this.second == this.answer ? "Correct" : "Opps";
          break;

        case "divison":
          this.feedback =
            this.first / this.second == this.answer ? "Correct" : "Opps";
          break;

        default:
          this.feedback = "No operator is selected";
          break;
      }

      if (this.feedback == "Correct") {
        // setTimeout(() => {
        this.first += 1;
        this.second += 1;
        this.answer = null;
        // }, 1000);
      }
    },
    typing() {
      this.feedback = "";
    },
    newQuestion() {
      this.first = Math.floor(Math.random() * Math.floor(firstLimit));
      this.second = Math.floor(Math.random() * Math.floor(secondLimit));
      this.operator = this.operation[
        Math.floor(Math.random() * Math.floor(this.operation.length))
      ];
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

.feedback {
  /* transition: opacity 1s linear;; */
}
</style>
