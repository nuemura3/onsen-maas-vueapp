<template>
  <Clock />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import Clock from "@/components/Clock.vue";
// import "moment";
// import "snapsvg-cjs";
// import Gantt from "./ex_lib/frappe/frappe-gantt.js";

export default defineComponent({
  components: { Clock },
});
</script>
  
<style lang="scss">
</style>
