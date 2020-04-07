<template>
  <div id="sweetCalendar">
    <div class="container calendar">
      <div class="header">
        <div v-if="previousMonthMustBeVisible" class="left-arrow"
             @click="prevMonth">
          <span>&lt;</span>
        </div>
        <div class="month">{{ selectedMonthName }} {{ selectedYear }}</div>
        <div class="right-arrow"
             @click="nextMonth">
          <span>&gt;</span>
        </div>
      </div>
      <div class="body">
        <div v-for="(day, index) in weekdays"
             :key="`day-name-${index + 1}`"
             class="day-name"
             :title="day">
          {{ day[0] }}
        </div>
        <div v-for="(day,index) in days"
             :key="index"
             class="day-container">

          <div class="before"
               v-if="day"
               :style="generateBeforeStyle(day)">&nbsp;</div>
          <div v-if="day"
               :class="[
            'day',
            `day-${day.getDate()},
            weekday-${day.getDay()}`,
            offDays.includes(day.getDay()) ? 'off-day' : null,
            day.toDateString() === today.toDateString() ? 'today' : null,
            ]"
               @click="selectDay(day)"
               :style="generateDayStyle(day)">
            <span>{{ day.getDate() }}</span>
          </div>
          <div class="after"
               v-if="day"
               :style="generateAfterStyle(day)">&nbsp;</div>

          <div style="
            position: absolute;
            bottom: 0;
            text-align: center;
            width: 100%;
            font-size: 10px;"><span :style="generateColor(day)" v-html="generateText(day)"></span></div>

        </div>
      </div>
    </div>
  </div>
</template>

<script>

  import DateTime from '../DateTime.js'

  export default {
    name: 'Calendar',
    data() {
      return {
        today: new DateTime(),
        date: null,
        weekdays: null
      }
    },
    computed: {
      previousMonthMustBeVisible: function () {
        var checkDate = new Date();
        if (selectedYear == checkDate.getFullYear()) {
          if (checkDate.getMonth() == this.selectedMonth - 1)
            return false;
        }

        return true;
      },
      days() {
        let emptyDays = Array((this.startWeekDayOfMonth - this.firstDayOfWeek + 7) % 7).fill(null)
        let days = Array(this.numberOfDays).fill().map((item, index) => new DateTime(this.selectedYear, this.selectedMonth, index + 1))
        return emptyDays.concat(days)
      },
      startWeekDayOfMonth() {
        return this.date.getFirstWeekdayOfMonth()
      },
      numberOfDays() {
        return this.date.getNumberOfDaysInMonth()
      },
      selectedMonth() {
        return this.date.getMonth()
      },
      selectedMonthName() {
        const monthNames = ["Gennaio", "Febbraio", "Marzo", "Aprile", "Maggio", "Giugno",
          "Luglio", "Agosto", "Settembre", "Ottobre", "Novembre", "Dicembre"
        ];

        return monthNames[this.date.getMonth() - 1];
      },
      selectedYear() {
        return this.date.getFullYear()
      }
    },
    methods: {
      prevMonth() {
        this.date = new DateTime(this.selectedYear, this.selectedMonth - 1, 1)
      },
      nextMonth() {
        this.date = new DateTime(this.selectedYear, this.selectedMonth + 1, 1)
      },
      generateWeekdayNames(firstDayOfWeek = 1) {
        let weekdays = [
          'Domenica',
          'Lunedi',
          'Martedi',
          'Mercoledi',
          'Giovedi',
          'Venerdi',
          'Sabato'
        ]
        for (let i = 2; i <= firstDayOfWeek; i++) {
          let first = weekdays.shift()
          weekdays.push(first)
        }
        return weekdays
      },
      selectDay(day) {
        this.$emit('setDate', {
          iso: day.toISOString(),
          timeZoneOffset: day.getTimezoneOffset()
        })
      },
      generateDayStyle(date) {
        let style = {}
        for (let event of this.events) {
          if (date.isInRange(event.start, event.end, event.repeat)) {
            let category = this.eventCategories.find(item => item.id === event.categoryId) || {}
            Object.assign(style, {
              color: category.id ? category.textColor : null,
              backgroundColor: category.id ? category.backgroundColor : null,
              fontWeight: category.id ? 'bold' : 'normal'
            })
          }
        }
        return style
      },
      generateText(date) {
        for (let text of this.textDays) {
          if (date === null) continue;
          if (date.isInRange(text.day, text.day, 'never')) {
            return text.text
          }
        }

        return ''
      },
      generateColor(date) {
        for (let text of this.textDays) {
          if (date === null) continue;
          if (date.isInRange(text.day, text.day, 'never')) {
            return "color:" + text.color;
          }
        }

        return ''
      },
      generateBeforeStyle(date) {
        let style = {}
        for (let event of this.events) {
          if (date.isInRange(event.start, event.end, event.repeat) && date.getPrevDay().isInRange(event.start, event.end, event.repeat)) {
            let category = this.eventCategories.find(item => item.id === event.categoryId) || {}
            Object.assign(style, {
              backgroundColor: category.backgroundColor
            })
          }
        }
        return style
      },
      generateAfterStyle(date) {
        let style = {}
        for (let event of this.events) {
          if (date.isInRange(event.start, event.end, event.repeat) && date.getNextDay().isInRange(event.start, event.end, event.repeat)) {
            let category = this.eventCategories.find(item => item.id === event.categoryId) || {}
            Object.assign(style, {
              backgroundColor: category.backgroundColor
            })
          }
        }
        return style
      },
      goToday() {
        this.date = this.today
      }
    },
    props: {
      initialDate: {
        type: String,
        default: null
      },
      firstDayOfWeek: {
        type: Number,
        default: 1 // 1: Sunday, 2: Monday, etc
      },
      eventCategories: {
        type: Array,
        default() {
          return []
        }
      },
      events: {
        type: Array,
        default() {
          return []
        }
      },
      textDays: {
        type: Array,
        default() {
          return []
        }
      },
      offDays: {
        type: Array,
        default() {
          return [1, 7]
        }
      }
    },
    beforeMount() {
      this.date = Date.parse(this.initialDate) ? new DateTime(this.initialDate) : new DateTime()
      this.weekdays = this.generateWeekdayNames(this.firstDayOfWeek)
    }
  }
</script>

<style lang="sass" scoped>
  @import '../styles/index.sass'
</style>
