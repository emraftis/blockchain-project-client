<template>
  <main-page></main-page>
</template>

<script>
import MainPage from './components/MainPage.vue'

export default {
  name: 'App',
  components: {
    MainPage
  }
}
</script>

<style>
#app {
  font-family: 'News Cycle', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 5rem 20%;
}
html {
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
  background-repeat:no-repeat;
  background: -webkit-linear-gradient( 135deg, #72EDF2 10%, #5151E5 100%);
  background: -moz-linear-gradient( 135deg, #72EDF2 10%, #5151E5 100%);
  background: -ms-linear-gradient( 135deg, #72EDF2 10%, #5151E5 100%);
  background: -o-linear-gradient( 135deg, #72EDF2 10%, #5151E5 100%);
  background: linear-gradient( 135deg, #72EDF2 10%, #5151E5 100%);
  min-height: 100vh;
  transition: background-size 3s ease-in-out;
}

@media screen and (max-width: 700px) {
  #app {
    margin: 1.5rem 1%;
  }
}
</style>
