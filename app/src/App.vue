<template>
  <div>
    <label for="day">Day : </label>
    <select v-model="form.day">
      <option v-for="day in days" :key="day" :value="day">{{ day }}</option>
    </select>
    <label for="startTime">StartTime : </label>
    <input type="time" v-model="form.startTime" required />
    <label for="endTime">EndTime : </label>
    <input type="time" v-model="form.endTime" required />
    <button type="submit" @click="handleFormSubmission">Add</button>
  </div>
  <div class="timetable-container">
    <div class="timetable">
      <div class="time-column">
        <h3 style="font-weight: normal">Time</h3>
        <div
          class="time-slot"
          :key="time"
          v-for="time in times"
          :style="{ height: getSlotHeight() }"
        >
          {{ time }}
        </div>
      </div>
      <div v-for="day in days" :key="day" class="day">
        <h3>{{ day }}</h3>
        <div class="day-slots">
          <TimeSlot
            v-for="time in times"
            :key="time"
            :time="time"
            :height="getSlotHeight()"
            :appointment="getAppointment(day, time)"
            @selected-time="handleTimeSelection(day, time)"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, reactive } from "vue";
import TimeSlot from "@/components/TimeSlot.vue";

export default {
  name: "App",
  components: {
    TimeSlot,
  },
  setup() {
    const days = [
      "Sunday",
      "Monday",
      "Tuesday",
      "Wednesday",
      "Thursday",
      "Friday",
      "Saturday",
    ];
    const times = ref(generateHourlyTimes());

    const appointments = ref([]);
    const form = reactive({
      day: "Sunday",
      startTime: "09:00",
      endTime: "10:00",
    });

    function handleFormSubmission() {
      if (form.startTime < form.endTime) {
        addAppointment(form.day, form.startTime, form.endTime);
      } else {
        alert("The end time must be later than the start time");
      }
    }

    function generateHourlyTimes() {
      const times = [];
      for (let hour = 0; hour < 24; hour++) {
        times.push(`${String(hour).padStart(2, "0")}:00`);
      }
      times.push(`${String(23).padStart(2, "0")}:59`);
      return times;
    }

    function getSlotHeight() {
      return `20px`;
    }

    function handleTimeSelection(day, time) {
      console.log(day, time);
    }

    function addAppointment(day, startTime, endTime) {
      appointments.value.push({
        day,
        startTime,
        endTime,
      });
      updateTimes(startTime, endTime);
    }

    function updateTimes(startTime, endTime) {
      const appointmentTimes = generateTimesBetween(startTime, endTime);
      appointmentTimes.forEach((time) => {
        if (!times.value.includes(time)) {
          times.value.push(time);
        }
      });

      times.value.sort((a, b) => {
        return a.localeCompare(b);
      });
    }

    function generateTimesBetween(startTime, endTime) {
      let times = [];
      let [startHour, startMinute] = startTime.split(":").map(Number);
      let [endHour, endMinute] = endTime.split(":").map(Number);

      while (
        startHour < endHour ||
        (startHour === endHour && startMinute <= endMinute)
      ) {
        const formattedTime = `${String(startHour).padStart(2, "0")}:${String(
          startMinute
        ).padStart(2, "0")}`;
        times.push(formattedTime);
        if (startMinute === 59) {
          startMinute = 0;
          startHour++;
        } else {
          startMinute++;
        }
      }
      return times;
    }

    function getAppointment(day, time) {
      return appointments.value.find(
        (app) => app.day === day && app.startTime <= time && app.endTime >= time
      );
    }

    function shouldMerge(day, time) {
      const appointment = getAppointment(day, time);
      if (appointment) {
        return time.startTime > appointment.startTime;
      }
      return false;
    }

    return {
      days,
      times,
      form,
      handleFormSubmission,
      getSlotHeight,
      shouldMerge,
      handleTimeSelection,
      getAppointment,
      addAppointment,
    };
  },
};
</script>

<style>
.timetable-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.timetable {
  display: flex;
  flex-direction: row;
}

.time-column {
  display: flex;
  flex-direction: column;
}

.day-column {
  display: flex;
  flex-direction: column;
  margin-left: 10px;
}

.day-slots {
  display: flex;
  flex-direction: column;
}

.time-slot {
  border: 1px solid #ccc;
  padding: 1px;
  margin-bottom: -1px;
}

h3 {
  margin: 0;
  text-align: center;
}
</style>
