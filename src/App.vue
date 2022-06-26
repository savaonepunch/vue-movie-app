<script setup></script>

<template>
  <div class="container">
    <RouterView></RouterView>
  </div>
</template>

<style>
div.container {
  max-width: 1920px;
  margin: auto;
  padding: 60px;
}

@media (max-width: 700px) {
  div.container {
    padding: 60px 20px;
  }
}
</style>
