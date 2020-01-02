<template>
  <div class="home">
    <div v-if="!quizRunning" class="quiz-settings">
      <h1>Improve Your Mental Math Power</h1>
      <h3>Quiz settings</h3>
      <div class="pure-g quiz-settings__title">
        <div class="pure-u-1-3">Operation</div>
        <div class="pure-u-1-3">Highest Limit</div>
        <div class="pure-u-1-3">Preview</div>
      </div>
      <div class="pure-g">
        <div class="pure-u-1-3 quiz-operator">
          <input type="checkbox" id="addition" value="addition" v-model="operation.addition.active" />
          <label for="addition">Addition</label>
        </div>
        <div class="pure-u-1-3">
          <div class="limit">
            <input type="number" v-model="operation.addition.firstLimit" />
            <input type="number" v-model="operation.addition.secondLimit" />
          </div>
        </div>
        <div class="pure-u-1-3">
          <div class="preview">
            <span class="preview__limit">{{operation.addition.firstLimit}}</span>
            <span class="preview__operator">+</span>
            <span class="preview__limit">{{operation.addition.secondLimit}}</span>
            <span class="preview__operator">=</span>
            <span
              class="preview__operator"
            >{{ parseInt(operation.addition.firstLimit) + parseInt(operation.addition.secondLimit) }}</span>
          </div>
        </div>
      </div>

      <div class="pure-g">
        <div class="pure-u-1-3 quiz-operator">
          <input
            type="checkbox"
            id="subtraction"
            value="subtraction"
            v-model="operation.subtraction.active"
          />
          <label for="subtraction">Subtraction</label>
        </div>
        <div class="pure-u-1-3">
          <div class="limit">
            <input type="number" v-model="operation.subtraction.firstLimit" />
            <input type="number" v-model="operation.subtraction.secondLimit" />
          </div>
        </div>
        <div class="pure-u-1-3">
          <div class="preview">
            <span class="preview__limit">{{operation.subtraction.firstLimit}}</span>
            <span class="preview__operator">-</span>
            <span class="preview__limit">{{operation.subtraction.secondLimit}}</span>
            <span class="preview__operator">=</span>
            <span
              class="preview__operator"
            >{{ parseInt(operation.subtraction.firstLimit) - parseInt(operation.subtraction.secondLimit) }}</span>
          </div>
        </div>
      </div>

      <div class="pure-g">
        <div class="pure-u-1-3 quiz-operator">
          <input
            type="checkbox"
            id="multiplication"
            value="multiplication"
            v-model="operation.multiplication.active"
          />
          <label for="multiplication">Multiplication</label>
        </div>
        <div class="pure-u-1-3">
          <div class="limit">
            <input type="number" v-model="operation.multiplication.firstLimit" />
            <input type="number" v-model="operation.multiplication.secondLimit" />
          </div>
        </div>
        <div class="pure-u-1-3">
          <div class="preview">
            <span class="preview__limit">{{operation.multiplication.firstLimit}}</span>
            <span class="preview__operator">x</span>
            <span class="preview__limit">{{operation.multiplication.secondLimit}}</span>
            <span class="preview__operator">=</span>
            <span
              class="preview__operator"
            >{{ parseInt(operation.multiplication.firstLimit) * parseInt(operation.multiplication.secondLimit) }}</span>
          </div>
        </div>
      </div>

      <div class="pure-g">
        <div class="pure-u-1-3 quiz-operator">
          <input type="checkbox" id="division" value="division" v-model="operation.division.active" />
          <label for="division">division</label>
        </div>
        <div class="pure-u-1-3">
          <div class="limit">
            <input type="number" v-model="operation.division.firstLimit" />
            <input type="number" v-model="operation.division.secondLimit" />
          </div>
        </div>
        <div class="pure-u-1-3">
          <div class="preview">
            <span class="preview__limit">{{operation.division.firstLimit}}</span>
            <span class="preview__operator">÷</span>
            <span class="preview__limit">{{operation.division.secondLimit}}</span>
            <span class="preview__operator">=</span>
            <span
              class="preview__operator"
            >{{ parseInt(operation.division.firstLimit) / parseInt(operation.division.secondLimit) }}</span>
          </div>
        </div>
      </div>

      <hr />

      <div class="pure-g">
        <div class="pure-u-1-3">
          <p>Total Time</p>
        </div>
        <div class="pure-u-2-3">
          <input type="text" v-model="quizTime" /> Seconds
        </div>
      </div>

      <div class="quiz-main-button">
        <button @click="startQuiz" class="main-button">Start</button>
      </div>
    </div>

    <div v-else class="quiz-body">
      <div>{{ quizTime }}</div>
      <form @submit.prevent="check">
        <span class="first-arg">{{ first }}</span>
        <span class="operator">{{ operator }}</span>
        <span class="second-arg">{{ second }}</span>
        =
        <input type="number" class="answer" v-model="answer" @keydown="typing" />
        <div class="feedback">{{ feedback }}</div>
      </form>
      Time: {{ individualTime }}s
      <dir>
        <button @click="stopQuiz">Stop</button>
      </dir>
    </div>

    <div v-if="logs.length && !quizRunning" class="quiz-log">
      <h3>Your quiz result</h3>
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
      quizTime: 60,
      quizTimer: null,
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
      logs: [],
      questions: []
    };
  },
  methods: {
    startQuiz() {
      this.startQuizTimer();
      this.quizRunning = true;
      this.newQuestion();
    },
    stopQuiz() {
      this.stopQuizTimer();
      this.quizRunning = false;
      this.stopIndividualTimer();
    },
    check() {
      this.result = false;
      let correct = false;

      switch (this.operator) {
        case "+":
          this.result = this.first + this.second;
          if (this.result === parseInt(this.answer)) {
            correct = true;
          }
          break;

        case "-":
          this.result = this.first - this.second;
          if (this.result === parseInt(this.answer)) {
            correct = true;
          }
          break;

        case "x":
          this.result = this.first * this.second;
          if (this.result === parseInt(this.answer)) {
            correct = true;
          }
          break;

        case "÷":
          this.result = this.first / this.second;
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
      this.individualTimer = setInterval(
        () => (this.individualTime += 1),
        1000
      );
    },
    stopIndividualTimer() {
      clearInterval(this.individualTimer);
      this.individualTime = 0;
    },
    startQuizTimer() {
      this.quizTimer = setInterval(() => (this.quizTime -= 1), 1000);
    },
    stopQuizTimer() {
      clearInterval(this.quizTimer);
      this.quizTime = 60;
    }
  },
  computed: {
    operatorMap() {
      return Object.keys(this.operation).filter(
        key => this.operation[key].active
      );
    }
  },
  watch: {
    quizTime(value) {
      if (parseInt(value) === 0) {
        this.stopQuiz();
      }
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
.quiz-settings__title > div {
  font-size: 18px;
}
.quiz-operator {
  display: flex;
  align-items: center;
}
.quiz-operator label {
  margin-left: 5px;
}
.preview {
  display: block;
}
.preview__limit {
  padding: 10px 30px;
  border: 1px solid black;
  display: inline-block;
}

.preview__operator {
  margin: 0 20px;
  display: inline-block;
}

.limit {
  margin-top: 10px;
}

.limit > input:last-child {
  margin-left: 10px;
}

.quiz-main-button {
  text-align: center;
  margin: 50px;
}

.main-button {
  font-size: 28px;
  padding: 5px 15px;
  border: 3px solid orange;
  border-radius: 30px;
}
</style>
