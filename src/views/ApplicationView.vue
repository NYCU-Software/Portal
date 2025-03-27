<template>
  <div>
    <table class="w-full text-xl text-center border">
      <thead class="text-2xl border">
        <tr>
          <th scope="col">Client ID</th>
          <th scope="col">Client Name</th>
          <th scope="col">Redirect URIs</th>
          <th scope="col">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="client in clients" :key="client.client_id">
          <td>{{ client.client_id }}</td>
          <td>{{ client.client_name }}</td>
          <td>{{ client.redirect_uris.join(", ") }}</td>
          <td>
            <button @click="editClient(client)" class="mx-2">修改</button>
            <button @click="deleteClient(client.client_id)" class="mx-2">
              刪除
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    <br />
    <hr />
    <br />
    <form @submit.prevent="isEditing ? updateClient() : addClient()">
      <div>
        <label>Client ID:</label>
        <br />
        <input
          v-model="form.client_id"
          class="w-full p-2.5 text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 mt-5"
          :disabled="isEditing"
          required
        />
      </div>
      <div>
        <label>Client Name:</label>
        <br />
        <input
          v-model="form.client_name"
          class="w-full p-2.5 text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 mt-5"
          required
        />
      </div>
      <div>
        <label>Redirect URIs (逗號分隔，必須為有效 URL):</label>
        <br />
        <input
          v-model="form.redirect_uris"
          class="w-full p-2.5 text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 mt-5"
          required
        />
      </div>
      <div>
        <label>Client Secret:</label>
        <br />
        <input
          v-model="form.client_secret"
          class="w-full p-2.5 text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 mt-5"
          required
        />
      </div>
      <br />
      <button
        type="submit"
        class="button hover:bg-white hover:text-gray-900 float-right mt-2 font-semibold py-2 px-4 border rounded"
      >
        {{ isEditing ? "更新" : "新增" }}
      </button>
      <button
        type="button"
        v-if="isEditing"
        @click="cancelEdit"
        class="float-right mt-2 font-semibold py-2 px-4 border rounded mr-2"
      >
        取消
      </button>
      <div style="clear: both"></div>
    </form>
  </div>
</template>

<script>
import axios from "axios";

const baseAPI = import.meta.env.VITE_BASE_API;

export default {
  data() {
    return {
      clients: [],
      isEditing: false,
      form: {
        client_id: "",
        client_name: "",
        redirect_uris: "",
        client_secret: "",
      },
    };
  },
  mounted() {
    this.fetchClients();
  },
  methods: {
    async fetchClients() {
      try {
        const res = await axios.get(`${baseAPI}/clients/`, {
          headers: { "Content-Type": "application/json" },
        });
        this.clients = res.data;
      } catch (error) {
        console.error("取得 Clients 失敗:", error);
        alert("無法取得 Clients 資料");
      }
    },
    async addClient() {
      const data = {
        client_id: this.form.client_id.trim(),
        client_name: this.form.client_name.trim(),
        redirect_uris: this.form.redirect_uris.split(",").map((s) => s.trim()),
        client_secret: this.form.client_secret.trim(),
      };

      // 檢查 redirect_uris 是否為合法 URL
      for (const uri of data.redirect_uris) {
        try {
          new URL(uri);
        } catch (e) {
          alert(`Redirect URI ${uri} 格式錯誤`);
          return;
        }
      }

      try {
        await axios.post(`${baseAPI}/clients`, data, {
          headers: { "Content-Type": "application/json" },
        });
        this.fetchClients();
        this.resetForm();
      } catch (error) {
        console.error("新增失敗:", error);
        alert(error.response?.data?.error || "新增 Client 失敗");
      }
    },
    async updateClient() {
      const data = {
        client_id: this.form.client_id.trim(),
        client_name: this.form.client_name.trim(),
        redirect_uris: this.form.redirect_uris.split(",").map((s) => s.trim()),
        client_secret: this.form.client_secret.trim(),
      };

      // 檢查 redirect_uris 格式
      for (const uri of data.redirect_uris) {
        try {
          new URL(uri);
        } catch (e) {
          alert(`Redirect URI ${uri} 格式錯誤`);
          return;
        }
      }

      try {
        await axios.put(`${baseAPI}/clients/${this.form.client_id}`, data, {
          headers: { "Content-Type": "application/json" },
        });
        this.fetchClients();
        this.resetForm();
        this.isEditing = false;
      } catch (error) {
        console.error("更新失敗:", error);
        alert(error.response?.data?.error || "更新 Client 失敗");
      }
    },
    async deleteClient(clientId) {
      if (!confirm(`確定要刪除 Client ${clientId} 嗎？`)) return;
      try {
        await axios.delete(`${baseAPI}/clients/${clientId}`, {
          headers: { "Content-Type": "application/json" },
        });
        this.fetchClients();
      } catch (error) {
        console.error("刪除失敗:", error);
        alert(error.response?.data?.error || "刪除 Client 失敗");
      }
    },
    editClient(client) {
      this.isEditing = true;
      this.form = {
        client_id: client.client_id,
        client_name: client.client_name,
        redirect_uris: client.redirect_uris.join(", "),
        client_secret: client.client_secret || "",
      };
    },
    cancelEdit() {
      this.isEditing = false;
      this.resetForm();
    },
    resetForm() {
      this.form = {
        client_id: "",
        client_name: "",
        redirect_uris: "",
        client_secret: "",
      };
    },
  },
};
</script>
