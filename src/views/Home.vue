<template>
  <div class="home">
    <div v-if="!quizRunning">
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

      <div>
        Total Time
        <input type="text" v-model="quizTime" /> minute
      </div>

      <dir>
        <button @click="startQuiz">Start</button>
      </dir>
    </div>

    <div v-else>
      <div>{{quizTime}}</div>
      <form @submit.prevent="check">
        <span class="first-arg">{{ first }}</span>
        <span class="operator">{{ operator }}</span>
        <span class="second-arg">{{ second }}</span>
        =
        <input
          type="number"
          class="answer"
          v-model="answer"
          @keydown="typing"
        />
        <div class="feedback">{{ feedback }}</div>
      </form>

      Time: {{ individualTime }}s

      <dir>
        <button @click="stopQuiz">Stop</button>
      </dir>
    </div>

    <div v-if="logs.length && !quizRunning">
      <table border="1">
        <tr>
          <th>Question</th>
          <th>Your Answer</th>
          <th>Correct Answer</th>
          <th>Time Taken</th>
        </tr>
        <tr v-for="(log, index) of logs" :key="index">
          <td>{{ log.first }} {{ log.operator }} {{ log.second }}</td>
          <td>{{ log.answer }}</td>
          <td>{{ log.result }}</td>
          <td>{{ log.time }}s</td>
        </tr>
      </table>
    </div>
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
      quizTime: 2,
      individualTime: 0,
      individualTimer: null,
      quizRunning: false,
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
          firstLimit: 12,
          secondLimit: 10
        },
        division: {
          symbol: "÷",
          active: true,
          firstLimit: 12,
          secondLimit: 10
        }
      },
      logs: [
        {
          first: 2,
          second: 3,
          operator: "x",
          answer: 6,
          result: 5,
          time: 1
        }
      ],
      questions: []
    };
  },
  methods: {
    startQuiz() {
      this.quizRunning = true;
      this.newQuestion();
    },
    stopQuiz() {
      this.quizRunning = false;
      this.stopIndividualTimer();
    },
    check() {
      this.result = false;
      let correct = false;

      switch (this.operator) {
        case "+":
          this.result = this.first + this.second
          if (this.result === parseInt(this.answer)) {
            correct = true;
          }
          break;

        case "-":
          this.result = this.first - this.second
          if (this.result === parseInt(this.answer)) {
            correct = true;
          }
          break;

        case "x":
          this.result = this.first * this.second
          if (this.result === parseInt(this.answer)) {
            correct = true;
          }
          break;

        case "÷":
          this.result = this.first / this.second
          if (this.result === parseInt(this.answer)) {
            correct = true;
          }
          break;

        default:
          this.result = "No operator is selected";
          break;
      }

      if (correct) {
        this.feedback = "Correct";
        this.saveToLog();
        this.stopIndividualTimer();
        this.answer = null;
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
      this.startIndividualTimer();

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
      if (this.answer) {
        this.logs.push({
          first: this.first,
          second: this.second,
          operator: this.operator,
          answer: this.answer,
          result: this.result,
          time: this.individualTime
        });
      }
    },
    startIndividualTimer() {
      this.individualTimer = setInterval(() => (this.individualTime += 1), 1000);
    },
    stopIndividualTimer() {
      clearInterval(this.individualTimer);
      this.individualTime = 0;
    }
  },
  computed: {
    operatorMap() {
      return Object.keys(this.operation).filter(
        key => this.operation[key].active
      );
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
