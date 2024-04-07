<template>
  <div
    v-if="isOpen"
    class="fixed inset-0 flex justify-center items-center"
    :class="{ 'opacity-0 pointer-events-none': !isOpen }"
    @click.self="closeModal"
  >
    <div
      class="absolute inset-0 bg-black opacity-50 transition-opacity duration-300"
    ></div>

    <div
      class="bg-white p-8 rounded-lg w-3/4 z-10 transform transition-all duration-300"
      :class="{ 'scale-95 opacity-0': !isOpen }"
    >
      <div class="flex flex-col items-center" v-if="isMessage">
        <p class="message-text text-center mb-4">{{ message }}</p>
        <button
          type="button"
          @click="close"
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
        >
          Close
        </button>
      </div>
      <div v-else>
        <h2 class="text-xl font-semibold mb-4">
          {{ entry._id ? 'Edit Entry' : 'New Entry' }}
        </h2>
        <form @submit.prevent="submitForm">
          <div class="mb-4">
            <label
              for="title"
              class="block text-gray-700 text-sm font-bold mb-2"
              >Title</label
            >
            <input
              type="text"
              id="title"
              v-model="entry.title"
              maxlength="20"
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
              required
            />
          </div>

          <div class="mb-4">
            <label
              for="description"
              class="block text-gray-700 text-sm font-bold mb-2"
              >Description</label
            >
            <textarea
              id="description"
              v-model="entry.description"
              maxlength="100"
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            ></textarea>
          </div>

          <div class="mb-4">
            <label
              for="startTime"
              class="block text-gray-700 text-sm font-bold mb-2"
              >Start Time:</label
            >
            <input
              type="datetime-local"
              id="startTime"
              v-model="form.startTime"
              :max="form.endTime"
              class="form-input"
              required
            />
          </div>

          <div class="mb-4">
            <label
              for="endTime"
              class="block text-gray-700 text-sm font-bold mb-2"
              >End Time:</label
            >
            <input
              type="datetime-local"
              id="endTime"
              v-model="form.endTime"
              :min="form.startTime"
              class="form-input"
              required
            />
          </div>

          <div class="flex items-center justify-between">
            <button
              type="submit"
              class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
            >
              Save
            </button>
            <button
              type="button"
              @click="close"
              class="bg-yellow-500 hover:bg-yellow-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
            >
              Cancel
            </button>
            <button
              type="button"
              @click="deleteEntry"
              class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
              v-if="entry._id"
            >
              Delete
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    isOpen: Boolean,
    isMessage: Boolean,
    message: String,
    isError: Boolean,
    entry: {
      type: Object,
      default: () => ({
        title: '',
        description: '',
        startTime: '',
        endTime: '',
      }),
    },
  },
  data() {
    return {
      form: {
        title: '',
        description: '',
        startTime: '',
        endTime: '',
      },
    }
  },
  watch: {
    entry: {
      handler(newVal) {
        this.form = { ...newVal }
        if (newVal.startTime && newVal.endTime) {
          this.form.startTime = newVal.startTime
          this.form.endTime = newVal.endTime
        }
      },
      immediate: true,
      deep: true,
    },
  },
  methods: {
    submitForm() {
      this.$emit('save', {
        ...this.entry,
        startTime: this.form.startTime,
        endTime: this.form.endTime,
      })
    },
    deleteEntry() {
      this.$emit('deleteEntry', this.entry._id)
    },
    close() {
      this.$emit('close')
    },
  },
}
</script>
