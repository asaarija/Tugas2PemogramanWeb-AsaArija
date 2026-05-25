<script setup>
import { ref } from "vue";
import Swal from "sweetalert2";
import logoUt from "../../assets/logoUt.jpg";

const props = defineProps({
  activePage: {
    type: String,
    default: "Dashboard",
  },
});

const emit = defineEmits(["logout", "navigate"]);

const navItems = [
  { label: "Informasi Bahan Ajar", page: "Stock" },
  { label: "Tracking Pengiriman", page: "Tracking" },
  {
    label: "Laporan",
    dropdown: true,
    items: [
      {
        label: "Monitoring Progress DO Pengiriman",
        page: "#",
      },
      { label: "Rekap Bahan Ajar", page: "#" },
    ],
  },
  { label: "History Bahan Ajar", page: "# " },
];

const handleLogout = () => {
  const namaUser = localStorage.getItem("namaUser");
  Swal.fire({
    title: "Logout Berhasil",
    text: `Sampai Jumpa, ${namaUser}!`,
    icon: "success",
    confirmButtonColor: "blue",
    confirmButtonText: "OK",
  }).then(() => {
    localStorage.removeItem("namaUser");
    emit("logout");
  });
};

const handleNavigate = (page) => {
  emit("navigate", page);
};
// State untuk mengelola dropdown
const openDropDown = ref(null);

const toggleDropDown = (label) => {
  console.log("Toggling dropdown for:", label);
  openDropDown.value = openDropDown.value === label ? null : label;
};
</script>

<template>
  <header
    class="fixed top-0 left-0 z-20 w-full bg-blue-500 h-[10vh] border-b-4 border-yellow-400 shadow-lg"
  >
    <div
      class="container mx-auto h-full px-4 flex items-center justify-between gap-4"
    >
      <div class="flex items-center gap-4">
        <img
          :src="logoUt"
          alt="Logo UT"
          class="w-20 h-auto cursor-pointer"
          @click.prevent="handleNavigate('Dashboard')"
        />

        <nav>
          <ul class="flex gap-6 items-center">
            <li v-for="item in navItems" :key="item.page" class="font-bold">
              <button
                v-if="!item.dropdown"
                @click.prevent="handleNavigate(item.page)"
                :class="{
                  'text-yellow-200': props.activePage === item.page,
                  'text-white hover:text-yellow-400':
                    props.activePage !== item.page,
                }"
                class="transition-colors cursor-pointer"
              >
                {{ item.label }}
              </button>
              <!-- Dropdown starts -->
              <div v-else>
                <button
                  @click="toggleDropDown(item.label)"
                  class="text-white hover:text-yellow-400 transition-colors"
                >
                  {{ item.label }}
                </button>
                <ul
                  v-if="openDropDown === item.label"
                  class="absolute mt-2 bg-white shadow-lg rounded-lg min-w-62.5 z-50"
                >
                  <li v-for="subItem in item.items" :key="subItem.label">
                    <button
                      @click.prevent="handleNavigate(subItem.page)"
                      class="block w-full text-left px-4 py-2 hover:bg-gray-100 text-black"
                    >
                      {{ subItem.label }}
                    </button>
                  </li>
                </ul>
              </div>
              <!-- Dropdown ends -->
            </li>
          </ul>
        </nav>
      </div>

      <div class="flex items-center gap-4">
        <button
          @click="handleLogout"
          class="flex items-center gap-2 bg-yellow-400 text-black px-4 py-2 rounded-full shadow hover:bg-yellow-300 transition-colors cursor-pointer"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            height="20px"
            viewBox="0 -960 960 960"
            width="20px"
            fill="#000000"
          >
            <path
              d="M200-120q-33 0-56.5-23.5T120-200v-560q0-33 23.5-56.5T200-840h280v80H200v560h280v80H200Zm440-160-55-58 102-102H360v-80h327L585-622l55-58 200 200-200 200Z"
            />
          </svg>
          Logout
        </button>
      </div>
    </div>
  </header>
</template>
