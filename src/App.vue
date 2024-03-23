<script setup>
import { RouterView } from "vue-router";
import Navigation from "./components/Navigation.vue";
</script>

<template>
  <header>
    <nav>
      <Navigation />
    </nav>
  </header>

  <RouterView />
</template>
