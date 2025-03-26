<script setup>
import { RouterLink, RouterView } from "vue-router";
</script>

<template>
  <div class="max-sm:hidden">
    <div
      class="dark sidebar fixed w-1/3 lg:w-1/5 float-left min-h-[calc(100dvh)]"
    >
      <MenuComponent
        :isLogged="isLogged"
        :username="username"
        :isAdmin="isAdmin"
      />
    </div>
    <div class="light w-2/3 lg:w-4/5 float-right min-h-[calc(100dvh)]">
      <RouterView />
    </div>
  </div>
  <div class="sm:hidden">
    <div class="dark fixed w-full min-h-[calc(100dvh)]" v-if="!show">
      <MenuComponent
        :isLogged="isLogged"
        :username="username"
        :isAdmin="isAdmin"
      />
      <div @click="turn_page">
        <div class="triangle light-tri1"></div>
        <div class="triangle dark-tri2"></div>
        <div class="triangle light-tri3"></div>
      </div>
    </div>
    <div class="light min-h-[calc(100dvh)] min-w-[calc(100dvw)]" v-if="show">
      <RouterView />
      <div class="h-[100px]"></div>
      <div @click="turn_page">
        <div class="triangle dark-tri1"></div>
        <div class="triangle light-tri2"></div>
        <div class="triangle dark-tri3"></div>
      </div>
    </div>
  </div>
</template>

<script>

import axios from "axios"
import Cookies from "js-cookie";
import MenuComponent from "./components/MenuComponent.vue";

const baseAPI = import.meta.env.VITE_KRATOS_URL;

export default {
  components: {
    MenuComponent,
  },
  data() {
    return {
      show: 1,
      isLogged: 0,
      username: "Anonymous",
      isAdmin: 0,
    };
  },
  mounted() {
    if (window.location.pathname === "/") this.show = 0;
    this.checkLogged();
  },
  methods: {
    turn_page() {
      this.show = (this.show + 1) % 2;
    },
    async checkLogged() {
      axios
        .get(`${baseAPI}/sessions/whoami`, { withCredentials: true })
        .then((res) => {
          const data = res.data
          this.isLogged = 1
          this.username = data.identity?.traits?.username || "Anonymous"
          this.isAdmin = data.identity?.traits?.is_admin ? 1 : 0
        })
        .catch(() => {
          this.isLogged = 0
          this.isAdmin = 0
          this.username = "Anonymous"
        })
    },
  },
};
</script>
