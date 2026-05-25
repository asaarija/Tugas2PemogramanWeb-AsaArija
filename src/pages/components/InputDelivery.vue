<script setup>
import { ref, onMounted } from "vue";
import Swal from "sweetalert2";
import { dataBahanAjar } from "../../containts/data.js";
import { apps as bahanAjarApps } from "../../containts/dataBahanAjar.js";

const emit = defineEmits(["save-success"]);

const inputDONew = ref("");
const inputDateNew = ref("");
const inputNIMNew = ref("");
const inputNamaPenerimaNew = ref("");
const inputEkspedisiNew = ref("");
const selectBahanAjarNew = ref("");

const deliveryRecords = ref([]);

onMounted(() => {
  // Load data dari localStorage
  const stored = localStorage.getItem("deliveryRecords");
  if (stored) {
    deliveryRecords.value = JSON.parse(stored);
    bahanAjarApps.deliveryRecords.length = 0;
    bahanAjarApps.deliveryRecords.push(...deliveryRecords.value);
  }
});

const handleSearch = () => {
  if (
    !inputDONew.value.trim() ||
    !inputDateNew.value ||
    !inputNIMNew.value.trim() ||
    !inputNamaPenerimaNew.value.trim() ||
    !inputEkspedisiNew.value ||
    !selectBahanAjarNew.value
  ) {
    Swal.fire({
      title: "Form Belum Lengkap",
      text: "Silakan isi semua field yang diperlukan.",
      icon: "warning",
      confirmButtonColor: "#1e3a8a",
      confirmButtonText: "OK",
    });
    return;
  }

  const newRecord = {
    id: Date.now(),
    nomorDO: inputDONew.value,
    tanggal: inputDateNew.value,
    nim: inputNIMNew.value,
    namaPenerima: inputNamaPenerimaNew.value,
    ekspedisi: inputEkspedisiNew.value,
    bahanAjar: selectBahanAjarNew.value,
    status: "Dalam Proses",
  };

  deliveryRecords.value.push(newRecord);
  bahanAjarApps.deliveryRecords.push(newRecord);

  // Simpan ke localStorage
  localStorage.setItem(
    "deliveryRecords",
    JSON.stringify(deliveryRecords.value),
  );

  // Update stok di Stock - kurangi qty berdasarkan bahanAjar yang dikirim
  const storedStok = localStorage.getItem("stok");
  if (storedStok) {
    try {
      const stokList = JSON.parse(storedStok);
      const stokIndex = stokList.findIndex(
        (item) => item.kode === selectBahanAjarNew.value,
      );
      if (stokIndex !== -1) {
        // Kurangi stok sebanyak 1 unit (atau bisa disesuaikan)
        stokList[stokIndex].qty = Math.max(0, stokList[stokIndex].qty - 1);
        localStorage.setItem("stok", JSON.stringify(stokList));
      }
    } catch (error) {
      console.error("Error updating stock:", error);
    }
  }

  Swal.fire({
    title: "Berhasil!",
    text: "Data pengiriman telah ditambahkan dan stok telah diperbarui.",
    icon: "success",
    confirmButtonColor: "#1e3a8a",
    confirmButtonText: "OK",
  }).then(() => {
    emit("save-success", newRecord);
  });

  // Reset form
  inputDONew.value = "";
  inputDateNew.value = "";
  inputNIMNew.value = "";
  inputNamaPenerimaNew.value = "";
  inputEkspedisiNew.value = "";
  selectBahanAjarNew.value = "";
};

const handleDelete = (id) => {
  Swal.fire({
    title: "Hapus Data?",
    text: "Anda yakin ingin menghapus data pengiriman ini?",
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#dc2626",
    cancelButtonColor: "#6b7280",
    confirmButtonText: "Ya, Hapus",
    cancelButtonText: "Batal",
  }).then((result) => {
    if (result.isConfirmed) {
      deliveryRecords.value = deliveryRecords.value.filter(
        (record) => record.id !== id,
      );
      Swal.fire({
        title: "Terhapus!",
        text: "Data pengiriman telah dihapus.",
        icon: "success",
        confirmButtonColor: "#1e3a8a",
      });
    }
  });
};
</script>

<template>
  <section class="mx-10 mt-20 p-5 bg-white rounded-lg shadow-lg m-auto">
    <h1 class="text-2xl font-bold mb-6">Input Pengiriman Bahan Ajar</h1>

    <!-- Form Input Start -->
    <div class="mb-8 p-6 bg-gray-50 rounded-lg border border-gray-200">
      <label
        for="nomorDO"
        class="block text-sm font-semibold text-gray-700 mb-4"
        >Form Pengiriman Baru:</label
      >
      <div class="space-y-4">
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <input
            v-model="inputDONew"
            type="text"
            id="nomorDO"
            placeholder="Nomor DO (misal: 2023001234)"
            class="border border-gray-300 rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
          <input
            v-model="inputDateNew"
            type="date"
            id="tanggalDO"
            class="border border-gray-300 rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <input
            v-model="inputNIMNew"
            type="text"
            id="nim"
            placeholder="NIM Penerima"
            class="border border-gray-300 rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
          <input
            v-model="inputNamaPenerimaNew"
            type="text"
            id="namaPenerima"
            placeholder="Nama Penerima"
            class="border border-gray-300 rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <select
            v-model="inputEkspedisiNew"
            id="ekspedisi"
            class="border border-gray-300 rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="" disabled>Pilih Ekspedisi</option>
            <option value="JNE">JNE</option>
            <option value="TIKI">TIKI</option>
            <option value="POS Indonesia">POS Indonesia</option>
          </select>

          <select
            v-model="selectBahanAjarNew"
            id="bahanAjar"
            class="border border-gray-300 rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="" disabled>Pilih Bahan Ajar</option>
            <option
              v-for="bahan in dataBahanAjar"
              :key="bahan.kodeBarang"
              :value="bahan.kodeBarang"
            >
              {{ bahan.namaBarang }} ({{ bahan.kodeBarang }})
            </option>
          </select>
        </div>

        <button
          @click="handleSearch"
          class="w-full bg-blue-600 text-white px-6 py-2 rounded-md hover:bg-blue-700 transition-colors font-semibold cursor-pointer"
        >
          Simpan Data Pengiriman
        </button>
      </div>
    </div>
    <!-- Form Input End -->
  </section>
</template>
