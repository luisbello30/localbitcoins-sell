<template>
  <div id="app" class="container">
    <div class="px-3 py-3 pt-md-5 pb-md-4 mx-auto text-center">
      <p class="lead">
        Find the best offer to sell your bitcoins for Venezuelan Bolívar (VES)
        in
        <a href="https://localbitcoins.com/" target="_blank">Localbitcoins</a>.
      </p>
    </div>
    <AdList />
  </div>
</template>

<script>
import AdList from "./components/AdList.vue";

export default {
  name: "app",
  components: {
    AdList
  }
};
</script>
