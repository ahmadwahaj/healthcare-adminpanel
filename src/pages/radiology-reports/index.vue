<template>
  <router-view />
</template>
  <script>
import { mapActions } from "vuex";
export default {
  methods: {
    ...mapActions("radiologyReport", ["setSelectedRadiologyReport"]),
  },
  beforeDestroy() {
    this.setSelectedRadiologyReport(null);
  },
};
</script>