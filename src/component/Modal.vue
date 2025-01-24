<template>
  <TransitionRoot as="template" :show="isOpen">
    <Dialog class="relative z-20" @close="closeModal">
      <TransitionChild
          as="template"
          enter="ease-out duration-300"
          enter-from="opacity-0"
          enter-to="opacity-100"
          leave="ease-in duration-200"
          leave-from="opacity-100"
          leave-to="opacity-0"
      >
        <div class="fixed inset-0 bg-gray-500/75 transition-opacity"></div>
      </TransitionChild>

      <div class="fixed inset-0 z-10 w-screen overflow-y-auto">
        <div class="flex min-h-full items-end justify-center p-4 text-center sm:items-center sm:p-0">
          <TransitionChild
              as="template"
              enter="ease-out duration-300"
              enter-from="opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
              enter-to="opacity-100 translate-y-0 sm:scale-100"
              leave="ease-in duration-200"
              leave-from="opacity-100 translate-y-0 sm:scale-100"
              leave-to="opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
          >
            <DialogPanel
                class="relative transform overflow-hidden rounded-lg bg-white text-left shadow-xl transition-all sm:my-8 sm:w-full sm:max-w-3xl"
            >
              <div class="bg-white px-4 pt-5 pb-4 sm:p-8 sm:pb-8">
                <div class="sm:flex sm:items-start">
                  <div
                      class="mx-auto flex h-12 w-12 shrink-0 items-center justify-center rounded-full bg-green-100 sm:mx-0 sm:h-10 sm:w-10"
                  >
                  </div>
                  <div class="mt-3 text-center sm:mt-0 sm:ml-4 sm:text-left">
                    <DialogTitle as="h3" class="text-base font-semibold text-gray-900">
                      {{ title }}
                    </DialogTitle>
                    <div class="mt-4">
                      <div class="grid grid-cols-1 gap-y-4 border-t border-gray-200 pt-4 sm:grid-cols-2 sm:gap-x-6 sm:gap-y-6">
                        <!-- Base section -->
                        <div
                            v-for="(value, key) in data.base"
                            :key="key"
                            class="flex justify-between items-start text-sm bg-white rounded-lg shadow-md p-4 hover:bg-gray-50"
                        >
            <span class="font-semibold text-gray-700">
              {{ camelToPascalWithSpaces(key) }}:
            </span>
                          <span class="text-gray-600">
              {{ key === "uptime" ? value = formatUptime(value) : value }}
            </span>
                        </div>
                      </div>
                    </div>

                    <!-- System Stats Section -->
                    <div class="mt-4">
                      <div class="grid grid-cols-1 gap-y-4 border-t border-gray-200 pt-4 sm:grid-cols-2 sm:gap-x-6 sm:gap-y-6">
                        <div
                            v-for="(value, key) in data.systemStats"
                            :key="key"
                            :class="{
                'flex justify-between items-start text-sm bg-white rounded-lg shadow-md p-4 hover:bg-gray-50': true,
                'bg-red-50 border border-red-200': key.includes('usage') // Example for coloring based on usage
              }"
                        >
            <span class="font-semibold text-gray-700">
              {{ camelToPascalWithSpaces(snakeToPascal(key))}}:
            </span>
                          <span class="text-gray-600">
              <!-- Humanizing the values -->
              <template v-if="key === 'cpu_usage' || key === 'disk_usage' || key === 'memory_usage'">
                {{ humanizePercentage(value) }}
              </template>
              <template v-else-if="key === 'memory_used_gb' || key === 'memory_total_gb'">
                {{ humanizeMemory(value) }}
              </template>
              <template v-else-if="key === 'disk_used_gb' || key === 'disk_total_gb'">
                {{ humanizeDisk(value) }}
              </template>
              <template v-else-if="key === 'net_upload_mb' || key === 'net_download_mb'">
                {{ humanizeNetwork(value) }}
              </template>
              <template v-else>
                {{ value }}
              </template>
            </span>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
              <div class="bg-gray-50 px-4 py-3 sm:flex sm:flex-row-reverse sm:px-6">
                <button
                    type="button"
                    class="mt-3 inline-flex w-full justify-center rounded-md bg-white px-3 py-2 text-sm font-semibold text-gray-900 ring-1 shadow-sm ring-gray-300 ring-inset hover:bg-gray-50 sm:mt-0 sm:w-auto"
                    @click="closeModal"
                    ref="cancelButtonRef"
                >
                  {{ cancelText }}
                </button>
              </div>
            </DialogPanel>
          </TransitionChild>
        </div>
      </div>
    </Dialog>
  </TransitionRoot>
</template>

<script setup>
import {ref, defineProps, defineEmits, onMounted} from 'vue'
import { Dialog, DialogPanel, DialogTitle, TransitionChild, TransitionRoot } from '@headlessui/vue'


const props = defineProps({
  isOpen: {
    type: Boolean,
    required: true,
  },
  title: {
    type: String,
    default: 'Modal Title',
  },
  message: {
    type: String,
    default: 'Modal description goes here. Customize as needed.',
  },
  confirmText: {
    type: String,
    default: 'Confirm',
  },
  cancelText: {
    type: String,
    default: 'Cancel',
  },
  data: {
    type: Object,
    default: {}
  }
})

const emit = defineEmits(['close', 'confirm'])

const closeModal = () => {
  emit('close')
}

const formatUptime = (seconds) => {
  const days = Math.floor(seconds / 86400);
  const hours = Math.floor((seconds % 86400) / 3600);
  return `${days} Days, ${hours} Hours`;
};

function camelToPascalWithSpaces(input) {
  return input
      .replace(/([a-z])([A-Z])/g, '$1 $2') // Tambahkan spasi sebelum huruf kapital
      .replace(/^./, (match) => match.toUpperCase()); // Kapitalisasi huruf pertama
}

function  humanizePercentage(value) {
  return value.toFixed(2) + "%";
}
function  humanizeMemory(value) {
  return value.toFixed(2) + " GB";
}
function humanizeDisk(value) {
  return value.toFixed(2) + " GB";
}

function snakeToPascal(str) {
  return str
      .split('_')
      .map(word => word.charAt(0).toUpperCase() + word.slice(1))
      .join('');
}

function humanizeNetwork(value) {
  if (value < 1024) {
    return value.toFixed(2) + " MB";
  } else {
    return (value / 1024).toFixed(2) + " GB";
  }
}

</script>

<style scoped>
</style>
