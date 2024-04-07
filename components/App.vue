<template>
  <div>
    <Header
      :summary="summary"
      @toggle-sidebar="toggleSidebar"
      @change-time="changeTime"
    />
    <Sidebar
      :showSidebar="showSidebar"
      @set-view="setView"
      @toggle-sidebar="toggleSidebar"
    />
    <CalendarGrid
      :view="view"
      :entries="entries"
      :currentDate="currentDate"
      @edit-entry="editEntry"
      @set-view="setView"
      @change-to-day="changeTimeToDay"
    />
    <AddButton @add="addEntry" />
    <EntryModal
      :isOpen="modalOpen"
      :entry="currentEntry"
      :isMessage="isModalMessage"
      :message="modalMessage"
      :isError="isModalError"
      @save="saveEntry"
      @deleteEntry="deleteEntry"
      @close="closeModal"
    />
  </div>
</template>

<script>
import Header from '~/components/Header.vue'
import Sidebar from '~/components/Sidebar.vue'
import CalendarGrid from '~/components/CalendarGrid.vue'
import AddButton from '~/components/AddButton.vue'
import EntryModal from '~/components/EntryModal.vue'

export default {
  components: {
    Header,
    Sidebar,
    CalendarGrid,
    AddButton,
    EntryModal,
  },
  data() {
    return {
      showSidebar: false,
      currentDate: new Date(),
      view: 'day',
      entries: [],
      modalOpen: false,
      currentEntry: {},
      modalMessage: null,
      isModalError: false,
      isModalMessage: false,
    }
  },
  computed: {
    summary() {
      switch (this.view) {
        case 'day':
          return this.$dateFns.format(this.currentDate, 'PPP')
        case 'week':
          const start = this.$dateFns.format(
            this.$dateFns.startOfWeek(this.currentDate),
            'PPP'
          )
          const end = this.$dateFns.format(
            this.$dateFns.endOfWeek(this.currentDate),
            'PPP'
          )
          return `${start} - ${end}`
        case 'month':
          return this.$dateFns.format(this.currentDate, 'MMMM yyyy')
        default:
          return ''
      }
    },
  },
  mounted() {
    this.fetchEntries()
  },
  methods: {
    toggleSidebar() {
      this.showSidebar = !this.showSidebar
    },
    setView(view) {
      this.view = view
    },
    changeTimeToDay(day) {
      this.currentDate = day
    },
    changeTime(direction) {
      switch (this.view) {
        case 'day':
          this.currentDate =
            direction === 'next'
              ? this.$dateFns.addDays(this.currentDate, 1)
              : this.$dateFns.subDays(this.currentDate, 1)
          break
        case 'week':
          this.currentDate =
            direction === 'next'
              ? this.$dateFns.addDays(this.currentDate, 7)
              : this.$dateFns.subDays(this.currentDate, 7)
          break
        case 'month':
          this.currentDate =
            direction === 'next'
              ? this.$dateFns.addDays(
                  this.$dateFns.endOfMonth(this.currentDate),
                  1
                )
              : this.$dateFns.subDays(
                  this.$dateFns.startOfMonth(this.currentDate),
                  1
                )
          break
      }
    },
    addEntry() {
      this.currentEntry = {}
      this.modalOpen = true
    },
    editEntry(entry) {
      this.currentEntry = Object.assign({}, entry)
      this.modalOpen = true
    },
    closeModal() {
      this.modalOpen = false
      this.currentEntry = {}
      this.modalMessage = null
      this.isModalError = false
      this.isModalMessage = false
    },
    adjustEntryTimesToLocal(entries) {
      return entries.map((entry) => {
        const startTime = new Date(entry.startTime)
        const endTime = new Date(entry.endTime)

        const timezoneOffset = new Date().getTimezoneOffset() * 60000

        const localStartTime = new Date(startTime.getTime() - timezoneOffset)
        const localEndTime = new Date(endTime.getTime() - timezoneOffset)

        return {
          ...entry,
          startTime: localStartTime.toISOString().slice(0, -1),
          endTime: localEndTime.toISOString().slice(0, -1),
        }
      })
    },
    async fetchEntries() {
      try {
        const response = await this.$axios.$get('/api/entries')
        this.entries = this.adjustEntryTimesToLocal(response.data)
      } catch (error) {
        console.error('Failed to fetch entries:', error)
      }
    },
    async saveEntry(entry) {
      try {
        const savedEntry = entry._id
          ? await this.$axios.$put(`/api/entries/${entry._id}`, entry)
          : await this.$axios.$post('/api/entries', entry)
        this.modalMessage = 'Entry saved successfully!'
        this.isModalError = false
        this.isModalMessage = true
        this.fetchEntries()
      } catch (error) {
        if (
          error.response &&
          error.response.data &&
          error.response.data.message
        ) {
          this.modalMessage = `Failed to save entry: ${error.response.data.message}`
        } else {
          this.modalMessage = 'Failed to save entry due to an unexpected error.'
        }
        this.isModalError = true
        this.isModalMessage = true
      }
    },
    async deleteEntry(id) {
      try {
        await this.$axios.$delete(`/api/entries/${id}`)
        this.modalMessage = 'Entry deleted successfully!'
        this.isModalError = false
        this.isModalMessage = true
        this.fetchEntries()
      } catch (error) {
        if (
          error.response &&
          error.response.data &&
          error.response.data.message
        ) {
          this.modalMessage = `Failed to delete entry: ${error.response.data.message}`
        } else {
          this.modalMessage =
            'Failed to delete entry due to an unexpected error.'
        }
        this.isModalError = true
        this.isModalMessage = true
      }
    },
  },
}
</script>
