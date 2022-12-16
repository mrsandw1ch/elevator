<template>
  <Heading msg="Эмулятор лифта"/>
  <Elevator/>
</template>

<script>
import Heading from './components/Heading.vue'
import Elevator from './components/Elevator/Elevator.vue'

export default {
  name: 'App',
  components: {
    Heading,
    Elevator
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
  margin-top: 60px;
}

* {
  box-sizing: border-box;
}
</style>
