<template>
  <div class="center-container" id="qr-container"></div>
</template>

<script>
import axios from "axios";
import Cookies from "js-cookie";
const baseAPI = import.meta.env.VITE_KRATOS_URL;

export default {
  data() {
    return {
      message: "",
      qrCodeUrl: "",
      totpCode: "",
      flowId: "",
    };
  },
  async mounted() {
    const url = new URL(window.location.href);
    const flow = url.searchParams.get("flow");
    if (!flow) {
      const getFlowURL = new URL("/self-service/settings/browser", baseAPI);
      window.location = getFlowURL.href;
    }
    this.flowId = flow;
    try {
      const postFlowURL = new URL("/self-service/settings", baseAPI);
      postFlowURL.searchParams.set("flow", this.flowId);
      const getFlowURL = new URL("/self-service/settings/browser", baseAPI);
      getFlowURL.searchParams.set("flow", this.flowId);
      const res = await axios.get(getFlowURL.toString(), {
        withCredentials: true,
      });
      const csrfNode = res.data.ui.nodes.find(
        (n) => n.attributes.name === "csrf_token",
      );
      const csrfToken = csrfNode ? csrfNode.attributes.value : "";
      const r = await axios.post(
        postFlowURL.toString(),
        {
          method: "totp",
          totp_code: "",
          csrf_token: csrfToken,
        },
        { withCredentials: true },
      );
    } catch (e) {
      const data = JSON.parse(e.request.response);
      const totpImage = data.ui.nodes.find(
        (node) => node.type === "img" && node.group === "totp",
      );
      const imgElement = document.createElement("img");
      imgElement.src = totpImage.attributes.src;
      document.getElementById("qr-container").appendChild(imgElement);

      console.log(
        JSON.parse(e.request?.response)?.ui.nodes.find(
          (node) =>
            node.group === "totp" && node["attributes"]["name"] === "totp_code",
        ),
      );
    }
  },
};
</script>

<style>
img {
  margin: 2em;
}
</style>
