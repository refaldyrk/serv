<template>
  <div class="w-full h-screen">
    <Header></Header>
    <section class="mr-10 ml-10 lg:mr-20 lg:ml-20 mt-10">
      <div>
        <div class="grid grid-flow-col grid-rows-4 gap-4">
          <span class="text-green-300 font-medium">
            Hi, Welcome To Serv
          </span>
          <span class="text-white">
            Real-time Server Monitoring Made Easy with
            <span class="text-green-300 font-medium">Serv</span>
          </span>
        </div>
      </div>
    </section>
    <section class="mr-10 ml-10 lg:mr-20 lg:ml-20">
      <div class="grid sm:grid-cols-1 lg:grid-cols-3 gap-4">
        <CardHome title="Total" :count="status.total" color="text-white"></CardHome>
        <CardHome title="Online" :count="status.online" color="text-green-400"></CardHome>
        <CardHome title="Offline" :count="status.offline" color="text-red-400"></CardHome>
      </div>
    </section>
    <section class="ml-10 mr-10">
      <div class="grid sm:grid-cols-1 md:grid-cols-3 xl:grid-cols-3 gap-6 justify-items-center px-3 py-4">
        <div
            v-for="(server, index) in servers"
            :key="index"
            class="max-w-sm w-full bg-gray-800 shadow-lg rounded-xl overflow-hidden transform hover:scale-105 transition-transform duration-300"
        >
          <!-- Header -->
          <div :class="{
  'bg-gradient-to-r from-green-400 via-green-500 to-green-600': server.isOnline,
  'bg-gradient-to-r from-red-400 via-red-500 to-red-600': !server.isOnline
}" class="p-4">
            <div class="flex justify-between items-center">
              <h2 class="text-lg font-semibold text-white">
                {{ server.data[0].base.hostname }}
              </h2>
              <span class="text-sm text-gray-200">
            {{ server.data[0].base.platform }} {{ server.data[0].base.platformVersion }}
          </span>
            </div>
          </div>

          <!-- Body -->
          <div class="px-6 py-4">
            <!-- System Stats -->
            <div class="mt-3 grid grid-cols-2 gap-4 text-gray-300">
              <!-- Uptime -->
              <div class="text-center">
                <span class="block text-sm font-medium">Uptime</span>
                <p class="text-lg font-semibold text-green-400">
                  {{ formatUptime(server.data[0].base.uptime) }}
                </p>
              </div>
              <!-- CPU Usage -->
              <div class="text-center">
                <span class="block text-sm font-medium">CPU Usage</span>
                <p class="text-lg font-semibold text-red-400">
                  {{ server.data[0].systemStats.cpu_usage.toFixed(2) }}%
                </p>
              </div>
            </div>

            <!-- Memory and Storage -->
            <div class="mt-4 grid grid-cols-2 gap-4 text-gray-300">
              <!-- Memory Usage -->
              <div class="text-center">
                <span class="block text-sm font-medium">Memory</span>
                <p class="text-lg font-semibold text-blue-400">
                  {{ server.data[0].systemStats.memory_usage.toFixed(1) }}%
                </p>
              </div>
              <!-- Disk Usage -->
              <div class="text-center">
                <span class="block text-sm font-medium">Storage</span>
                <p class="text-lg font-semibold text-yellow-400">
                  {{ server.data[0].systemStats.disk_usage.toFixed(1) }}%
                </p>
              </div>
            </div>
          </div>

          <!-- Footer -->
          <div class="bg-gray-900 px-4 py-3">
            <button @click="showDetails" :class="{
  'bg-green-600 hover:bg-green-700': server.isOnline,
  'bg-red-600 hover:bg-red-700': !server.isOnline
}" class="text-white text-sm font-medium px-4 py-2 rounded-full transition-colors duration-200">
              View Details
            </button>

            <Modal
                :data="server.data[server.data.length-1]"
                :isOpen="open"
                title="Detail Server"
                cancelText="Close"
                @close="open = false"
                @confirm="handleDeactivate"
            />
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";
import Header from "@/component/Header.vue";
import CardHome from "@/component/CardHome.vue";
import Modal from "@/component/Modal.vue";
import {data} from "autoprefixer";

const status = ref({
  offline: 0,
  online: 0,
  total: 0,
});

const servers = ref([]);

const fetchStatus = async () => {
  const apiUrl = import.meta.env.VITE_APP_SERV_BACKEND;
  try {
    const response = await fetch(`${apiUrl}/status`);
    status.value = await response.json();
  } catch (error) {
    console.error("Error fetching status:", error);
  }
};

const fetchServers = async () => {
  const apiUrl = import.meta.env.VITE_APP_SERV_BACKEND;
  try {
    const response = await fetch(`${apiUrl}/process?limit=10000&page=1`);
    servers.value = await response.json();
  } catch (error) {
    console.error("Error fetching server data:", error);
  }
};
const showDetails = (data) => {
  open.value =true
};

const open = ref(false)

const handleDeactivate = () => {
  console.log('Account deactivated')
  open.value = false
}

const formatUptime = (seconds) => {
  const days = Math.floor(seconds / 86400);
  const hours = Math.floor((seconds % 86400) / 3600);
  return `${days} Days, ${hours} Hours`;
};

onMounted(() => {
  fetchStatus();
  fetchServers();

  const interval = setInterval(() => {
    fetchStatus();
    fetchServers();
  }, 5000);

  onUnmounted(() => clearInterval(interval));
});
</script>