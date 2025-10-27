<template>
  <div class="about">
    <div class="content">
      <h1>Made by Karol Adamus</h1>
      <br>
      <h1>project is open and can be found here:
        <a href="https://github.com/karoltg/freatboard" target="_blank" rel="noopener">github</a>
      </h1>
      <div>
        <br>
        <h3>
          Made using
          <a href="https://vite.dev/" target="_blank" rel="noopener">Vite</a> +
          <a href="https://vuejs.org/" target="_blank" rel="noopener">Vue 3</a>.
        </h3>
      </div>
    </div>
  </div>
</template>

<style>
.about {
  min-height: 100vh; /* pełna wysokość ekranu */
  display: flex; /* flexbox */
  align-items: center; /* pionowo */
  justify-content: center; /* poziomo */
  text-align: center; /* wyrównanie tekstu */

}
.content {
  inline-size: fit-content;
  width: 100%;
}
</style>
