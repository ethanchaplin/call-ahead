<script setup>
import MainMap from './components/MainMap.vue'
import { ref } from 'vue'

const siteName = ref('')
const website = ref('')
function displayInfo(data) {
  siteName.value = data['name']
  website.value = data['website']
}
</script>

<template>
  <div>
    <MainMap @info="displayInfo" />
    <transition>
      <div v-if="siteName" class="ontop">
        <div class="site-contain">
          {{ siteName }}
        </div>
      </div>
    </transition>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  overflow: hidden !important;
}

.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.ontop {
  height: 80%;
  width: 22%;
  background-color: white;
  border-radius: 15px;
  box-shadow: -2px 10px 20px -6px rgba(0, 0, 0, 0.25);
  position: absolute;
  z-index: 1;
  transform: translate(10vh, -110%);
}

.site-contain {
  height: 100%;
  width: 100%;
  display: flex;
  justify-content: center;
  padding-top: 10%;
  font-weight: 1000;
  font-size: 2rem;
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
