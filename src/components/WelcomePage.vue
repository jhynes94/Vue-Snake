<template>
  <div>
      <h1>Snake Game</h1>
      <h4>Score: {{score}}</h4>
      <Snake @IncrementScore="IncrementScore" @ClearScore="ClearScore"/>
  </div>
</template>

<script>
import Snake from './Game.vue'

export default {
    name: 'WelcomePage',
    components: {
    Snake
    },
    data() {
    return {
        score: 0,
    };
    },
    methods: {
        IncrementScore: function () {
            this.score++;
        },
        ClearScore: function () {
            this.score = 0;
        }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
