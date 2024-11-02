<template>
  <div id="app">
    <DonationForm />
  </div>
</template>

<script>
import DonationForm from './components/DonationForm.vue';

export default {
  name: 'App',
  components: {
    DonationForm
  }
};
</script>

<style>
#app {
  font-family: 'VTB Group UI', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #f2f2f2;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
