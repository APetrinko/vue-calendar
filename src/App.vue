<script>
import { format, startOfMonth, endOfMonth, startOfWeek, endOfWeek, eachDayOfInterval, isSameDay, isToday } from 'date-fns';
import { utcToZonedTime } from 'date-fns-tz';
import { uk } from 'date-fns/locale';
import 'bulma';

export default {
  name: "Calendar",
  data() {
    return {
      currentMonth: utcToZonedTime(new Date('2023-03-01'), 'Europe/London'),
      daysOfWeek: ["Понеділок", "Вівторок", "Середа", "Четвер", "П'ятниця", "Субота", "Неділя"],
      events: [
        {
          start: utcToZonedTime(new Date("2023-03-26T14:00:00+03:00"), 'Europe/Kiev'),
          end: utcToZonedTime(new Date("2023-03-26T15:00:00+03:00"), 'Europe/Kiev'),
          timeZone: 'Europe/London',
          title: "Some Event",
          startInLondon: utcToZonedTime(new Date("2023-03-26T12:00:00+01:00"), 'Europe/London'),
          endInLondon: utcToZonedTime(new Date("2023-03-26T13:00:00+01:00"), 'Europe/London'),

        }
      ],
    };
  },
  computed: {
    weeks() {
      const startDate = startOfWeek(startOfMonth(this.currentMonth), { weekStartsOn: 1 });
      const endDate = endOfWeek(endOfMonth(this.currentMonth), { weekStartsOn: 1 });
      const dates = eachDayOfInterval({ start: startDate, end: endDate });

      const weeks = [];
      for (let i = 0; i < dates.length; i += 7) {
        weeks.push(dates.slice(i, i + 7));
      }

      return weeks;
    },
  },
  methods: {
    capitalizeFirstLetter(string) {
      return string.charAt(0).toUpperCase() + string.slice(1);
    },
    isToday(date) {
      return isToday(date);
    },
    formatDate(date, formatString) {
      return format(date, formatString, { locale: uk });
    },
    prevMonth() {
      this.currentMonth = new Date(this.currentMonth.getFullYear(), this.currentMonth.getMonth() - 1, 1);
    },
    nextMonth() {
      this.currentMonth = new Date(this.currentMonth.getFullYear(), this.currentMonth.getMonth() + 1, 1);
    },
    getEventsForDay(day) {
      return this.events.filter(event => isSameDay(event.start, day));
    },
  }
};
</script>

<template>
  <div class="calendar">
    <header class="header">
      <h1 class="title">{{ capitalizeFirstLetter(formatDate(currentMonth, "LLLL yyyy", { locale: uk })) }} р.</h1>
      <div class="buttons">
        <button class="button is-info" @click="prevMonth">&lt;</button>
        <button class="button is-info" @click="nextMonth">&gt;</button>
      </div>
    </header>

    <table class="table is-bordered">
      <thead>
        <tr>
          <th v-for="day in daysOfWeek" :key="day">{{ day }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="week in weeks" :key="week">
          <td v-for="day in week" :key="day" class="day"
            :class="{ 'is-info': getEventsForDay(day).length, 'is-warning': isToday(day) }">
            <div v-if="day && day.getMonth() === currentMonth.getMonth()">
              {{ formatDate(day, 'd') }}
            </div>
          </td>
        </tr>
      </tbody>
    </table>

    <table class="table is-bordered">
      <thead>
        <tr>
          <th>Event</th>
          <th>Time</th>
        </tr>
      </thead>
      <thead>
        <tr>
          <td class="notification is-info is-light" v-for="event in events">{{ event.title }}</td>
          <td class="notification is-warning is-light" v-for="time in events">
            {{ formatDate(time.startInLondon, 'dd MMMM HH:mm') }} - {{ formatDate(time.endInLondon, 'dd MMMM HH:mm') }}
          </td>
        </tr>
      </thead>
    </table>
  </div>
</template>

<style>
.header {
  display: flex;
  justify-content: space-around;
  width: 45%;
  align-items: center;
}

.calendar {
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
}

.day {
  height: 100px;
  width: 100px;
}

.day:hover {
  background-color: rgb(177, 177, 177);
  color: white;
  cursor: pointer;
}

.events {
  display: flex;
  justify-content: space-between;
}
</style>