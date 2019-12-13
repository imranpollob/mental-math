<template>
  <div class="home">
    <form @submit.prevent="check">
      <span class="first-arg" >{{first}}</span>
      <span class="operator">{{operator}}</span>
      <span class="first-arg">{{second}}</span>
      <input type="number" v-model="answer">
      <div>{{feedback}}</div>
    </form>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'

export default {
  name: 'home',
  components: {
    HelloWorld
  },
  data() {
    return {
      first: 1,
      second: 2,
      operator: '+',
      answer: null,
      feedback: ''
    }
  },
  methods: {
    check() {
      this.feedback = this.first + this.second == this.answer ? 'Correct' : 'Opps'
    }
  },
}
</script>
