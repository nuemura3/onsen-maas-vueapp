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
    <!-- 以下にふらっぺガントチャートを表示 -->
    <div class="myfrappe">
      <h1>ふらっぺのお試し</h1>
      <!-- <svg id="gantt"></svg> -->
      <svg ref="gantt"></svg>
    </div>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  reactive,
  toRefs,
  computed,
  ref,
  onMounted,
} from "vue";
// ふらっぺのganttの読み込み
// import Gantt from "../ex_lib/frappe/frappe-gantt.js";
import Gantt from "frappe-gantt";

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

    // ふらっぺのgantt用のタスク
    const tasks = [
      {
        id: "Task 1",
        name: "生地コネ",
        start: "2021-11-28 13:00",
        end: "2021-11-29 12:00",
        progress: 80,
        dependencies: "",
      },
      {
        id: "Task 2",
        name: "焼き",
        start: "2021-11-29 15:00",
        end: "2021-11-30 11:00",
        progress: 30,
        dependencies: "Task 1",
      },
      {
        id: "Task 3",
        name: "トッピング",
        start: "2021-11-30 13:00",
        end: "2021-12-1 9:00",
        progress: 0,
        dependencies: "Task 2",
      },
    ];
    // ふらっぺのgantt
    // var gantt = new Gantt("#gantt", tasks);
    const gantt = ref<SVGElement>(); //HTMLElement or SVGElement

    onMounted(() => {
      if (!gantt.value) {
        return;
      }
      new Gantt(gantt.value, tasks, {
        header_height: 50,
        // column_width: 30,
        column_width: 60,
        step: 24,
        view_modes: ["Quarter Day", "Half Day", "Day", "Week", "Month"],
        // bar_height: 20,
        bar_height: 40,
        bar_corner_radius: 3,
        arrow_curve: 5,
        padding: 18,
        view_mode: "Quarter Day",
        date_format: "YYYY-MM-DD",
        language: "zh",
      }).change_view_mode("Quarter Day"); // Quarter Day, Half Day, Day, Week, Month
    });

    return {
      state,
      setDate,
      year,
      month,
      day,
      hours,
      minutes,
      seconds,
      gantt,
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
  font-family: "Teko", sans-serif;
  font-size: 4rem;
  letter-spacing: 0.1em;
  margin: 0em 0;
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
  font-family: "Roboto Mono", monospace;
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
  font-family: "Teko", sans-serif;
  font-size: 1.4rem;
  letter-spacing: 0.05em;
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

.gantt .grid-background {
  fill: none;
}

.gantt .grid-header {
  fill: #ffffff;
  stroke: #e0e0e0;
  stroke-width: 1.4;
}

.gantt .grid-row {
  fill: #ffffff;
}

.gantt .grid-row:nth-child(even) {
  fill: #f5f5f5;
}

.gantt .row-line {
  stroke: #ebeff2;
}

.gantt .tick {
  stroke: #e0e0e0;
  stroke-width: 0.2;
}
.gantt .tick.thick {
  stroke-width: 0.4;
}

.gantt .today-highlight {
  fill: #fcf8e3;
  opacity: 0.5;
}

.gantt .arrow {
  fill: none;
  stroke: #666;
  stroke-width: 1.4;
}

.gantt .bar {
  fill: #b8c2cc;
  stroke: #8d99a6;
  stroke-width: 0;
  transition: stroke-width 0.3s ease;
  user-select: none;
}

/* .gantt .bar-progress {
  fill: #a3a3ff;
} */
.myfrappe .bar-progress {
  fill: red;
}

.gantt .bar-invalid {
  fill: transparent;
  stroke: #8d99a6;
  stroke-width: 1;
  stroke-dasharray: 5;
}
.gantt .bar-invalid ~ .bar-label {
  fill: #555;
}

.gantt .bar-label {
  fill: #fff;
  dominant-baseline: central;
  text-anchor: middle;
  font-size: 12px;
  font-weight: lighter;
}
.gantt .bar-label.big {
  fill: #555;
  text-anchor: start;
}

.gantt .handle {
  fill: #ddd;
  cursor: ew-resize;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease;
}

.gantt .bar-wrapper {
  cursor: pointer;
  outline: none;
}
.gantt .bar-wrapper:hover .bar {
  fill: #a9b5c1;
}
.gantt .bar-wrapper:hover .bar-progress {
  fill: #8a8aff;
}
.gantt .bar-wrapper:hover .handle {
  visibility: visible;
  opacity: 1;
}
.gantt .bar-wrapper.active .bar {
  fill: #a9b5c1;
}
.gantt .bar-wrapper.active .bar-progress {
  fill: #8a8aff;
}

.gantt .lower-text,
.gantt .upper-text {
  font-size: 12px;
  text-anchor: middle;
}

.gantt .upper-text {
  fill: #555;
}

.gantt .lower-text {
  fill: #333;
}

.gantt .hide {
  display: none;
}

.gantt-container {
  position: relative;
  overflow: auto;
  font-size: 12px;
}
.gantt-container .popup-wrapper {
  position: absolute;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.8);
  padding: 0;
  color: #959da5;
  border-radius: 3px;
}
.gantt-container .popup-wrapper .title {
  border-bottom: 3px solid #a3a3ff;
  padding: 10px;
}
.gantt-container .popup-wrapper .subtitle {
  padding: 10px;
  color: #dfe2e5;
}
.gantt-container .popup-wrapper .pointer {
  position: absolute;
  height: 5px;
  margin: 0 0 0 -5px;
  border: 5px solid transparent;
  border-top-color: rgba(0, 0, 0, 0.8);
}
</style>