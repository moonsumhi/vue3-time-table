<template>
  <div
    :style="{ height: height }"
    @click="onClick"
    class="time-slot"
    :class="{ 'with-appointment': appointment }"
  >
    <div v-if="appointment" class="appointment">
      <div v-if="appointment.startTime === time">
        ShutDown {{ appointment.startTime }} ~ {{ appointment.endTime }}
      </div>
    </div>
  </div>
</template>

<script>
import { computed, defineComponent } from "vue";

export default defineComponent({
  name: "TimeSlot",
  props: {
    time: {
      type: String,
      required: true,
    },
    height: {
      type: String,
      required: true,
    },
    appointment: {
      type: Object,
      default: null,
    },
  },
  setup(props, { emit }) {
    const formattedTime = computed(() => {
      const [hour, minute] = props.time.split(":");
      const hourNumber = parseInt(hour, 10);
      const ampm = hourNumber >= 12 ? "PM" : "AM";
      const formattedHour = hourNumber % 12 || 12;
      return `${formattedHour}:${minute} ${ampm}`;
    });

    function onClick() {
      emit("selected-time", props.time);
    }

    return {
      onClick,
      formattedTime,
    };
  },
});
</script>

<style scoped>
.time-slot {
  border: 1px solid #ddd;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: -1px;
  position: relative;
}

.appointment {
  background-color: red;
  padding: 1px;
  border-radius: 3px;
  text-align: center;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.time-slot.with-appointment {
  border-color: red;
}
</style>
