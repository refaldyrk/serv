<template>
  <div class="max-w-sm rounded-3xl overflow-hidden shadow-lg bg-gray-900">
    <div class="px-6 py-10">
      <div class="grid grid-cols-6 gap-4">
        <div class="col-start-1 col-end-3">
            <span :class="color">
              {{ title }}
            </span>
        </div>
        <div class="col-span-2 col-end-8">
            <span :class="color">
              {{ count }}
            </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: [
    "title",
    "count",
    "color"
  ]
}
</script>
