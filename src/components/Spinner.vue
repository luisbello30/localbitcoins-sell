<template>
  <div><div class="lds-dual-ring"></div></div>
</template>
<style>
.lds-dual-ring {
  display: inline-block;
  width: 35px;
  height: 35px;
}
.lds-dual-ring:after {
  content: " ";
  display: block;
  width: 25px;
  height: 25px;
  margin: 8px;
  border-radius: 50%;
  border: 3px solid #dfc;
  border-color: navy transparent navy transparent;
  animation: lds-dual-ring 1.2s linear infinite;
}
@keyframes lds-dual-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
