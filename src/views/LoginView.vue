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
            <label
              v-if="!showTotp"
              for="username"
              class="block mb-2 text-sm font-medium"
              >Username</label
            >
            <input
              type="text"
              name="username"
              id="username"
              class="border rounded-lg block w-full p-2.5 text-gray-900"
              placeholder="username"
              v-model="username"
              v-if="!showTotp"
            />
          </div>
          <div>
            <label for="loginField" class="block mb-2 text-sm font-medium">
              <span v-if="!showTotp">Password </span>
              <span v-else>TOTP Code </span>
            </label>
            <template v-if="!showTotp">
              <input
                type="password"
                name="password"
                id="loginField"
                placeholder="••••••••"
                class="border rounded-lg block w-full p-2.5 text-gray-900"
                v-model="password"
                required
              />
            </template>
            <template v-else>
              <input
                type="text"
                name="totp"
                id="loginField"
                placeholder="6-digit code"
                class="border rounded-lg block w-full p-2.5 text-gray-900"
                v-model="totpCode"
                required
              />
            </template>
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
import axios from "axios";
const baseAPI = import.meta.env.VITE_KRATOS_URL;

export default {
  data() {
    return {
      username: "",
      password: "",
      totpCode: "",
      message: "",
      showTotp: false,
    };
  },
  async mounted() {
    const url = new URL(window.location.href);
    if (
      !url.searchParams.get("flow") &&
      url.searchParams.get("login_challenge")
    ) {
      const getFlowURL = new URL(
        "/self-service/login/browser?refresh=true",
        baseAPI,
      );
      getFlowURL.searchParams.set(
        "login_challenge",
        url.searchParams.get("login_challenge"),
      );
      window.location = getFlowURL.href;
    } else if (!url.searchParams.get("flow")) {
      const getFlowURL = new URL(
        "/self-service/login/browser?refresh=true",
        baseAPI,
      );
      window.location = getFlowURL.href;
    }
    const flowDetailsURL = new URL("/self-service/login/flows", baseAPI);
    flowDetailsURL.searchParams.set("id", url.searchParams.get("flow"));
    const flowRes = await axios.get(flowDetailsURL.toString(), {
      withCredentials: true,
    });
    const csrfNode = flowRes.data.ui.nodes.find(
      (n) => n.attributes.name === "csrf_token",
    );
    this.csrfToken = csrfNode ? csrfNode.attributes.value : "";
    if (flowRes.data.requested_aal === "aal2") {
      this.showTotp = true;
    }
  },
  methods: {
    toggleTotp() {
      this.showTotp = !this.showTotp;
    },
    async submit() {
      try {
        const url = new URL(window.location.href);
        const flow = url.searchParams.get("flow");
        const postFlowURL = new URL("/self-service/login", baseAPI);
        postFlowURL.searchParams.set("flow", flow);
        let payload = {};
        if (this.showTotp) {
          payload = {
            method: "totp",
            totp_code: this.totpCode,
            csrf_token: this.csrfToken,
          };
        } else {
          payload = {
            method: "password",
            password_identifier: this.username,
            password: this.password,
            csrf_token: this.csrfToken,
          };
        }
        const r = await axios.post(postFlowURL.toString(), payload, {
          withCredentials: true,
        });
        if (r.data.session) {
          window.location.href = "/";
        }
      } catch (e) {
        console.log(e);
        if (e.status == 422) {
          const newURL = new URL(
            "/self-service/login/browser?aal=aal2",
            baseAPI,
          );
          window.location.href = newURL.href;
        }
        this.message =
          JSON.parse(e.request.response)?.error?.reason ||
          JSON.parse(e.response.request.response)?.ui?.messages[0]?.text ||
          "Login error";
      }
    },
  },
};
</script>
