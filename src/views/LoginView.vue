<template>
  <div
    class="flex flex-col items-center justify-center px-6 py-8 mx-auto md:h-screen lg:py-0"
  >
    <div class="w-full md:mt-0 sm:max-w-md xl:p-0">
      <div class="p-6 space-y-4 md:space-y-6 sm:p-8">
        <h1 class="text-xl font-bold leading-tight tracking-tight md:text-2xl">
          Sign in to your account
        </h1>
        <form class="space-y-4 md:space-y-6" @submit.prevent="submit">
          <div>
            <label for="username" class="block mb-2 text-sm font-medium"
              >Username</label
            >
            <input
              type="text"
              name="username"
              id="username"
              class="border rounded-lg block w-full p-2.5 text-gray-900"
              placeholder="username"
              v-model="username"
              required
            />
          </div>
          <div>
            <label for="password" class="block mb-2 text-sm font-medium"
              >Password</label
            >
            <input
              type="password"
              name="password"
              id="password"
              placeholder="••••••••"
              class="border rounded-lg block w-full p-2.5 text-gray-900"
              v-model="password"
              required
            />
            <p>{{ message }}</p>
          </div>
          <a
            href="/register"
            class="hover:text-gray-200 float-left font-semibold py-2 px-4 border rounded"
          >
            Register
          </a>
          <button
            type="submit"
            class="hover:bg-white hover:text-gray-900 float-right mt-2 font-semibold py-2 px-4 border rounded"
          >
            Sign In
          </button>
        </form>
      </div>
    </div>
  </div>
</template>


<script>
import axios from "axios"

export default {
  data() {
    return {
      username: "",
      password: "",
      message: ""
    }
  },
    mounted(){
        const params = new URLSearchParams(window.location.search)
        let flowId = params.get("flow")

        if (!flowId) {
            window.location.href = `${import.meta.env.VITE_KRATOS_URL}/self-service/login/browser?refresh=true&return_to=${encodeURIComponent(window.location.href)}`
          return
        }
    },
  methods: {
    async submit() {
      try {
        const params = new URLSearchParams(window.location.search)
        let flowId = params.get("flow")
        const flowRes = await axios.get(`${import.meta.env.VITE_KRATOS_URL}/self-service/login/flows?id=${flowId}`, { withCredentials: true })
        const flowData = flowRes.data
        const csrfNode = flowData.ui.nodes.find(n => n.attributes.name === "csrf_token")
        const csrfToken = csrfNode ? csrfNode.attributes.value : ""
        await axios.post(`${import.meta.env.VITE_KRATOS_URL}/self-service/login?flow=${flowId}`, {
          method: "password",
          csrf_token: csrfToken,
          identifier: this.username,
          password: this.password
        }, { withCredentials: true })
        window.location.href = '/'
      } catch (error) {
        console.log(JSON.parse(error.response.request.response))
        this.message = JSON.parse(error.response.request.response).ui?.messages[0]?.text || JSON.parse(error.response.request.response).error.reason
      }
    }
  }
}
</script>
