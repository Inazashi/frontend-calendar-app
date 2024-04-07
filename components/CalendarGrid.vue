<template>
  <div>
    <!-- Day View -->
    <template v-if="view === 'day'">
      <div v-for="(hour, index) in hours" :key="index" class="flex hour-slot">
        <span>{{ hour }}:00</span>
      </div>
      <div
        v-for="entry in filteredEntriesForDay()"
        :key="entry._id"
        :style="entryStyleDay(entry)"
        class="entry p-2 rounded shadow-inner bg-blue-200 truncate"
        @click="openEntry(entry)"
      >
        {{ entry.title }}
      </div>
    </template>

    <!-- Week View -->
    <template v-else-if="view === 'week'">
      <div class="grid grid-cols-8">
        <div
          v-for="(day, index) in [null, ...weekDays]"
          :key="index"
          class="text-center py-2"
        >
          <span v-if="day">{{ formatDay(day, 'EEE') }}</span>
        </div>
      </div>
      <div v-for="hour in hours" :key="hour" class="hour-slot">
        <span>{{ hour }}:00</span>
      </div>
      <div v-for="(day, index) in weekDays" :key="day">
        <div
          v-for="entry in filteredEntriesForWeek(day)"
          :key="entry._id"
          :style="entryStyleWeek(entry, day, index)"
          class="entry week p-2 rounded shadow-inner bg-blue-200"
          @click="openEntry(entry)"
        >
          {{ entry.title }}
        </div>
      </div>
    </template>

    <!-- Month View -->
    <template v-else-if="view === 'month'">
      <div
        :class="['h-screen', 'w-screen']"
        :style="{ height: 'calc(100vh - 60px)' }"
      >
        <div class="grid grid-cols-7">
          <div
            v-for="(day, index) in weekDays"
            :key="index"
            class="text-center py-2"
          >
            {{ formatDay(day, 'EEE') }}
          </div>
        </div>
        <div :class="gridClass" :style="{ height: 'calc(100% - 40px)' }">
          <div
            v-for="(day, index) in monthDays"
            :key="index"
            @click="switchToDayView(day)"
            class="day-cell border p-2 relative cursor-pointer overflow-hidden"
          >
            <div v-if="day">
              <div class="text-sm">{{ formatDay(day, 'd') }}</div>
              <div
                v-for="entry in filteredEntriesForMonth()[
                  formatDay(day, 'yyyy-MM-dd')
                ].entries"
                :key="entry._id"
                class="entry month p-2 mb-2 rounded shadow-inner bg-blue-200 truncate"
              >
                {{ entry.title }}
              </div>
              <div
                v-if="
                  filteredEntriesForMonth()[formatDay(day, 'yyyy-MM-dd')]
                    .length > 2
                "
                class="more-entries-indicator absolute bottom-0 right-0 bg-gray-200 text-gray-800 px-1 text-xs"
              >
                +{{
                  filteredEntriesForMonth()[formatDay(day, 'yyyy-MM-dd')]
                    .length - 2
                }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
export default {
  props: {
    view: String,
    entries: Array,
    currentDate: Date,
  },
  computed: {
    gridClass() {
      switch (this.view) {
        case 'day':
          return 'grid grid-cols-1 mb-4'
        case 'week':
          return 'grid grid-cols-7 mb-4'
        case 'month':
          return 'grid grid-cols-7 grid-rows-6 auto-rows-auto'
      }
    },
    hours() {
      return this.$dateFns
        .eachHourOfInterval({
          start: this.$dateFns.startOfDay(this.currentDate),
          end: this.$dateFns.endOfDay(this.currentDate),
        })
        .map((date) => this.$dateFns.format(date, 'HH'))
    },
    weekDays() {
      return this.$dateFns.eachDayOfInterval({
        start: this.$dateFns.startOfWeek(this.currentDate, { weekStartsOn: 0 }),
        end: this.$dateFns.endOfWeek(this.currentDate, { weekStartsOn: 0 }),
      })
    },
    monthDays() {
      const start = this.$dateFns.startOfMonth(this.currentDate)
      const end = this.$dateFns.endOfMonth(this.currentDate)
      let days = this.$dateFns.eachDayOfInterval({ start, end })

      const firstDayOfWeek = this.$dateFns.getDay(start)
      const emptyDays = firstDayOfWeek === 0 ? 6 : firstDayOfWeek

      for (let i = 0; i < emptyDays; i++) {
        days.unshift(null)
      }

      return days
    },
  },
  methods: {
    formatDay(day, dayFormat) {
      return this.$dateFns.format(day, dayFormat)
    },
    filteredEntriesForDay() {
      const dayStart = new Date(this.currentDate.setHours(0, 0, 0, 0))
      const dayEnd = new Date(this.currentDate.setHours(23, 59, 59, 999))

      return this.entries
        .filter((entry) => {
          const entryStart = new Date(entry.startTime)
          const entryEnd = new Date(entry.endTime)
          return (
            (entryStart >= dayStart && entryStart < dayEnd) ||
            (entryEnd > dayStart && entryEnd <= dayEnd)
          )
        })
        .sort((a, b) => new Date(a.startTime) - new Date(b.startTime))
    },
    filteredEntriesForWeek(day) {
      return this.entries
        .filter((entry) => {
          const entryStart = new Date(entry.startTime)
          const entryEnd = new Date(entry.endTime)
          return (
            this.isSameDay(entryStart, day) || this.isSameDay(entryEnd, day)
          )
        })
        .sort((a, b) => new Date(a.startTime) - new Date(b.startTime))
    },
    filteredEntriesForMonth() {
      const entriesGroupedByDay = {}
      this.monthDays
        .filter((day) => !!day)
        .forEach((day) => {
          const dayKey = this.$dateFns.format(day, 'yyyy-MM-dd')
          const dateEntries = this.entries.filter(
            (entry) =>
              this.$dateFns.format(new Date(entry.startTime), 'yyyy-MM-dd') ===
                dayKey ||
              this.$dateFns.format(new Date(entry.endTime), 'yyyy-MM-dd') ===
                dayKey
          )
          entriesGroupedByDay[dayKey] = {
            entries: dateEntries
              .sort((a, b) => new Date(a.startTime) - new Date(b.startTime))
              .slice(0, 2),
            length: dateEntries.length,
          }
        })
      return entriesGroupedByDay
    },
    switchToDayView(day) {
      this.$emit('set-view', 'day')
      this.$emit('change-to-day', day)
    },
    isSameDay(date1, date2) {
      return (
        this.$dateFns.format(date1, 'yyyy-MM-dd') ===
        this.$dateFns.format(date2, 'yyyy-MM-dd')
      )
    },
    openEntry(entry) {
      this.$emit('edit-entry', entry)
    },
    entryStyleDay(entry) {
      const startOfDay = new Date(this.currentDate).setHours(0, 0, 0, 0)
      const endOfDay = new Date(this.currentDate).setHours(23, 59, 59, 999)
      const startTime = new Date(entry.startTime)
      const endTime = new Date(entry.endTime)

      let top = 0 + 60
      let height

      if (startTime < startOfDay && endTime > endOfDay) {
        height = (endOfDay - startOfDay) / 60000
      } else if (startTime < startOfDay) {
        height = (endTime - startOfDay) / 60000
      } else if (endTime > endOfDay) {
        top = (startTime - startOfDay) / 60000 + 60
        height = (endOfDay - startTime) / 60000
      } else {
        top = (startTime - startOfDay) / 60000 + 60
        height = (endTime - startTime) / 60000
      }

      return {
        top: `${top}px`,
        height: `${height}px`,
        position: 'absolute',
        marginLeft: '50px',
        width: 'calc(100vw - 100px)',
      }
    },
    entryStyleWeek(entry, day, index) {
      const startOfDay = new Date(day).setHours(0, 0, 0, 0)
      const endOfDay = new Date(day).setHours(23, 59, 59, 999)
      const startTime = new Date(entry.startTime)
      const endTime = new Date(entry.endTime)

      let top = 0 + 100
      let height

      if (startTime < startOfDay && endTime > endOfDay) {
        height = (endOfDay - startOfDay) / 60000
      } else if (startTime < startOfDay) {
        height = (endTime - startOfDay) / 60000
      } else if (endTime > endOfDay) {
        top = (startTime - startOfDay) / 60000 + 100
        height = (endOfDay - startTime) / 60000
      } else {
        top = (startTime - startOfDay) / 60000 + 100
        height = (endTime - startTime) / 60000
      }

      return {
        top: `${top}px`,
        height: `${height}px`,
        left: `calc((100vw / 8) * ${index + 1})`,
        position: 'absolute',
        width: `calc(100vw / 8)`,
      }
    },
  },
}
</script>

<style scoped>
.hour-slot {
  height: 60px;
}

.hour-slot,
.day-column,
.day-cell {
  border-bottom: 1px solid #ccc;
}

.day-header,
.day-number {
  font-weight: bold;
}
</style>
