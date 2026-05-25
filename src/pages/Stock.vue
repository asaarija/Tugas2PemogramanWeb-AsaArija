<script setup>
import { ref, computed, onMounted } from "vue";
import Swal from "sweetalert2";
import Header from "./components/Header.vue";
import InputNewBhnAjar from "./components/InputNewBhnAjar.vue";
import { apps } from "../containts/dataBahanAjar.js";

const emit = defineEmits(["logout", "navigate"]);
const showAddModal = ref(false);
const selectedStockFilter = ref("all");
const selectedUPBJJ = ref("");
const selectedKategori = ref("");
const searchQuery = ref("");
const stokData = ref([]);

const handleLogout = () => {
  emit("logout");
};

const handleNavigate = (page) => {
  emit("navigate", page);
};

const loadStockFromLocalStorage = () => {
  // Cek localStorage terlebih dahulu
  const stored = localStorage.getItem("stok");
  if (stored) {
    try {
      const parsed = JSON.parse(stored);
      if (Array.isArray(parsed) && parsed.length > 0) {
        stokData.value = parsed;
        return;
      }
    } catch (error) {
      console.error("Error parsing localStorage stok:", error);
    }
  }

  // Fallback ke apps.stok dari dataBahanAjar.js
  if (Array.isArray(apps.stok) && apps.stok.length > 0) {
    stokData.value = [...apps.stok];
  } else {
    stokData.value = [];
  }
};

onMounted(() => {
  loadStockFromLocalStorage();
});

const filteredStocks = computed(() => {
  return stokData.value.filter((item) => {
    const statusMatch =
      selectedStockFilter.value === "all" ||
      (selectedStockFilter.value === "stokKosong" && item.qty === 0) ||
      (selectedStockFilter.value === "stokSafety" && item.qty <= item.safety);

    const upbjjMatch =
      !selectedUPBJJ.value || item.upbjj === selectedUPBJJ.value;

    const kategoriMatch =
      !selectedKategori.value || item.kategori === selectedKategori.value;

    const query = searchQuery.value.trim().toLowerCase();
    const searchMatch =
      !query ||
      item.kode.toLowerCase().includes(query) ||
      item.judul.toLowerCase().includes(query) ||
      item.kategori.toLowerCase().includes(query) ||
      item.upbjj.toLowerCase().includes(query) ||
      item.lokasiRak.toLowerCase().includes(query);

    return statusMatch && upbjjMatch && kategoriMatch && searchMatch;
  });
});

const saveStockToLocalStorage = () => {
  localStorage.setItem("stok", JSON.stringify(stokData.value));
  apps.stok = stokData.value;
};

const handleHapusStock = (index) => {
  const item = filteredStocks.value[index];
  if (!item) return;

  Swal.fire({
    title: "Apakah Anda yakin?",
    text: "Data stok akan dihapus secara permanen.",
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#d33",
    cancelButtonColor: "#3085d6",
    confirmButtonText: "Ya, hapus!",
    cancelButtonText: "Batal",
  }).then((result) => {
    if (result.isConfirmed) {
      stokData.value = stokData.value.filter(
        (stokItem) => stokItem.kode !== item.kode,
      );
      saveStockToLocalStorage();

      Swal.fire({
        icon: "success",
        title: "Dihapus!",
        text: "Data stok berhasil dihapus.",
      });
    }
  });
};

const handleAddStock = () => {
  showAddModal.value = true;
};

