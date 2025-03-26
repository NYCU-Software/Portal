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
        <li><a href="https://dns.nycu.me">申請三級網域</a></li>
        <li><a href="https://mail.nycu.me">申請臨時 Email</a></li>
        <li><a href="https://nasa.nycu.me">System Judge</a></li>
    </ul>
  </div>
  <br />
  <div v-if="isAdmin">
    <br />
    <p class="text-2xl">Administrator</p>
    <ul class="max-w-md space-y-1 list-disc list-inside">
      <li><a href="/admin/user">User Manager</a></li>
    </ul>
  </div>
</template>

<script>
import axios from "axios"
import Cookies from "js-cookie"

import { Configuration, FrontendApi } from '@ory/kratos-client'

const kratos = new FrontendApi(
  new Configuration({
    basePath: import.meta.env.VITE_KRATOS_URL,
    baseOptions: {
      withCredentials: true,
    },
  })
)

console.log(kratos)
export default {
  props: {
    username: { type: String, default: "Anonymous" },
    isAdmin: { type: Number, default: 0 },
    isLogged: { type: Number, default: 0 },
  },
  data() {
    return {}
  },
  mounted() {
  },
  methods: {

    async logout() {
      try {
		  const { data: flow } = await kratos.createBrowserLogoutFlow()
		  await kratos.updateLogoutFlow({
			token: flow.logout_token,
		  })
		  window.location.reload()
      } catch (error) {
        console.error('Logout error:', error)
      }
    }
  },
}
</script>

