<template>
  <p class="text-3xl"><a href="/">NYCU-ME Portal</a></p>
  <br />
  <div v-if="isLogged">
    <p class="text-2xl">{{ username }}</p>
    <ul class="max-w-md space-y-1 list-disc list-inside">
      <li><a href="/profile">Profile</a></li>
      <li><a href="#" @click="logout">Logout</a></li>
    </ul>
  </div>
  <div v-if="!isLogged">
    <p class="text-2xl">{{ username }}</p>
    <ul class="max-w-md space-y-1 list-disc list-inside">
      <li><a href="/login">Login</a></li>
    </ul>
  </div>
  <br />
  <div>
    <p class="text-2xl"><a href="/services">Services</a></p>
    <ul class="max-w-md space-y-1 list-disc list-inside">
    </ul>
  </div>
  <br />
  <div v-if="isAdmin">
    <br />
    <p class="text-2xl">Administrator</p>
    <ul class="max-w-md space-y-1 list-disc list-inside">
      <li><a href="/admin/clients">應用程式管理</a></li>
      <li><a href="/admin/users">用戶管理</a></li>
    </ul>
  </div>
</template>

<script>
import axios from "axios";
import Cookies from "js-cookie";
const baseAPI = import.meta.env.VITE_KRATOS_URL;

export default {
  props: {
    username: { type: String, default: "Anonymous" },
    isAdmin: { type: Number, default: 0 },
    isLogged: { type: Number, default: 0 },
  },
  data() {
    return {};
  },
  mounted() {},
  methods: {
    async logout() {
      try {
        const logoutURL = new URL("/self-service/logout/browser", baseAPI)
        const r = await axios.get(logoutURL.toString(), { withCredentials: true })
		console.log(r.data.logout_token)
		if (r.data.logout_token) {
		
          window.location.href = new URL(`/self-service/logout?token=${r.data.logout_token}`, baseAPI).toString()
        }
      } catch (e) {
        console.error("Logout error:", e)
      }
    },
  },
};
</script>
