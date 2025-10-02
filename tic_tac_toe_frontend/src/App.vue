<script setup lang="ts">
import GameBoard from '@/components/tictactoe/GameBoard.vue'
</script>

<template>
  <div class="page-wrap">
    <main class="container">
      <section class="header">
        <h1 class="title">Tic Tac Toe</h1>
        <p class="subtitle">Two-player game • Ocean Professional</p>
      </section>

      <GameBoard />
    </main>

    <footer class="footer">
      <p>Built with Vue 3 + Vite</p>
    </footer>
  </div>
</template>

<style scoped>
.page-wrap {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background:
    radial-gradient(1200px 600px at 50% -100px, rgba(37,99,235,0.08), transparent 60%),
    linear-gradient(180deg, rgba(37,99,235,0.05), rgba(249,250,251,1));
}

.container {
  flex: 1;
  max-width: 980px;
  margin: 0 auto;
  padding: 2rem 1rem 3rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.header {
  text-align: center;
  margin-bottom: 1.5rem;
}

.title {
  font-size: 2.2rem;
  line-height: 1.1;
  color: #111827;
  letter-spacing: -0.02em;
}

.subtitle {
  color: #6b7280;
  margin-top: 0.35rem;
}

.footer {
  text-align: center;
  color: #6b7280;
  padding: 1rem 0 2rem;
  font-size: 0.9rem;
}
</style>
