<script setup>
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <nav>
    <div class="container">
      <div class="top_menu">
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </div>
    </div>
  </nav>

  <RouterView />
</template>
<style lang="scss">
nav {
  position: sticky;
  top: 0px;
  z-index: 100;
  width: 100%;
  height: 28px;
  border-bottom: 1px solid var(--white500);
  background-color: var(--black300);
  .top_menu {
    width: 100%;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: flex-end;
    gap: 10px;
    padding: 2px 0px;
    color: var(--white300);
    @media (max-width: 1400px) {
      padding: 2px 20px;
    }
  }
}
</style>
