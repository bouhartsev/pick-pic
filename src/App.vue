<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header>
    <div>
      <img alt="Vue logo" class="logo" src="@/assets/logo.svg" width="64" height="64" />

      <div class="wrapper">
        <h1>pick pic</h1>
        <div>by <a href="//bouhartsev.top">bouhartsev</a></div>
      </div>
    </div>
    <nav>
      <RouterLink to="/">Home</RouterLink>
      <RouterLink to="/about">About</RouterLink>
    </nav>
  </header>

  <RouterView />
</template>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;

  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1em;
  padding: 2em;
}

/* .logo {
  display: block;
  margin: 0 auto 2rem;
} */

/* nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
} */

header>div {
  display: flex;
  gap: 1em;
}

header .wrapper {
  display: flex;
  flex-direction: column;
}
</style>
