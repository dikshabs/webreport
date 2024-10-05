<template>
  <div
   
  >
   
    <RouterView class="flex-1" >
      
    </RouterView>
  </div>
</template>

<script setup>
import { RouterView } from "vue-router";
</script>

<style>
.page-enter-active {
  transition: 600ms ease all;
}

.page-enter-from {
  opacity: 0;
}
</style>
