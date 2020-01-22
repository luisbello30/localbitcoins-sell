<template>
  <div>
    <button id="toTopBtn" ref="toTopBtn" class="btn-primary" @click="goToTop">
      ^
    </button>
  </div>
</template>
<script>
export default {
  mounted() {
    window.onscroll = () => {
      let currentScroll =
        document.documentElement.scrollTop || document.body.scrollTop;
      if (currentScroll > 300) {
        this.$refs.toTopBtn.style.display = "block";
      } else {
        this.$refs.toTopBtn.style.display = "none";
      }
    };
  },
  methods: {
    goToTop() {
      document.body.scrollTop = 0; // For Safari
      document.documentElement.scrollTop = 0; // For Chrome, Firefox, IE and Opera
    }
  }
};
</script>
<style>
#toTopBtn {
  display: none; /* Hidden by default */
  position: fixed; /* Fixed/sticky position */
  bottom: 20px; /* Place the button at the bottom of the page */
  right: 30px; /* Place the button 30px from the right */
  z-index: 99; /* Make sure it does not overlap */
  border: none; /* Remove borders */
  color: white; /* Text color */
  cursor: pointer; /* Add a mouse pointer on hover */
  padding: 15px; /* Some padding */
  border-radius: 25px; /* Rounded corners */
  font-size: 15px; /* Increase font size */
}
#toTopBtn:hover {
  background-color: navy; /* Set a background color */
}
</style>
