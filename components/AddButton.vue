<template>
  <button
    @click="$emit('add')"
    class="fixed bottom-4 right-4 bg-blue-500 text-white p-4 rounded-full shadow-lg flex items-center justify-center"
    style="width: 50px; height: 50px"
  >
    <svg viewBox="0 0 20 20" fill="currentColor" class="plus w-6 h-6">
      <path
        fill-rule="evenodd"
        d="M10 5a1 1 0 011 1v3h3a1 1 0 110 2h-3v3a1 1 0 11-2 0v-3H6a1 1 0 110-2h3V6a1 1 0 011-1z"
        clip-rule="evenodd"
      ></path>
    </svg>
  </button>
</template>
