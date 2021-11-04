<template>
  <div>
    <div class="container">
      <p class="date">{{ year }}/{{ month }}/{{ day }}</p>
      <div class="time">
        <p class="time-item hours">{{ hours }}</p>
        <p class="time-item minutes">{{ minutes }}</p>
        <p class="time-item seconds">{{ seconds }}</p>
      </div>
      <button @click="setDate">Get Now</button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs, computed } from "vue";

const zeroPadding = (num: number, digit: number) => {
  return (Array(digit).join("0") + num).slice(-digit);
};

interface _Data {
  date: Date;
}

export default defineComponent({
  setup(prop, context) {
    console.log(prop);
    console.log(context);

    const state = reactive<_Data>({ date: new Date() });
    const { date } = toRefs(state);

    const setDate = () => {
      date.value = new Date();
    };
    const year = computed(() => date.value.getFullYear());
    const month = computed(() => zeroPadding(date.value.getMonth() + 1, 2));
    const day = computed(() => zeroPadding(date.value.getDate(), 2));
    const hours = computed(() => zeroPadding(date.value.getHours(), 2));
    const minutes = computed(() => zeroPadding(date.value.getMinutes(), 2));
    const seconds = computed(() => zeroPadding(date.value.getSeconds(), 2));

    return {
      state,
      setDate,
      year,
      month,
      day,
      hours,
      minutes,
      seconds,
    };
  },
});
</script>

<style scoped>
.container {
  background-color: #3a4a5e;
  padding: 2%;
}
 
.date {
  text-align: right;
  color: #fff;
  font-family: 'Teko', sans-serif;
  font-size: 4rem;
  letter-spacing: .1em;
  margin: .0em 0;
  line-height: 1;
}
 
.time {
  display: flex;
}
 
.time-item {
  display: flex;
  justify-content: center;
  align-items: center;
  flex: 1 1;
  height: 100px;
  position: relative;
  z-index: 1;
  padding: 0.5em;
  margin: 3px;
  color: #fff;
  font-family: 'Roboto Mono', monospace;
  font-size: 3rem;
  line-height: 1;
  background-color: #48b883;
  box-sizing: border-box;
}
 
.time-item:before {
  position: absolute;
  right: 5px;
  bottom: 1px;
  z-index: 1;
  color: #3a4a5e;
  font-family: 'Teko', sans-serif;
  font-size: 1.4rem;
  letter-spacing: .05em;
}
 
.hours:before {
  content: "Hours";
}
.minutes:before {
  content: "Minutes";
}
 
.seconds:before {
  content: "Seconds";
}
</style>