<template>
  <div id="app">
    <div v-if="isLoggedIn">
      <navbar></navbar>
      <router-view></router-view>
      <p class="has-text-centered">
        <small @click="licenseModalActive = true"><a>Licenses</a></small>
      </p>
      <br>
    </div>
    <section class="hero is-info is-fullheight" v-else>
      <div class="hero-body">
        <div class="container has-text-centered">
          <a
            class="button is-large is-primary"
            :href="baseApiURL + '/oauth_flow_start'"
            :class="{ 'is-loading': buttonLoading }"
            @click="buttonLoading = true">Log in with Ripple</a><br>
          <small @click="licenseModalActive = true"><a>Licenses</a></small>
        </div>
      </div>
    </section>
    <snackbar></snackbar>
    <license-modal v-model="licenseModalActive"></license-modal>
  </div>
</template>

<script>
import router from "./router"
import store from "./store"

import Navbar from "./components/Navbar"
import LicenseModal from "./components/LicenseModal"
import Snackbar from "./components/Snackbar"

import backend from "./backend"
import queryString from "query-string"
import storage from "./storageHandler"
import firebase from "./firebase"

var App = {
  name: "app",
  components: {
    Navbar,
    LicenseModal,
    Snackbar,
  },
  data() {
    return {
      buttonLoading: false,
      baseApiURL: process.env.API_BASE_URL,
      licenseModalActive: false,
    }
  },
  computed: {
    isLoggedIn() {
      return storage.sessionToken() !== null
    }
  },
  mounted() {
    if (window.location.pathname === "/store_tokens") {
      storeTokens()
      window.location.href = "/"
    }
    if (App.computed.isLoggedIn()) {
      firebase.requestPermission()
      backend.getUser("self", user => {
        store.commit("setUser", user)
      })
    }
  }
}

export default App

function storeTokens() {
  const p = queryString.parse(location.search)
  if (isUndefined(p) || isUndefined(p.access) || isUndefined(p.session))
    return
  storage.storeTokens(p.session, p.access)
}

function isUndefined(x) {
  return typeof x === "undefined"
}
</script>

<style src="./assets/bulma.sass" lang="sass"></style>