const handleSaveSuccess = (newItem) => {
  stokData.value = [...stokData.value, newItem];
  saveStockToLocalStorage();
  showAddModal.value = false;

  Swal.fire({
    icon: "success",
    title: "Berhasil!",
    text: "Stok bahan ajar berhasil ditambahkan.",
    confirmButtonColor: "#3085d6",
  });
};
</script>
<template>
  <Header
    :activePage="'Stock'"
    @logout="handleLogout"
    @navigate="handleNavigate"
  />
  <!-- Filter starts -->
  <section class="w-full mt-20 px-10 py-5 space-y-6">
    <div class="flex flex-col md:flex-row md:items-center gap-2">
      <select v-model="selectedStockFilter" class="rounded-lg border px-3 py-2">
        <option value="all">Semua Stok</option>
        <option value="stokKosong">Stok Kosong</option>
        <option value="stokSafety">Stok Safety</option>
      </select>

      <select v-model="selectedUPBJJ" class="rounded-lg border px-3 py-2">
        <option value="">Semua UPBJJ</option>
        <option v-for="upbjj in apps.upbjjList" :key="upbjj" :value="upbjj">
          {{ upbjj }}
        </option>
      </select>

      <select v-model="selectedKategori" class="rounded-lg border px-3 py-2">
        <option value="">Semua Kategori</option>
        <option
          v-for="kategori in apps.kategoriList"
          :key="kategori"
          :value="kategori"
        >
          {{ kategori }}
        </option>
      </select>

      <!-- Filters end -->
      <!-- Tambahkan Stock Button starts -->
      <button
        class="bg-blue-500 text-white px-4 py-2 rounded-md hover:bg-blue-600 transition-colors font-semibold cursor-pointer flex items-center gap-2"
        @click="handleAddStock"
      >
        <i class="fa-solid fa-plus">+</i>
        <h1>Tambah Stock</h1>
      </button>
    </div>
    <!-- Tambahkan Stock Button ends -->

    <!-- data bahan ajar start -->
    <div class="shadow-lg p-6 rounded-lg bg-white">
      <h2 class="text-2xl font-bold mb-4">Rekap Stok Bahan Ajar</h2>
      <div class="overflow-x-auto">
        <table class="min-w-full border border-gray-200 text-sm">
          <thead class="bg-gray-100 text-left">
            <tr>
              <th class="p-3 border-b">Kode Lokasi</th>
              <th class="p-3 border-b">Judul</th>
              <th class="p-3 border-b">Kategori</th>
              <th class="p-3 border-b">UPBJJ</th>
              <th class="p-3 border-b">Lokasi Rak</th>
              <th class="p-3 border-b">Harga</th>
              <th class="p-3 border-b">Stok</th>
              <th class="p-3 border-b">Safety Stock</th>
              <th class="p-3 border-b">Catatan HTML</th>
              <th class="p-3 border-b">Aksi</th>
            </tr>
          </thead>
          <tbody id="stockTableBody">
            <tr
              v-for="(item, index) in filteredStocks"
              :key="item.kode + index"
              class="border-b hover:bg-gray-50"
            >
              <td class="p-3">{{ item.kode }}</td>
              <td class="p-3">{{ item.judul }}</td>
              <td class="p-3">{{ item.kategori }}</td>
              <td class="p-3">{{ item.upbjj }}</td>
              <td class="p-3">{{ item.lokasiRak }}</td>
              <td class="p-3">{{ item.harga }}</td>
              <td class="p-3">{{ item.qty }}</td>
              <td class="p-3">{{ item.safety }}</td>
              <td class="p-3" v-html="item.catatanHTML"></td>
              <td class="p-3">
                <button
                  class="bg-blue-500 text-white px-4 py-2 rounded-md hover:bg-blue-600 transition-colors font-semibold cursor-pointer"
                >
                  Edit Stok
                </button>
                <button
                  class="bg-red-500 text-white px-4 py-2 rounded-md hover:bg-red-600 transition-colors font-semibold cursor-pointer ml-2"
                  @click="handleHapusStock(index)"
                >
                  Hapus Stock
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <!-- data bahan ajar end -->

    <div
      v-if="showAddModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
      @click.self="showAddModal = false"
    >
      <div
        class="bg-white rounded-lg shadow-2xl w-full max-w-3xl max-h-[90vh] overflow-y-auto"
      >
        <div
          class="sticky top-0 bg-white border-b border-gray-200 p-6 flex justify-between items-center"
        >
          <h2 class="text-2xl font-bold text-gray-800">
            Tambah Stock Bahan Ajar
          </h2>
          <button
            @click="showAddModal = false"
            class="text-gray-500 hover:text-gray-700 text-2xl font-bold"
          >
            ×
          </button>
        </div>
        <div class="p-6">
          <InputNewBhnAjar @save-success="handleSaveSuccess" />
        </div>
      </div>
    </div>
  </section>
</template>
