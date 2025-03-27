<template>
  <div
    class="flex flex-col items-center justify-center px-6 py-8 mx-auto md:h-screen lg:py-0"
  >
    <div class="w-full md:mt-0 sm:max-w-md xl:p-0">
      <div class="p-6 space-y-4 md:space-y-6 sm:p-8">
        <h1 class="text-xl font-bold leading-tight tracking-tight md:text-2xl">
          Register a new account
        </h1>
        <form class="space-y-4 md:space-y-6" @submit.prevent="submit">
          <div>
            <label for="username" class="block mb-2 text-sm font-medium"
              >Username (Letters, numerics, and underlines)</label
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
            <label for="email" class="block mb-2 text-sm font-medium"
              >Email</label
            >
            <input
              type="text"
              name="email"
              id="email"
              class="border rounded-lg block w-full p-2.5 text-gray-900"
              placeholder="email"
              v-model="email"
              required
            />
          </div>
          <div>
            <label for="password" class="block mb-2 text-sm font-medium"
              >Password (At least 8 digits)</label
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
          </div>
          <div>
            <label for="password" class="block mb-2 text-sm font-medium"
              >Password (Repeated)</label
            >
            <input
              type="password"
              name="password2"
              id="password2"
              placeholder="••••••••"
              class="border rounded-lg block w-full p-2.5 text-gray-900"
              v-model="password2"
              required
            />
            <p>{{ message }}</p>
          </div>
          <button
            type="submit"
            class="hover:bg-white hover:text-gray-900 float-right mt-2 font-semibold py-2 px-4 border rounded"
          >
            Register
          </button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
const baseAPI = import.meta.env.VITE_KRATOS_URL;

export default {
  data() {
    return {
      username: "",
      password: "",
      password2: "",
      message: "",
      email: "",
    };
  },
  methods: {
    async submit() {
      if (this.password != this.password2) {
        this.message = "The password does not match.";
        return;
      }
      this.message = "";

      try {
        const createFlowURL = new URL(
          "/self-service/registration/browser",
          baseAPI,
        ).toString();
        const createFlowRes = await axios.get(createFlowURL, {
          withCredentials: true,
        });
        const flowData = createFlowRes.data;
        const flowId = flowData.id;

        let csrfToken = "";
        const nodes = flowData.ui?.nodes || [];
        for (const node of nodes) {
          if (node.attributes?.name === "csrf_token") {
            csrfToken = node.attributes.value;
            break;
          }
        }
        const updateFlowURL = new URL(
          `/self-service/registration?flow=${flowId}`,
          baseAPI,
        );
        const updateRes = await axios.post(
          updateFlowURL,
          {
            method: "password",
            password: this.password,
            traits: {
              email: this.email,
              username: this.username,
            },
            csrf_token: csrfToken,
          },
          {
            withCredentials: true,
          },
        );

        console.log("Registration success:", updateRes.data);
        window.location.href = "/login";
      } catch (error) {
        console.error(error);
        const errMsg =
          error.response?.data?.error?.reason ||
          error.response?.data?.ui?.messages?.[0]?.text ||
          "Registration failed.";
        this.message = errMsg;
      }
    },
  },
};
</script>
