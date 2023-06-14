<template>
  <div class="title">
    <b-navbar toggleable="lg" type="dark" variant="info">
      <b-navbar-brand class="ml-2">
        <b>{{ appName }}</b>
      </b-navbar-brand>
    </b-navbar>
    <h3></h3>
    <AnaliseFrete />
  </div>
</template>

<script>
import { BNavbar, BNavbarBrand } from 'bootstrap-vue'
import AnaliseFrete from './AnaliseFrete.vue'

export default {
  components: {
    BNavbar,
    BNavbarBrand,
    AnaliseFrete,
  },
  data() {
    return {
      appName: 'Melhor Frete',
    }
  },
}
</script>

<style scoped>
.title .navbar {
  background-color: #00aca6 !important;
}

.title .navbar-brand {
  margin-left: 20px;
}
</style>