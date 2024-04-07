<template>
  <aside
    class="bg-gray-700 text-white p-4 fixed inset-y-0 transform -translate-x-full transition-transform duration-300"
    :class="{ 'translate-x-0': showSidebar }"
    :style="{ zIndex: '2' }"
  >
    <button @click="$emit('toggle-sidebar')" class="absolute top-0 right-0 m-4">
      <svg
        class="h-6 w-6 text-white"
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke="currentColor"
        aria-hidden="true"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M6 18L18 6M6 6l12 12"
        />
      </svg>
    </button>

    <ul class="p-4 m-4">
      <li>
        <button @click="$emit('set-view', 'day')" class="block w-full p-2 mb-2">
          Day
        </button>
      </li>
      <li>
        <button
          @click="$emit('set-view', 'week')"
          class="block w-full p-2 mb-2"
        >
          Week
        </button>
      </li>
      <li>
        <button @click="$emit('set-view', 'month')" class="block w-full p-2">
          Month
        </button>
      </li>
    </ul>
  </aside>
</template>

<script>
export default {
  props: ['showSidebar'],
}
</script>
