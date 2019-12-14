<template>
  <div class="home">
    <div>
      Limit of first argument
      <input type="text" v-model="firstLimit"/>
    </div>
    <div>
      Limit of second argument
      <input type="text" v-model="secondLimit"/>
    </div>

    Operations:

    <input
      type="checkbox"
      id="addition"
      value="addition"
      v-model="operation.addition"
    />
    <label for="addition">Addition</label>

    <input
      type="checkbox"
      id="subtraction"
      value="subtraction"
      v-model="operation.subtraction"
    />
    <label for="subtraction">Subtraction</label>

    <input
      type="checkbox"
      id="multiplication"
      value="multiplication"
      v-model="operation.multiplication"
    />
    <label for="multiplication">Multiplication</label>

    <input
      type="checkbox"
      id="division"
      value="division"
      v-model="operation.division"
    />
    <label for="division">Division</label>
    <br />
    <br />
    <br />

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
      result: false,
      feedback: "",
      firstLimit: 12,
      secondLimit: 10,
      operation: {
        addition: true,
        subtraction: true,
        multiplication: true,
        division: false
      },
      symbol: {
        addition: "+",
        subtraction: "-",
        multiplication: "x",
        division: "÷"
      },
      log: [
        {
          first: 4,
          second: 5,
          operator: "+",
          answer: 9,
          result: true,
          time: 3
        }
      ]
    };
  },
  methods: {
    check() {
      this.result = false;

      switch (this.operator) {
        case "+":
          this.result = this.first + this.second == this.answer;
          break;

        case "-":
          this.result = this.first - this.second == this.answer;
          break;

        case "X":
          this.result = this.first * this.second == this.answer;
          break;

        case "÷":
          this.result = this.first / this.second == this.answer;
          break;

        default:
          this.result = "No operator is selected";
          break;
      }

      if (this.result) {
        this.first += 1;
        this.second += 1;
        this.answer = null;
        this.feedback = "Correct";

        this.saveToLog();
        this.newQuestion();
      } else {
        this.saveToLog();
        this.feedback = "Opps";
      }
    },
    typing() {
      this.feedback = "";
    },
    newQuestion() {
      this.first = Math.floor(Math.random() * Math.floor(this.firstLimit));
      this.second = Math.floor(Math.random() * Math.floor(this.secondLimit));
      this.operator = this.operatorMap[
        Math.floor(Math.random() * Math.floor(this.operatorMap.length))
      ];
    },
    saveToLog() {
      this.log.push({
        first: this.first,
        second: this.second,
        operator: this.operator,
        answer: this.answer,
        result: this.result,
        time: 3
      });
    }
  },
  computed: {
    operatorMap() {
      let map = [];

      Object.keys(this.operation)
        .filter(key => this.operation[key])
        .map(key => {
          map.push(this.symbol[key]);
        });

      return map;
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
</style>
