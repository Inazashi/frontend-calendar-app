<template>
  <header
    class="flex justify-between items-center bg-gray-800 text-white p-4"
    :style="{ height: '60px', zIndex: '1' }"
  >
    <button @click="$emit('toggle-sidebar')" class="p-2">☰</button>
    <div>
      <button @click="$emit('change-time', 'prev')" class="mx-2">←</button>
      <span>{{ summary }}</span>
      <button @click="$emit('change-time', 'next')" class="mx-2">→</button>
    </div>
  </header>
</template>

<script>
export default {
  props: ['summary'],
}
</script>
