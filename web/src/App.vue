<script setup lang="ts">
import Game from "./components/game.vue";
import Rsvp from "./components/rsvp.vue";
</script>

<template>
  <header class="header">
    <h1>Konishi &amp; Ofstad</h1>
  </header>
  <main class="content">
    <Rsvp />
    <Game />
  </main>
</template>

<style scoped>
.header {
  text-align: center;
  padding: 0.5rem 0 0.25rem;
}

h1 {
  margin: 0;
  padding-bottom: 1rem;
  line-height: 1.05;
  font-family: 'Engagement', cursive;
  /* responsive size: min 2.2rem, scales with viewport, max 3.5rem */
  font-size: clamp(2.2rem, 8vw, 5rem);
  color: #ffe787;
  text-shadow: 2px 2px 2px rgba(0, 0, 0, 0.5);
}

/* game area fills remaining height */
.content {
  flex: 1;                  /* let Game.vue take the rest of the screen */
  display: flex;
}
</style>
