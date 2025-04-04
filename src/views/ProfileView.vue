<template>
  <p class="text-white-500 mt-2">{{ message }}</p>
  <div class="center-container qr-container">
    <div v-if="!showTotpForm">
      <button
        @click="showEnableTotpForm"
        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
      >
        Enable TOTP
      </button>
    </div>
    <div v-else>
      <div class="mb-4">
        <p class="text-white-700 mb-2">Scan the QR code with your authenticator app:</p>
        <img v-if="qrCodeUrl" :src="qrCodeUrl" alt="TOTP QR Code" class="mx-auto" />
      </div>
      <div class="flex items-center">
        <input
          type="text"
          v-model="totpCode"
          placeholder="6-digit code"
          class="text-gray-900 border rounded px-2 py-1"
        />
        <button
          @click="enableTotp"
          class="ml-2 bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
        >
          Activate TOTP
        </button>
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
      message: "",
      qrCodeUrl: "",
      totpCode: "",
      flowId: "",
      showTotpForm: false,
    };
  },
  async mounted() {
    const url = new URL(window.location.href);
    const flow = url.searchParams.get("flow");
    if (!flow) {
      const getFlowURL = new URL("/self-service/settings/browser", baseAPI);
      window.location.href = getFlowURL.href;
      return;
    }
    this.flowId = flow;
  },
  methods: {
    async showEnableTotpForm() {
      // Fetch CSRF token and trigger totp enable call with empty totp_code to obtain QR code
      const flow = this.flowId;
      const postFlowURL = new URL("/self-service/settings", baseAPI);
      postFlowURL.searchParams.set("flow", flow);
      const getFlowURL = new URL("/self-service/settings/browser", baseAPI);
      getFlowURL.searchParams.set("flow", flow);
      try {
        const res = await axios.get(getFlowURL.toString(), { withCredentials: true });
        const csrfNode = res.data.ui.nodes.find(
          (n) => n.attributes.name === "csrf_token"
        );
        const csrfToken = csrfNode ? csrfNode.attributes.value : "";
        // Trigger a totp enable request with empty totp_code to get the QR code in the error response
        await axios.post(
          postFlowURL.toString(),
          {
            method: "totp",
            totp_code: "",
            csrf_token: csrfToken,
          },
          { withCredentials: true }
        );
		this.message = "TOTP Enabled."
      } catch (e) {
        this.showTotpForm = true;
        try {
          const data = JSON.parse(e.request.response);
          const totpImage = data.ui.nodes.find(
            (node) => node.type === "img" && node.group === "totp"
          );
          if (totpImage) {
            this.qrCodeUrl = totpImage.attributes.src;
          }
        } catch (err) {
          this.message = "Failed to retrieve QR code";
        }
      }
    },
    async enableTotp() {
      const flow = this.flowId;
      const postFlowURL = new URL("/self-service/settings", baseAPI);
      postFlowURL.searchParams.set("flow", flow);
      const getFlowURL = new URL("/self-service/settings/browser", baseAPI);
      getFlowURL.searchParams.set("flow", flow);
      try {
        const res = await axios.get(getFlowURL.toString(), { withCredentials: true });
        const csrfNode = res.data.ui.nodes.find(
          (n) => n.attributes.name === "csrf_token"
        );
        const csrfToken = csrfNode ? csrfNode.attributes.value : "";
        const r = await axios.post(
          postFlowURL.toString(),
          {
            method: "totp",
            totp_code: this.totpCode,
            csrf_token: csrfToken,
          },
          { withCredentials: true }
        );
        this.message = "TOTP enabled successfully";
        this.showTotpForm = false;
      } catch (e) {
        try {
          const data = JSON.parse(e.request.response);
          this.message = data.error?.reason || data.ui?.messages[0]?.text || "Activation failed";
        } catch (err) {
          this.message = "Activation failed";
        }
      }
    },
  },
};
</script>

<style>
img {
  margin: 2em;
}
</style>
