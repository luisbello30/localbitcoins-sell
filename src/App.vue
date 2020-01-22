<template>
  <div id="app">
    <NavBar />
    <div class="container">
      <div class="px-3 py-3 pt-md-5 pb-md-4 mx-auto text-center">
        <p class="lead">
          Find the best offer to sell your bitcoins for Venezuelan Bolívar (VES)
          in
          <a href="https://localbitcoins.com/" target="_blank">Localbitcoins</a
          >.
        </p>
      </div>
      <AdsList />
      <p class="text-center">
        Favicon made by
        <a href="https://www.flaticon.com/authors/freepik" title="Freepik"
          >Freepik</a
        >
        from
        <a href="https://www.flaticon.com/" title="Flaticon"
          >www.flaticon.com</a
        >
      </p>
    </div>
    <ScrollToTopBtn />
  </div>
</template>

<script>
import AdsList from "./components/AdsList.vue";
import NavBar from "./components/NavBar.vue";
import ScrollToTopBtn from "./components/ScrollToTopBtn.vue";

export default {
  name: "app",
  components: {
    AdsList,
    NavBar,
    ScrollToTopBtn
  }
};
</script>
