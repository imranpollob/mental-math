<template>
  <div class="home">
    <div>
      <input
        type="checkbox"
        id="addition"
        value="addition"
        v-model="operation.addition.active"
      />
      <label for="addition">Addition</label>
      <input type="text" v-model="operation.addition.firstLimit" />
      <input type="text" v-model="operation.addition.secondLimit" />
    </div>

    <div>
      <input
        type="checkbox"
        id="subtraction"
        value="subtraction"
        v-model="operation.subtraction.active"
      />
      <label for="subtraction">Subtraction</label>
      <input type="text" v-model="operation.subtraction.firstLimit" />
      <input type="text" v-model="operation.subtraction.secondLimit" />
    </div>

    <div>
      <input
        type="checkbox"
        id="multiplication"
        value="multiplication"
        v-model="operation.multiplication.active"
      />
      <label for="multiplication">Multiplication</label>
      <input type="text" v-model="operation.multiplication.firstLimit" />
      <input type="text" v-model="operation.multiplication.secondLimit" />
    </div>

    <div>
      <input
        type="checkbox"
        id="division"
        value="division"
        v-model="operation.division.active"
      />
      <label for="division">Division</label>
      <input type="text" v-model="operation.division.firstLimit" />
      <input type="text" v-model="operation.division.secondLimit" />
    </div>

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
const r = require("random-int");

export default {
  name: "home",
  data() {
    return {
      first: null,
      second: null,
      operator: null,
      answer: null,
      result: false,
      feedback: "",
      operation: {
        addition: {
          symbol: "+",
          active: true,
          firstLimit: 100,
          secondLimit: 100
        },
        subtraction: {
          symbol: "-",
          active: true,
          firstLimit: 100,
          secondLimit: 100
        },
        multiplication: {
          symbol: "x",
          active: true,
          firstLimit: 100,
          secondLimit: 100
        },
        division: {
          symbol: "÷",
          active: true,
          firstLimit: 100,
          secondLimit: 100
        }
      },
      log: [],
      questions: []
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
    generateQuestion() {
      let possibleOperator = this.operatorMap[r(this.operatorMap.length - 1)];

      return {
        possibleFirst: r(
          1,
          parseInt(this.operation[possibleOperator].firstLimit)
        ),
        possibleSecond: r(
          1,
          parseInt(this.operation[possibleOperator].secondLimit)
        ),
        possibleOperator: this.operation[possibleOperator].symbol
      };
    },
    newQuestion() {
      let {
        possibleFirst,
        possibleSecond,
        possibleOperator
      } = this.generateQuestion();

      let match = `${possibleFirst}${possibleOperator}${possibleSecond}`;

      if (this.questions.includes(match)) {
        this.newQuestion();
      } else {
        this.questions.push(match);

        if (possibleOperator === "÷") {
          this.first = possibleFirst * possibleSecond;
        } else {
          this.first = possibleFirst;
        }
        this.second = possibleSecond;
        this.operator = possibleOperator;
      }
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
      return Object.keys(this.operation).filter(
        key => this.operation[key].active
      );
    }
  },
  created() {
    this.newQuestion();
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
