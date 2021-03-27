<template>
  <router-view />
</template>

<style>
:root {
  --accent-color: #52d67f;
  --accent-color-darker: hsl(140, 62%, 48%);
  --background-color: #131313;
  --font-size-50: 5rem;
  --font-size-30: 3rem;
  --font-size-20: 2rem;
  --font-size-15: 1.5rem;
}

html {
  background-color: var(--background-color);
  font-family: "Roboto", sans-serif;
  font-size: 62.5%;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  scroll-behavior: smooth;
}

body {
  margin: 0;
  overflow: initial;
}

h1,
h2,
h3,
h4,
h5,
h6 {
  font-family: "Playfair Display", serif;
}

.style-light {
  color: #000;
}

.style-dark {
  color: #fff;
}

::selection {
  background-color: var(--accent-color);
}

.btn {
  background-color: var(--accent-color);
  padding: 12px 30px;
  border-radius: 10em;
  transition: ease-in-out 0.2s;
  color: #000;
  text-decoration: none;
  display: inline-block;
  font-weight: 600;
  text-transform: uppercase;
  border: none;
  font-size: 1.6rem;
}

.btn:hover {
  background-color: #fff;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
