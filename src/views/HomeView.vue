<template>
  <div class="home">
    <div class="content">
      <time-rule
        @changCurrentTime="changCurrentTime"
        :videoRecords="videoRecords"
      ></time-rule>
    </div>
  </div>
</template>

<script>
import timeRule from "@/components/time-rule";
export default {
  name: "home",
  components: { timeRule },
  data() {
    return {
      videoRecords: {
        startTime: 1711525845,
        endTime: 1711526845,
      },
    };
  },
  methods: {
    changCurrentTime(e) {
      console.log("time", e);
    },
  },
};
</script>

<style lang="scss" scoped>
.home {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  box-sizing: border-box;
  .content {
    width: 1000px;
    height: 60px;
    background: #999;
  }
}
</style>