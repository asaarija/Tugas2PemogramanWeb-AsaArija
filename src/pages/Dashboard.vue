<script setup>
import { ref, onMounted } from "vue";
import Header from "./components/Header.vue";

const emit = defineEmits(["logout", "navigate"]);
const greeting = ref("");

onMounted(() => {
  const namaUser = localStorage.getItem("namaUser");
  if (namaUser) {
    greeting.value = `Selamat Datang, ${namaUser}!`;
  } else {
    greeting.value = "Selamat Datang di Dashboard SITTA!";
  }
});

const handleLogout = () => {
  emit("logout");
};

const handleNavigate = (page) => {
  emit("navigate", page);
};
</script>
<template>
  <Header
    :activePage="'Dashboard'"
    @logout="handleLogout"
    @navigate="handleNavigate"
  />
  <div class="mx-10 mt-20 p-5 bg-white rounded-lg shadow-lg m-auto">
    <div class="flex items-center justify-between mb-4">
      <div>
        <h1 class="text-3xl font-bold">Dashboard Aplikasi SITTA</h1>
        <h2
          id="greeting"
          class="text-2xl font-semibold text-blue-900 leading-snug"
        >
          {{ greeting }}
        </h2>
      </div>
    </div>
  </div>
</template>
