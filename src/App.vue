<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <div class="layout">
    <header class="site-header sticky">
    <img alt="Freatboard logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="Freatboard" />
      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/freatboard">Freatboard</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>
    </div>
    </header>

    <main class="content">
      <RouterView />
    </main>
  </div>
</template>

<style scoped>
.layout {
  min-height: 100vh;
  display: grid;
  grid-template-rows: auto 1fr;  /* header, potem reszta */
}

.sticky {
  position: sticky;
  top: 0;
  z-index: 1000;
/*  background: #fff;*/
/*  border-bottom: 1px solid #eee;*/
  /* opcjonalnie stała wysokość, ale nie jest wymagana */
  /* height: 72px; display:flex; align-items:center; */
}

.content {
  min-height: 0;  /* ważne przy grid/flex, żeby 1fr mógł się kurczyć */
  /* padding: 1rem;  // jeżeli chcesz wewn. odstępy */
}

header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
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
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
