<template>
  <div class="home style-dark">
    <h1>Découvrez des artistes à travers le monde</h1>
    <router-link to="artists-selection" class="btn"
      >Démarrer l'expérience</router-link
    >
  </div>
</template>

<script>
// @ is an alias to /src

export default {
  name: "Home",
};
</script>

<style>
.home {
  height: 100vh;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.home h1 {
  font-size: var(--font-size-50);
}
</style>
