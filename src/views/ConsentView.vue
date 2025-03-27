<template>
  <div
    class="flex flex-col items-center justify-center px-6 py-8 mx-auto md:h-screen lg:py-0"
  >
    <div class="p-6 space-y-4 md:space-y-6 sm:p-8">
      <h1 class="text-2xl font-bold leading-tight tracking-tight md:text-2xl">
        Consent for {{ clientName }}
      </h1>
      <ul>
        <li
          class="max-w-md space-y-1 list-disc list-inside"
          v-for="scope in requestedScopes"
          :key="scope"
        >
          {{ scope }}
        </li>
      </ul>
      <button
        class="hover:bg-white hover:text-gray-900 float-left m-2 font-semibold py-2 px-4 border"
        @click="reject"
      >
        Reject
      </button>
      <button
        class="hover:bg-white hover:text-gray-900 float-right m-2 font-semibold py-2 px-4 border"
        @click="accept"
      >
        Accept
      </button>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Cookies from "js-cookie";
const baseAPI = import.meta.env.VITE_BASE_API;

export default {
  data() {
    return {
      clientName: "",
      requestedScopes: [],
    };
  },
  async mounted() {
    const cc = new URLSearchParams(window.location.search).get(
      "consent_challenge",
    );
    if (!cc) return;
    try {
      const url = new URL("/consent", baseAPI);
      url.searchParams.set("consent_challenge", cc);
      const res = await axios.get(url.href);
      this.clientName = res.data.client_name;
      this.requestedScopes = res.data.requested_scopes || [];
    } catch (e) {
      console.error(e);
    }
  },
  methods: {
    async accept() {
      const token = Cookies.get("token") || "";
      const cc = new URLSearchParams(window.location.search).get(
        "consent_challenge",
      );
      if (!cc) return;
      try {
        const url = new URL("/consent/accept", baseAPI).href;
        const res = await axios.post(
          url,
          { consent_challenge: cc },
          {
            headers: {
              Authorization: `Bearer ${token}`,
            },
          },
        );
        console.log(res.data.url);
        window.location.href = res.data.url;
      } catch (e) {
        console.error(e);
      }
    },
    async reject() {
      const cc = new URLSearchParams(window.location.search).get(
        "consent_challenge",
      );
      if (!cc) return;
      try {
        const url = new URL("/consent/reject", baseAPI).href;
        const token = Cookies.get("token") || "";
        const res = await axios.post(
          url,
          { consent_challenge: cc },
          {
            headers: {
              Authorization: `Bearer ${token}`,
            },
          },
        );
        window.location.href = res.data.url;
      } catch (e) {
        console.error(e);
      }
    },
  },
};
</script>
